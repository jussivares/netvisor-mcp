# Netvisor MCP TECH SPEC – Peer Review

> **Arvioija:** Claude (MCP-kehityskokemus: PRH MCP, Tilastokeskus MCP, Finlex MCP)  
> **Päivämäärä:** 2025-12-21  
> **Arvioitavat dokumentit:** TECH_SPEC_Netvisor_MCP.md v1.0, Addendum v1.2  
> **Kokonaisarvio:** ⭐⭐⭐⭐ (4/5) – Vahva pohja, muutama kriittinen puute

---

## Yhteenveto

Dokumentaatio on **erittäin laadukas** ja osoittaa syvällistä ymmärrystä sekä MCP-arkkitehtuurista että Netvisor-integraatiosta. DataProvider-abstraktio on erinomainen ratkaisu Phase 2:ta varten. Kuitenkin **kolme kriittistä puutetta** vaatii huomiota ennen toteutusta:

| Prioriteetti | Puute | Riski |
|--------------|-------|-------|
| 🔴 KRIITTINEN | Token-hallinta puuttuu | Claude Desktop overflow |
| 🔴 KRIITTINEN | Multi-tenant (customerSwitch) epäselvä | API-kutsut väärään asiakkaaseen |
| 🟡 KORKEA | Concurrent request handling | Race conditions |

---

## 1. KRIITTINEN: Token-hallinta ja vastausten koko

### Ongelma

Netvisorin pääkirja voi sisältää **tuhansia tositteita** per tilikausi. Esimerkiksi:
- 100 myyntilaskua/kk × 12 kk = 1200 tositetta
- Jokainen tosite 5-10 riviä = 6000-12000 kirjausriviä
- JSON-muodossa helposti 500KB - 2MB

Claude Desktop context window on rajallinen. PRH MCP:ssä ja Tilastokeskus MCP:ssä tämä ratkaistiin **token-hallinnalla**:

### Suositeltu ratkaisu

```javascript
// src/utils/tokenManager.js
const MAX_RESPONSE_TOKENS = 50000; // ~200KB JSON

class TokenManager {
  constructor(maxTokens = MAX_RESPONSE_TOKENS) {
    this.maxTokens = maxTokens;
  }
  
  /**
   * Arvioi JSON-objektin token-määrä (karkea arvio: 4 merkkiä = 1 token)
   */
  estimateTokens(obj) {
    const json = JSON.stringify(obj);
    return Math.ceil(json.length / 4);
  }
  
  /**
   * Leikkaa vastaus token-rajaan ja lisää jatkumisinfo
   */
  truncateResponse(items, options = {}) {
    const { 
      itemName = 'items',
      includeMetadata = true 
    } = options;
    
    let result = [];
    let currentTokens = 0;
    let truncated = false;
    
    for (const item of items) {
      const itemTokens = this.estimateTokens(item);
      
      if (currentTokens + itemTokens > this.maxTokens * 0.9) { // 90% raja
        truncated = true;
        break;
      }
      
      result.push(item);
      currentTokens += itemTokens;
    }
    
    const response = {
      [itemName]: result,
      _meta: {
        totalCount: items.length,
        returnedCount: result.length,
        truncated,
        estimatedTokens: currentTokens
      }
    };
    
    if (truncated) {
      response._meta.continuationHint = 
        `Palautettiin ${result.length}/${items.length} ${itemName}. ` +
        `Käytä tarkempia suodattimia (accountNumberStart/End, startDate/endDate) ` +
        `saadaksesi loput tulokset.`;
    }
    
    return response;
  }
}

module.exports = { TokenManager, MAX_RESPONSE_TOKENS };
```

### Vaikutus Task-09 (getLedger)

```javascript
// src/tools/getLedger.js (LISÄYS)
const { TokenManager } = require('../utils/tokenManager');

async function handler(params) {
  // ... olemassa oleva koodi ...
  
  const vouchers = await dataProvider.getLedger(/*...*/);
  const transformed = vouchers.map(v => transformVoucher(v, accountMap));
  
  // UUSI: Token-hallinta
  const tokenManager = new TokenManager();
  const response = tokenManager.truncateResponse(transformed, {
    itemName: 'vouchers'
  });
  
  return {
    success: true,
    data: response
  };
}
```

### Uudet Test Scenariot

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-09.10 | EC | Yli 1000 tositetta → vastaus katkaistaan, _meta.truncated = true |
| TS-09.11 | HP | Katkaistu vastaus sisältää continuationHint-viestin |
| TS-09.12 | HP | Token-arvio on ±20% tarkkuudella |

---

## 2. KRIITTINEN: Multi-tenant ja customerSwitch

### Ongelma

Dokumenteissa mainitaan `organizationId` (Y-tunnus) parametrina, mutta **@rantalainen/netvisor-api-client** käyttää eri mekanismia:

```javascript
// Kirjaston tapa vaihtaa asiakasta:
const client = new NetvisorApiClient({
  customerId: 'PARTNER_ID',  // Partnerin ID
  customerKey: 'PARTNER_KEY',
  // ...
});

// Vaihtaa kontekstin toiseen asiakkaaseen:
await client.customers.switchCustomer({ 
  organizationId: '1234567-8' 
});
```

### Ongelma nykyisessä suunnitelmassa

Task-05 `createClient()` näyttää luovan uuden clientin joka kutsulle:

```javascript
// src/client.js (nykyinen)
function createClient(organizationId) {
  const config = loadConfig();
  // Ongelma: Miten organizationId käytetään?
  // Kirjasto ei hyväksy sitä konstruktorissa!
}
```

### Suositeltu ratkaisu

```javascript
// src/client.js (KORJATTU)
const { NetvisorApiClient } = require('@rantalainen/netvisor-api-client');
const { loadConfig } = require('./config');

// Singleton client instance
let clientInstance = null;
let currentOrganizationId = null;

/**
 * Luo tai palauttaa NetvisorApiClient-instanssin.
 * Vaihtaa asiakaskontekstin tarvittaessa.
 * 
 * @param {string} organizationId - Kohdeasiakkaan Y-tunnus
 * @returns {Promise<NetvisorApiClient>}
 */
async function getClient(organizationId) {
  const config = loadConfig();
  
  // Luo client jos ei ole
  if (!clientInstance) {
    clientInstance = new NetvisorApiClient({
      customerId: config.customerId,
      customerKey: config.customerKey,
      partnerId: config.partnerId,
      partnerKey: config.partnerKey,
      language: 'FI',
      timeout: config.timeout
    });
  }
  
  // Vaihda asiakaskonteksti jos eri kuin nykyinen
  if (organizationId !== currentOrganizationId) {
    await clientInstance.customers.switchCustomer({
      organizationId: organizationId
    });
    currentOrganizationId = organizationId;
  }
  
  return clientInstance;
}

/**
 * Resetoi client (testaukseen)
 */
function resetClient() {
  clientInstance = null;
  currentOrganizationId = null;
}

module.exports = { getClient, resetClient };
```

### HUOMIO: switchCustomer vaatii tarkistuksen

**Ennen toteutusta tarkista:**
1. Tukeeko `@rantalainen/netvisor-api-client v4.1.1` switchCustomer-metodia?
2. Mikä on kirjaston API multi-tenant-käytölle?
3. Tarvitaanko erillinen "partner mode" -konfiguraatio?

Katso: https://github.com/rantalainen/netvisor-api-client

### Uudet Test Scenariot

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-05.8 | HP | getClient('1234567-8') → switchCustomer kutsutaan |
| TS-05.9 | HP | Sama organizationId peräkkäin → switchCustomer EI kutsuta |
| TS-05.10 | ER | Tuntematon organizationId → NotFoundError (asiakas ei olemassa) |

---

## 3. KORKEA: Concurrent Request Handling

### Ongelma

MCP-serverit voivat saada useita kutsuja samanaikaisesti (esim. Claude kutsuu getLedger ja getAccounts yhtä aikaa). Nykyinen singleton-client + switchCustomer aiheuttaa **race condition**:

```
Kutsu A: getClient('1111111-1') → switchCustomer('1111111-1')
Kutsu B: getClient('2222222-2') → switchCustomer('2222222-2')  // Ohittaa A:n!
Kutsu A: client.accounting.ledger() → Väärä asiakas!
```

### Suositeltu ratkaisu

**Vaihtoehto 1: Request-kohtainen client (suositeltu MVP:lle)**

```javascript
// src/client.js (CONCURRENT-SAFE)
async function getClient(organizationId) {
  const config = loadConfig();
  
  // Luo AINA uusi client joka kutsulle
  const client = new NetvisorApiClient({
    customerId: config.customerId,
    customerKey: config.customerKey,
    partnerId: config.partnerId,
    partnerKey: config.partnerKey,
    language: 'FI',
    timeout: config.timeout
  });
  
  // Vaihda asiakaskonteksti
  await client.customers.switchCustomer({
    organizationId: organizationId
  });
  
  return client;
}
```

**Vaihtoehto 2: Request queue (Phase 2)**

```javascript
// src/utils/requestQueue.js
class RequestQueue {
  constructor() {
    this.queue = [];
    this.processing = false;
  }
  
  async enqueue(organizationId, operation) {
    return new Promise((resolve, reject) => {
      this.queue.push({ organizationId, operation, resolve, reject });
      this.processNext();
    });
  }
  
  async processNext() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const { organizationId, operation, resolve, reject } = this.queue.shift();
    
    try {
      const client = await getClient(organizationId);
      const result = await operation(client);
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.processing = false;
      this.processNext();
    }
  }
}
```

### Suositus

MVP: Käytä **Vaihtoehto 1** (uusi client per kutsu). Se on yksinkertainen ja toimii. Netvisor-yhteyden muodostaminen on nopea operaatio.

---

## 4. KESKITASO: getAccounts kutsutaan liian usein

### Ongelma

Addendum v1.2 korjasi getBalances:n kutsumaan getAccounts:ia sisäisesti. Tämä tarkoittaa:

```
getBalances() → getAccounts() + getAccountBalance() = 2 API-kutsua
getBalances() → getAccounts() + getAccountBalance() = 2 API-kutsua (uudelleen!)
```

accountMap ei muutu usein, mutta sitä haetaan joka kerta.

### Suositeltu ratkaisu

```javascript
// src/data/ApiDataProvider.js (LISÄYS: In-memory cache)

class ApiDataProvider extends DataProvider {
  constructor() {
    super();
    // Yksinkertainen in-memory cache accountMap:lle
    this._accountMapCache = new Map(); // organizationId → { accountMap, timestamp }
    this._cacheMaxAge = 5 * 60 * 1000; // 5 minuuttia
  }
  
  async getAccountMap(organizationId) {
    const cached = this._accountMapCache.get(organizationId);
    
    if (cached && (Date.now() - cached.timestamp) < this._cacheMaxAge) {
      return cached.accountMap;
    }
    
    const { accountMap } = await this.getAccounts(organizationId);
    this._accountMapCache.set(organizationId, {
      accountMap,
      timestamp: Date.now()
    });
    
    return accountMap;
  }
  
  async getBalances(organizationId, balanceDates, intervalType = 0) {
    const client = await getClient(organizationId);
    
    // Käytä cachettua accountMap:ia
    const accountMap = await this.getAccountMap(organizationId);
    
    const response = await client.accounting.getAccountBalance({
      balanceDates,
      intervalType
    });
    
    return { rawBalances: response, accountMap };
  }
}
```

---

## 5. KESKITASO: getVoucher endpoint-tarkistus

### Ongelma

Addendum ehdottaa:
```javascript
if (typeof client.accounting.getVoucher === 'function') {
  return await client.accounting.getVoucher({ netvisorKey });
}
```

**Mutta:** Tarkistin @rantalainen/netvisor-api-client v4.1.1 dokumentaation – `getVoucher`-metodia **ei ole** kirjastossa!

### Suositeltu ratkaisu

Netvisor API:n `getvoucher.nv` endpoint toimii näin:

```
GET /getvoucher.nv?NetvisorKey=12345
```

Kirjasto ei tarjoa tätä suoraan, joten tarvitaan oma implementaatio:

```javascript
// src/api/getVoucherDirect.js
const https = require('https');
const { createAuthHeaders } = require('./netvisorAuth');

/**
 * Hae yksittäinen tosite suoraan Netvisor API:sta.
 * Käytetään koska @rantalainen/netvisor-api-client ei tue getvoucher.nv:tä.
 * 
 * @param {object} config - Netvisor-konfiguraatio
 * @param {string} organizationId - Y-tunnus
 * @param {number} netvisorKey - Tositteen ID
 * @returns {Promise<object>} Tositteen tiedot
 */
async function getVoucherDirect(config, organizationId, netvisorKey) {
  const url = `https://integration.netvisor.fi/getvoucher.nv?NetvisorKey=${netvisorKey}`;
  
  const headers = createAuthHeaders(config, organizationId, url);
  
  return new Promise((resolve, reject) => {
    const req = https.get(url, { headers }, (res) => {
      let data = '';
      res.on('data', chunk => data += chunk);
      res.on('end', () => {
        try {
          // Netvisor palauttaa XML:ää, tarvitaan parsinta
          const parsed = parseNetvisorXml(data);
          resolve(parsed);
        } catch (e) {
          reject(e);
        }
      });
    });
    
    req.on('error', reject);
    req.setTimeout(config.timeout || 30000, () => {
      req.destroy();
      reject(new Error('Timeout'));
    });
  });
}
```

### VAIHTOEHTOINEN: Avaa issue kirjastoon

Jos `getvoucher.nv` on tärkeä, harkitse:
1. Avaa GitHub issue: https://github.com/rantalainen/netvisor-api-client/issues
2. Pyydä `getVoucher`-metodin lisäämistä
3. Tai tee PR itse

---

## 6. MATALA: Timeout ja MCP-yhteensopivuus

### Ongelma

```javascript
timeout: z.number().default(120000) // 2 minuuttia
```

Claude Desktop MCP-yhteys saattaa katkaista **ennen 120s**. Tyypillinen timeout on 30-60s.

### Suositus

```javascript
// Lyhyempi timeout, mutta retry:
timeout: z.number().default(45000), // 45s
retryAttempts: z.number().default(2)
```

Ja lisää käyttäjäystävällinen viesti:

```javascript
if (error.code === 'ETIMEDOUT') {
  throw new TimeoutError(
    'Netvisor-kutsu aikakatkaistiin. Kokeile pienempää aikaväliä ' +
    '(esim. 1 kuukausi kerrallaan) tai tarkempia suodattimia.',
    { suggestion: 'Jaa haku pienempiin osiin' }
  );
}
```

---

## 7. MATALA: Sivutuksen puuttuminen

### Ongelma

Netvisor API **ei tue sivutusta** pääkirjahaussa. API palauttaa kaiken kerralla.

### Suositus dokumentaatioon

Lisää TECH_SPEC:iin selkeä maininta:

```markdown
### Netvisor API:n rajoitukset

**Sivutus:** Netvisor accounting API ei tue sivutusta. Kaikki data palautetaan 
yhdessä vastauksessa. Suurilla datamäärillä käytä päivämääräsuodatusta:

- `startDate` / `endDate`: Rajaa aikaväli (suositus: max 3kk kerrallaan)
- `accountNumberStart` / `accountNumberEnd`: Rajaa tilinumerot

**Rate limiting:** Netvisor rajoittaa pyyntöjä, mutta ei palauta Retry-After 
headeria. Käytä exponential backoff -strategiaa.
```

---

## 8. MATALA: Päivämäärä- ja aikavyöhykekäsittely

### Ongelma

Netvisor käyttää **Suomen aikaa** (EET/EEST). Dokumenteissa ei mainita aikavyöhykettä.

### Suositus

```javascript
// src/utils/dates.js

/**
 * Muunna päivämäärä Netvisor-muotoon (YYYY-MM-DD, Suomen aika).
 * Netvisor olettaa kaikki päivämäärät Suomen ajassa.
 */
function toNetvisorDate(dateString) {
  // Validoi muoto
  if (!/^\d{4}-\d{2}-\d{2}$/.test(dateString)) {
    throw new ValidationError(`Virheellinen päivämäärä: ${dateString}. Käytä muotoa YYYY-MM-DD`);
  }
  return dateString; // Netvisor hyväksyy tämän sellaisenaan
}

/**
 * Muunna Netvisor-päivämäärä ISO-muotoon vastauksessa.
 * Lisää Suomen aikavyöhyke selkeyden vuoksi.
 */
function fromNetvisorDate(dateString) {
  if (!dateString) return null;
  // Netvisor: "2024-01-15" → ISO: "2024-01-15" (ei muunnosta tarvita)
  return dateString;
}
```

---

## 9. Positiiviset huomiot

| Osa-alue | Kommentti |
|----------|-----------|
| **DataProvider-abstraktio** | Erinomainen! Mahdollistaa Phase 2 SQLite-cachen ilman refaktorointia |
| **Zod-validointi** | Oikea valinta MCP-kontekstiin – selkeät virheviestit |
| **Suomenkieliset virheet** | Claude-käyttäjäkokemus paranee merkittävästi |
| **Traceability matrix** | Ammattimainen – helpottaa testien kattavuuden varmistamista |
| **Test scenarios** | Kattavat HP/ER/EC-kategoriat |
| **CommonJS-valinta** | Yhteensopiva Claude Desktop MCP:n kanssa |
| **Gemini peer review** | Hyvä esimerkki iteratiivisesta parantamisesta |

---

## 10. Suositeltu toteutusjärjestys (päivitetty)

```
┌─────────────────────────────────────────────────────────────────┐
│  VAIHE 0: Esiselvitykset (ENNEN KOODAUSTA)                     │
│  ─────────────────────────────────────────────────────────────  │
│  □ Tarkista @rantalainen/netvisor-api-client:                  │
│    - switchCustomer-metodin olemassaolo ja käyttö              │
│    - getVoucher-metodin olemassaolo                            │
│    - Multi-tenant dokumentaatio                                │
│  □ Hanki sandbox-tunnukset testausta varten                    │
│                                                                 │
│  VAIHE 1: Infrastruktuuri (Task-01 – Task-05)                  │
│  ─────────────────────────────────────────────────────────────  │
│  Task-01: Konfiguraatio (+ timeout 45s)                        │
│  Task-02: Error-luokat                                          │
│  Task-03: Zod-skeemat                                           │
│  Task-04: Retry-logiikka (lyhennetyt viiveet)                  │
│  Task-05: Client wrapper (+ concurrent-safe)     ← PÄIVITETTY  │
│  Task-NEW: TokenManager                          ← UUSI        │
│                                                                 │
│  VAIHE 2: DataProvider + Transformerit                         │
│  ─────────────────────────────────────────────────────────────  │
│  Task-05b: DataProvider (+ accountMap cache)                    │
│  Task-06: Pääkirja-transformer                                  │
│  Task-07: Saldo-transformer                                     │
│  Task-08: Dimensio-transformer (sumea tunnistus)                │
│                                                                 │
│  VAIHE 3: MCP Tools                                             │
│  ─────────────────────────────────────────────────────────────  │
│  Task-10: getAccounts                                           │
│  Task-09: getLedger (+ token truncation)         ← PÄIVITETTY  │
│  Task-11: getBalances                                           │
│  Task-12: getDimensions                                         │
│  Task-13: getVoucher (tarkista API-tuki!)        ← HUOMIO      │
│  Task-14: MCP Server                                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 11. Tarkistuslista ennen toteutusta

- [ ] **Tarkista kirjaston multi-tenant tuki** – switchCustomer vai muu mekanismi?
- [ ] **Tarkista getVoucher-endpoint** – tukeeko kirjasto, vai tarvitaanko oma implementaatio?
- [ ] **Hanki sandbox-tunnukset** – integraatiotestejä varten
- [ ] **Lisää TokenManager** – estä context window overflow
- [ ] **Päätä concurrent strategy** – uusi client per kutsu (suositus MVP)
- [ ] **Päivitä timeout** – 120s → 45s
- [ ] **Lisää accountMap cache** – vähennä API-kutsuja

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.0 | 2025-12-21 | Alkuperäinen peer review |

---

*Arvioija: Claude (MCP-kehityskokemus)*  
*Projekti: Numbers Tilintarkastaja-Controller*
