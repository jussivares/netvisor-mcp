# TECH_SPEC_Netvisor_MCP – Addendum v1.2

> **Versio:** 1.2 (Addendum)  
> **Päivitetty:** 2025-12-18  
> **Muuttaa:** TECH_SPEC_Netvisor_MCP.md v1.0  
> **Muutoksen syy:** DataProvider-abstraktio (v1.1) + Gemini peer review -korjaukset (v1.2)

---

## Yhteenveto muutoksista

### v1.1 Muutokset (DataProvider)

| Muutos | Tyyppi | Vaikutus |
|--------|--------|----------|
| **Task-05b** | UUSI | DataProvider interface + ApiDataProvider |
| **Task-09** | PÄIVITYS | Käyttää DataProvideria |
| **Task-10** | PÄIVITYS | Käyttää DataProvideria |
| **Task-11** | PÄIVITYS | Käyttää DataProvideria |
| **Projektirakenne** | PÄIVITYS | Uusi `src/data/` kansio |
| **Toteutusjärjestys** | PÄIVITYS | Task-05b ennen Task-09 |

### v1.2 Muutokset (Gemini peer review)

| Muutos | Tyyppi | Vaikutus | Gemini # |
|--------|--------|----------|----------|
| **Task-03** | PÄIVITYS | Zod-skeemat tukevat v4.0.0 wrapper-rakennetta | #1 |
| **Task-04** | PÄIVITYS | Retry-viiveet lyhennetty MCP-yhteensopiviksi | #4 |
| **Task-07** | PÄIVITYS | getBalances hakee accountMap:n sisäisesti | #2 |
| **Task-08** | PÄIVITYS | Dimensioiden sumea tunnistus | #5 |
| **Task-13** | PÄIVITYS | getVoucher käyttää suoraa API-endpointia | #3 |

---

## 1. UUSI: Task-05b – DataProvider Interface

**Toteuttaa:** Valmistautuminen Phase 2 SQLite-cacheen  
**Arvioitu kesto:** 2h  
**Prioriteetti:** MVP  
**Sijainti:** Task-05:n jälkeen, ennen Task-06:ta

### Kuvaus

Luo DataProvider-abstraktiokerros joka erottaa datan hakemisen sen lähteestä. MVP:ssä käytetään `ApiDataProvider`:ia joka hakee suoraan Netvisorista. Phase 2:ssa lisätään `CachedDataProvider` joka käyttää SQLite-välimuistia.

**Hyödyt:**
- Ei refaktorointia kun SQLite lisätään
- Testattavuus: MockDataProvider testeissä
- Clean architecture: Single Responsibility

### Test Scenarios

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-05b.1 | HP | `getDataProvider()` palauttaa ApiDataProvider-instanssin |
| TS-05b.2 | HP | ApiDataProvider.getLedger() kutsuu Netvisor API:a |
| TS-05b.3 | HP | ApiDataProvider.getAccounts() kutsuu Netvisor API:a |
| TS-05b.4 | HP | ApiDataProvider.getBalances() kutsuu Netvisor API:a |
| TS-05b.5 | HP | ApiDataProvider.getDimensions() kutsuu Netvisor API:a |
| TS-05b.6 | HP | DataProvider voidaan injektoida (dependency injection) |

### Koodirunko

```javascript
// src/data/DataProvider.js
/**
 * Abstract base class for data providers.
 * MVP: ApiDataProvider (direct API calls)
 * Phase 2: CachedDataProvider (SQLite cache)
 */
class DataProvider {
  /**
   * Hae pääkirja
   * @param {string} organizationId - Y-tunnus
   * @param {string} startDate - YYYY-MM-DD
   * @param {string} endDate - YYYY-MM-DD
   * @param {object} options - Lisäparametrit
   * @returns {Promise<Array>} Tositteet
   */
  async getLedger(organizationId, startDate, endDate, options = {}) {
    throw new Error('getLedger() must be implemented by subclass');
  }

  /**
   * Hae tilikartta
   * @param {string} organizationId - Y-tunnus
   * @returns {Promise<object>} { accounts: [], defaultAccounts: {}, accountMap: Map }
   */
  async getAccounts(organizationId) {
    throw new Error('getAccounts() must be implemented by subclass');
  }

  /**
   * Hae tilisaldot
   * @param {string} organizationId - Y-tunnus
   * @param {string} balanceDates - YYYY-MM-DD,YYYY-MM-DD
   * @param {number} intervalType - 0-4
   * @returns {Promise<Array>} Saldot
   */
  async getBalances(organizationId, balanceDates, intervalType = 0) {
    throw new Error('getBalances() must be implemented by subclass');
  }

  /**
   * Hae dimensiot
   * @param {string} organizationId - Y-tunnus
   * @param {boolean} showHidden - Näytä piilotetut
   * @returns {Promise<object>} { costCenters: [], projects: [], custom: {} }
   */
  async getDimensions(organizationId, showHidden = false) {
    throw new Error('getDimensions() must be implemented by subclass');
  }

  /**
   * Hae yksittäinen tosite
   * @param {string} organizationId - Y-tunnus
   * @param {number} netvisorKey - Tositteen ID
   * @returns {Promise<object>} Tosite
   */
  async getVoucher(organizationId, netvisorKey) {
    throw new Error('getVoucher() must be implemented by subclass');
  }

  /**
   * Tyhjennä välimuisti (Phase 2)
   * @param {string} organizationId - Y-tunnus (optional, tyhjentää kaiken jos ei annettu)
   */
  async invalidateCache(organizationId = null) {
    // MVP: no-op, Phase 2: tyhjentää SQLite-cachen
  }

  /**
   * Hae välimuistin tila (Phase 2)
   * @param {string} organizationId - Y-tunnus
   * @returns {Promise<object>} Cache status
   */
  async getCacheStatus(organizationId) {
    // MVP: palauttaa { cached: false }
    return { cached: false, message: 'Cache not implemented in MVP' };
  }
}

module.exports = { DataProvider };
```

```javascript
// src/data/ApiDataProvider.js
const { DataProvider } = require('./DataProvider');
const { createClient } = require('../client');
const { transformVoucher } = require('../transformers/ledger');
const { transformBalances } = require('../transformers/balances');
const { transformDimensions } = require('../transformers/dimensions');

/**
 * MVP DataProvider - hakee datan suoraan Netvisor API:sta.
 * Ei välimuistia, jokainen kutsu menee API:in.
 */
class ApiDataProvider extends DataProvider {
  
  async getLedger(organizationId, startDate, endDate, options = {}) {
    const client = createClient(organizationId);
    
    const vouchers = await client.accounting.accountingLedger({
      startDate,
      endDate,
      accountNumberStart: options.accountNumberStart,
      accountNumberEnd: options.accountNumberEnd,
      voucherStatus: 1 // Vain voimassaolevat
    });
    
    return vouchers;
  }

  async getAccounts(organizationId) {
    const client = createClient(organizationId);
    const response = await client.accounting.accountList();
    
    // Rakenna accountMap (netvisorKey → account info)
    const accountMap = new Map();
    const accounts = response.accounts?.account || [];
    
    for (const acc of accounts) {
      accountMap.set(acc.netvisorKey, {
        number: acc.number,
        name: acc.name,
        accountType: acc.accountType,
        isActive: acc.isActive
      });
    }
    
    return {
      accounts,
      defaultAccounts: response.companyDefaultAccounts || {},
      accountMap
    };
  }

  async getBalances(organizationId, balanceDates, intervalType = 0) {
    const client = createClient(organizationId);
    
    const response = await client.accounting.getAccountBalance({
      balanceDates,
      intervalType
    });
    
    return response;
  }

  async getDimensions(organizationId, showHidden = false) {
    const client = createClient(organizationId);
    
    const response = await client.dimensions.dimensionList({
      showhidden: showHidden ? 1 : undefined
    });
    
    return response;
  }

  async getVoucher(organizationId, netvisorKey) {
    const client = createClient(organizationId);
    
    // Hae pääkirjasta tositteen tiedot
    // Huom: Netvisor API:ssa ei ole suoraa "get single voucher" endpointia,
    // joten haetaan pääkirjasta ja filtteröidään
    const vouchers = await client.accounting.accountingLedger({
      // Tarvitaan päivämääräväli - haetaan laajalta
      startDate: '2020-01-01',
      endDate: new Date().toISOString().split('T')[0],
      voucherStatus: 1
    });
    
    const voucher = vouchers.find(v => v.netvisorKey === netvisorKey);
    
    if (!voucher) {
      const { NotFoundError } = require('../errors');
      throw new NotFoundError(`Tositetta ${netvisorKey} ei löytynyt`);
    }
    
    return voucher;
  }
}

module.exports = { ApiDataProvider };
```

```javascript
// src/data/index.js
const { DataProvider } = require('./DataProvider');
const { ApiDataProvider } = require('./ApiDataProvider');

// Singleton instance
let dataProviderInstance = null;

/**
 * Factory function - palauttaa DataProvider-instanssin.
 * MVP: ApiDataProvider
 * Phase 2: CachedDataProvider (konfiguroidaan ympäristömuuttujalla)
 * 
 * @returns {DataProvider}
 */
function getDataProvider() {
  if (!dataProviderInstance) {
    // MVP: Aina ApiDataProvider
    // Phase 2: Tarkista NETVISOR_CACHE_ENABLED env
    dataProviderInstance = new ApiDataProvider();
  }
  return dataProviderInstance;
}

/**
 * Aseta custom DataProvider (testaukseen)
 * @param {DataProvider} provider
 */
function setDataProvider(provider) {
  dataProviderInstance = provider;
}

/**
 * Resetoi provider (testaukseen)
 */
function resetDataProvider() {
  dataProviderInstance = null;
}

module.exports = { 
  DataProvider, 
  ApiDataProvider, 
  getDataProvider, 
  setDataProvider,
  resetDataProvider
};
```

---

## 2. PÄIVITYS: Task-09 – netvisor_get_ledger

### Muutos

**Vanha:** Kutsuu suoraan `createClient()` ja `client.accounting.accountingLedger()`

**Uusi:** Käyttää `getDataProvider().getLedger()`

### Päivitetty koodirunko

```javascript
// src/tools/getLedger.js (PÄIVITETTY)
const { getDataProvider } = require('../data');
const { GetLedgerInputSchema } = require('../schemas/input');
const { transformVoucher } = require('../transformers/ledger');
const { ValidationError } = require('../errors');

async function handler(params) {
  // 1. Validoi parametrit
  const validation = GetLedgerInputSchema.safeParse(params);
  if (!validation.success) {
    throw new ValidationError(validation.error.issues[0].message);
  }
  
  const { organizationId, startDate, endDate, accountNumberStart, accountNumberEnd } = validation.data;
  
  // 2. Hae DataProvider
  const dataProvider = getDataProvider();
  
  // 3. Hae tilikartta (accountNumber mapping)
  const { accountMap } = await dataProvider.getAccounts(organizationId);
  
  // 4. Hae pääkirja DataProviderin kautta
  const vouchers = await dataProvider.getLedger(organizationId, startDate, endDate, {
    accountNumberStart,
    accountNumberEnd
  });
  
  // 5. Transformoi
  const rows = vouchers.flatMap(v => transformVoucher(v, accountMap));
  
  return {
    success: true,
    data: {
      rowCount: rows.length,
      voucherCount: vouchers.length,
      rows
    }
  };
}

module.exports = { definition, handler };
```

---

## 3. PÄIVITYS: Task-10 – netvisor_get_accounts

### Päivitetty koodirunko

```javascript
// src/tools/getAccounts.js (PÄIVITETTY)
const { getDataProvider } = require('../data');

async function handler(params) {
  // ... validointi ...
  
  const dataProvider = getDataProvider();
  const { accounts, defaultAccounts, accountMap } = await dataProvider.getAccounts(organizationId);
  
  return {
    success: true,
    data: {
      accountCount: accounts.length,
      accounts,
      defaultAccounts,
      // accountMap serialisoidaan objektiksi (Map ei serialisoidu JSON:ksi)
      accountMapping: Object.fromEntries(accountMap)
    }
  };
}
```

---

## 4. PÄIVITYS: Task-11 – netvisor_get_balances

### Päivitetty koodirunko

```javascript
// src/tools/getBalances.js (PÄIVITETTY)
const { getDataProvider } = require('../data');
const { transformBalances } = require('../transformers/balances');

async function handler(params) {
  // ... validointi ...
  
  const dataProvider = getDataProvider();
  
  // Hae tilikartta mappingia varten
  const { accountMap } = await dataProvider.getAccounts(organizationId);
  
  // Hae saldot
  const rawBalances = await dataProvider.getBalances(organizationId, balanceDates, intervalType);
  
  // Transformoi ja lisää accountNumber mapping
  const balances = transformBalances(rawBalances, accountMap);
  
  return {
    success: true,
    data: { balances }
  };
}
```

---

## 5. PÄIVITYS: Projektirakenne

```
netvisor-mcp/
├── src/
│   ├── data/                    # ← UUSI KANSIO
│   │   ├── DataProvider.js      # Abstract base class
│   │   ├── ApiDataProvider.js   # MVP: suora API-kutsu
│   │   └── index.js             # Factory + exports
│   │
│   ├── config.js
│   ├── client.js
│   ├── errors.js
│   ├── tools/
│   ├── schemas/
│   ├── transformers/
│   └── utils/
│
└── tests/
    ├── unit/
    │   ├── data/                # ← UUSI KANSIO
    │   │   ├── DataProvider.test.js
    │   │   └── ApiDataProvider.test.js
    │   └── ...
    └── ...
```

---

## 6. PÄIVITYS: Toteutusjärjestys

```
┌─────────────────────────────────────────────────────────────────┐
│  VAIHE 1: Infrastruktuuri (Task-01 – Task-05b)     +2h         │
│  ─────────────────────────────────────────────────────────────  │
│  Task-01: Konfiguraatio                                         │
│  Task-02: Error-luokat                                          │
│  Task-03: Zod-skeemat                                           │
│  Task-04: Retry-logiikka                                        │
│  Task-05: Client wrapper                                        │
│  Task-05b: DataProvider interface  ← UUSI                       │
│                                                                 │
│  VAIHE 2: Transformerit (Task-06 – Task-08)                    │
│  ─────────────────────────────────────────────────────────────  │
│  (ei muutoksia)                                                 │
│                                                                 │
│  VAIHE 3: MCP Tools (Task-09 – Task-14)                        │
│  ─────────────────────────────────────────────────────────────  │
│  Task-10: getAccounts             (käyttää DataProvideria)      │
│  Task-09: getLedger               (käyttää DataProvideria)      │
│  Task-11: getBalances             (käyttää DataProvideria)      │
│  Task-12: getDimensions           (käyttää DataProvideria)      │
│  Task-13: getVoucher              (käyttää DataProvideria)      │
│  Task-14: MCP Server                                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. PÄIVITYS: Traceability Matrix (lisäykset)

| REQ | AC | Task | Test Scenario | Status |
|-----|-----|------|---------------|--------|
| - | (Phase 2 prep) | Task-05b | TS-05b.1 – TS-05b.6 | 🔲 |

---

## 8. Phase 2 Roadmap (tiedoksi)

Kun MVP on valmis, SQLite-cache lisätään näillä taskeilla:

| Task | Kuvaus | Arvio |
|------|--------|-------|
| Task-15 | SQLite schema (ledger, accounts, balances, metadata) | 3h |
| Task-16 | CachedDataProvider extends DataProvider | 4h |
| Task-17 | Cache invalidation (TTL, manual refresh) | 2h |
| Task-18 | Tool: `netvisor_cache_status` | 1h |
| Task-19 | Tool: `netvisor_refresh_cache` | 1h |
| Task-20 | Tool: `netvisor_compare_years` | 2h |

**Phase 2 yhteensä:** ~13h

---

## 9. GEMINI PEER REVIEW -KORJAUKSET (v1.2)

### 9.1 Task-03: Zod-skeemat v4.0.0 -yhteensopiviksi [Gemini #1]

**Ongelma:** Output-skeemat eivät tue Netvisor v4.0.0:n wrapper-rakennetta (AccountBalances).

**Korjaus:** Päivitä `src/schemas/output.js` tukemaan molempia rakenteita:

```javascript
// src/schemas/output.js (PÄIVITETTY)
const { z } = require('zod');

// AccountBalance voi tulla kahdessa muodossa (pre-4.0.0 ja 4.0.0+)
const AccountBalanceItemSchema = z.object({
  account: z.object({
    attr: z.object({ netvisorkey: z.number() }),
    accountbalance: z.array(z.object({
      attr: z.object({ date: z.string() }),
      debet: z.number().optional().default(0),
      kredit: z.number().optional().default(0),
      balance: z.number().optional().default(0)
    })).optional().default([])
  })
});

// v4.0.0+ wrapper-rakenne
const AccountBalanceResponseSchema = z.union([
  // Uusi rakenne (v4.0.0+): AccountBalances wrapper
  z.object({
    AccountBalances: z.object({
      accountBalances: z.array(AccountBalanceItemSchema).optional().default([])
    })
  }),
  // Vanha rakenne (pre-4.0.0): suoraan accountBalances
  z.object({
    accountBalances: z.array(AccountBalanceItemSchema).optional().default([])
  })
]);

module.exports = { AccountBalanceResponseSchema, AccountBalanceItemSchema };
```

**Uusi Test Scenario:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-03.10 | HP | API-vastaus v4.0.0 wrapper-rakenteella → parsitaan onnistuneesti |
| TS-03.11 | HP | API-vastaus vanhalla rakenteella → parsitaan onnistuneesti |

---

### 9.2 Task-04: Retry-viiveet MCP-yhteensopiviksi [Gemini #4]

**Ongelma:** 60s/120s/300s viiveet voivat katkaista Claude Desktop MCP-yhteyden.

**Korjaus:** Lyhennä viiveet ja palauta virhe nopeasti:

```javascript
// src/utils/retry.js (PÄIVITETTY)

// VANHA: const DEFAULT_DELAYS = [60000, 120000, 300000];
// UUSI: Lyhyemmät viiveet MCP-yhteensopivuuteen
const DEFAULT_DELAYS = [5000, 15000, 30000]; // 5s, 15s, 30s

async function withRetry(operation, options = {}) {
  const {
    maxRetries = 3,
    delays = DEFAULT_DELAYS,
    onRetry = () => {},
    isRetryable = (error) => error.status === 429 || 
                            error.message?.includes('rate limit')
  } = options;

  let lastError;
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error;
      
      if (!isRetryable(error) || attempt === maxRetries) {
        throw error;
      }
      
      const delayMs = delays[Math.min(attempt, delays.length - 1)];
      onRetry(attempt + 1, delayMs, error);
      
      // Palauta heti virhe jos viive olisi yli 30s
      // → käyttäjä voi yrittää uudelleen manuaalisesti
      if (delayMs > 30000) {
        throw new RateLimitError(
          `Netvisor rajoittaa pyyntöjä. Yritä uudelleen ${Math.ceil(delayMs/1000)} sekunnin kuluttua.`,
          delayMs
        );
      }
      
      await delay(delayMs);
    }
  }
  
  throw lastError;
}
```

**Päivitetyt Test Scenariot:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-04.4 | HP | ~~Ensimmäinen odotus 60s, toinen 120s, kolmas 300s~~ → Ensimmäinen 5s, toinen 15s, kolmas 30s |
| TS-04.7 | HP | UUSI: Viive yli 30s → RateLimitError heti käyttäjälle |

---

### 9.3 Task-07: getBalances hakee accountMap:n sisäisesti [Gemini #2]

**Ongelma:** ApiDataProvider.getBalances() ei hae accountMap:ia, joten tilinumerot puuttuvat.

**Korjaus:** getBalances kutsuu sisäisesti getAccounts():

```javascript
// src/data/ApiDataProvider.js (PÄIVITETTY getBalances)

async getBalances(organizationId, balanceDates, intervalType = 0) {
  const client = createClient(organizationId);
  
  // 1. Hae accountMap tilinumero-mappingia varten
  const { accountMap } = await this.getAccounts(organizationId);
  
  // 2. Hae saldot API:sta
  const response = await client.accounting.getAccountBalance({
    balanceDates,
    intervalType
  });
  
  // 3. Palauta sekä raw-data että accountMap transformeria varten
  return {
    rawBalances: response,
    accountMap
  };
}
```

**Päivitetty Task-11 handler:**

```javascript
// src/tools/getBalances.js (PÄIVITETTY)
async function handler(params) {
  // ... validointi ...
  
  const dataProvider = getDataProvider();
  
  // getBalances palauttaa nyt { rawBalances, accountMap }
  const { rawBalances, accountMap } = await dataProvider.getBalances(
    organizationId, 
    balanceDates, 
    intervalType
  );
  
  // Transformoi ja lisää accountNumber mapping
  const balances = transformBalances(rawBalances, accountMap);
  
  return {
    success: true,
    data: { balances }
  };
}
```

---

### 9.4 Task-08: Dimensioiden sumea tunnistus [Gemini #5]

**Ongelma:** Kovakoodatut dimensionimet ("Kustannuspaikka", "Projekti") eivät tunnista variaatioita.

**Korjaus:** Implementoi sumea tunnistus:

```javascript
// src/transformers/dimensions.js (PÄIVITETTY)

// Avainsanat dimensiotyyppien tunnistamiseen (case-insensitive)
const COST_CENTER_KEYWORDS = [
  'kustannuspaikka', 'kustannusp', 'k-paikka', 'kp', 
  'cost center', 'costcenter', 'cc',
  'osasto', 'department'
];

const PROJECT_KEYWORDS = [
  'projekti', 'project', 'proj', 'hanke'
];

function identifyDimensionType(name) {
  const normalized = name.toLowerCase().trim();
  
  // Tarkista kustannuspaikka-avainsanat
  if (COST_CENTER_KEYWORDS.some(kw => normalized.includes(kw))) {
    return 'costCenter';
  }
  
  // Tarkista projekti-avainsanat
  if (PROJECT_KEYWORDS.some(kw => normalized.includes(kw))) {
    return 'project';
  }
  
  // Tuntematon → custom
  return 'custom';
}

function transformDimensions(dimensionList) {
  const result = {
    costCenters: [],
    projects: [],
    custom: {}
  };

  for (const dim of dimensionList) {
    const items = (dim.dimensionDetails?.dimensionDetail || []).map(d => ({
      netvisorKey: d.netvisorKey,
      name: d.name,
      isHidden: d.isHidden,
      level: d.level,
      parentId: d.fatherId || null
    }));

    const dimType = identifyDimensionType(dim.name || '');
    
    switch (dimType) {
      case 'costCenter':
        result.costCenters = items;
        break;
      case 'project':
        result.projects = items;
        break;
      default:
        // Custom: säilytä alkuperäinen nimi avaimena
        result.custom[dim.name] = items;
    }
  }

  return result;
}

module.exports = { transformDimensions, identifyDimensionType };
```

**Uudet Test Scenariot:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-08.6 | HP | Dimension name "K-paikka" → costCenters-listaan |
| TS-08.7 | HP | Dimension name "Osasto" → costCenters-listaan |
| TS-08.8 | HP | Dimension name "Hanke" → projects-listaan |
| TS-08.9 | EC | Dimension name "Tulosyksikkö" (tuntematon) → custom-objektiin |

---

### 9.5 Task-13: getVoucher käyttää suoraa API-endpointia [Gemini #3]

**Ongelma:** getVoucher hakee koko pääkirjan ja filtteröi – tehoton ja timeout-riski.

**Korjaus:** Käytä suoraa getvoucher.nv-endpointia:

```javascript
// src/data/ApiDataProvider.js (PÄIVITETTY getVoucher)

async getVoucher(organizationId, netvisorKey) {
  const client = createClient(organizationId);
  
  // VANHA (TEHOTON):
  // const vouchers = await client.accounting.accountingLedger({...});
  // return vouchers.find(v => v.netvisorKey === netvisorKey);
  
  // UUSI: Suora endpoint
  try {
    // Tarkista tukeeko kirjasto suoraa hakua
    if (typeof client.accounting.getVoucher === 'function') {
      return await client.accounting.getVoucher({ netvisorKey });
    }
    
    // Fallback: Käytä REST API:a suoraan jos metodia ei ole
    const response = await client.request({
      method: 'GET',
      endpoint: 'getvoucher.nv',
      params: { netvisorKey }
    });
    
    return response;
  } catch (error) {
    if (error.status === 404 || error.message?.includes('not found')) {
      const { NotFoundError } = require('../errors');
      throw new NotFoundError(`Tositetta ${netvisorKey} ei löytynyt`);
    }
    throw error;
  }
}
```

**Huomio toteutukseen:** Tarkista @rantalainen/netvisor-api-client -kirjaston dokumentaatiosta tukeeko se `getVoucher`-metodia. Jos ei, implementoi suora REST-kutsu tai avaa issue kirjaston GitHub-repoon.

**Päivitetyt Test Scenariot:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-13.5 | PF | UUSI: getVoucher vastausaika < 5s (ei koko pääkirjan hakua) |

---

## 10. Gemini-review: Positiiviset huomiot

| Huomio | Kommentti |
|--------|-----------|
| **DataProvider-abstraktio** | "Projektin vahvin lenkki" - mahdollistaa Phase 2 ilman refaktorointia ✅ |
| **Summat stringeinä** | Hyvä ratkaisu JSON-serialisoinnin kannalta ✅ |
| **Suomenkieliset virheet** | Erinomaista Claude-käyttäjäkokemuksen kannalta ✅ |

**Lisäsuositus (token-optimointi):** Jätä tyhjät kentät pois JSON-vastauksesta:

```javascript
// src/utils/format.js
function removeEmptyFields(obj) {
  return Object.fromEntries(
    Object.entries(obj).filter(([_, v]) => 
      v !== null && v !== undefined && v !== '' && 
      !(Array.isArray(v) && v.length === 0)
    )
  );
}
```

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.2 | 2025-12-18 | Gemini peer review -korjaukset: Task-03, 04, 07, 08, 13 |
| 1.1 | 2025-12-18 | DataProvider-abstraktio SQLite-välimuistia varten |
| 1.0 | 2025-12-18 | Alkuperäinen TECH_SPEC |

---

## Liittyvät dokumentit

| Dokumentti | Yhteys |
|------------|--------|
| TECH_SPEC_Netvisor_MCP.md v1.0 | Päädokumentti johon tämä addendum liittyy |
| SPEC_Netvisor_MCP.md | Toiminnalliset vaatimukset (ei muutoksia) |

---

*Claude Code: Lue tämä addendum YHDESSÄ alkuperäisen TECH_SPEC_Netvisor_MCP.md kanssa.*
