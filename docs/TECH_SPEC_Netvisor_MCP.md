# TECH_SPEC_Netvisor_MCP

> **Versio:** 1.0  
> **Päivitetty:** 2025-12-18  
> **Status:** Draft  
> **Projekti:** Numbers Tilintarkastaja-Controller  
> **Perustuu:** SPEC_Netvisor_MCP.md v1.0

---

## 1. Yleiskuvaus

### 1.1 Tekninen lähestymistapa

Netvisor MCP Server toteutetaan Node.js + CommonJS -stackilla, hyödyntäen olemassa olevaa MCP-infrastruktuuria. Palvelin on **stateless** ja käyttää `@rantalainen/netvisor-api-client` -kirjastoa Netvisor-kommunikaatioon.

### 1.2 Arkkitehtuuripäätökset

| Päätös | Valinta | Perustelu |
|--------|---------|-----------|
| Runtime | Node.js + CommonJS | Yhteensopiva olemassa olevan MCP-infrastruktuurin kanssa |
| Netvisor-kirjasto | @rantalainen/netvisor-api-client v4.1.1 | Aktiivisesti ylläpidetty, TypeScript-tyypit |
| Validointi | Zod | Runtime-validointi, hyvät virheviestit |
| Rahasummat | String (ei Decimal.js) | JSON-serialisoitavuus, tarkkuus säilyy |
| Virheenkäsittely | Custom Error classes | Selkeä virhetyypitys |

---

## 2. Teknologiavalinnat

| Komponentti | Teknologia | Versio | Perustelu |
|-------------|------------|--------|-----------|
| Runtime | Node.js | >=18.0.0 | LTS, ES2022 tuki |
| MCP SDK | @modelcontextprotocol/sdk | latest | Claude Desktop -yhteensopivuus |
| Netvisor Client | @rantalainen/netvisor-api-client | ^4.1.1 | Virallinen TypeScript client |
| Validointi | zod | ^3.22.0 | Schema validation |
| Env-muuttujat | dotenv | ^16.0.0 | .env-tiedoston luku |
| Testaus | Jest | ^29.0.0 | Yksikkö- ja integraatiotestit |

---

## 3. Projektirakenne

```
netvisor-mcp/
├── index.js                 # MCP Server entry point
├── package.json
├── .env.example             # Esimerkki ympäristömuuttujista
├── .env                     # (gitignore) Todelliset tunnukset
│
├── src/
│   ├── config.js            # Konfiguraation lataus ja validointi
│   ├── client.js            # NetvisorApiClient wrapper
│   ├── errors.js            # Custom error classes
│   │
│   ├── tools/
│   │   ├── index.js         # Tool-rekisteröinti
│   │   ├── getLedger.js     # netvisor_get_ledger
│   │   ├── getAccounts.js   # netvisor_get_accounts
│   │   ├── getBalances.js   # netvisor_get_balances
│   │   ├── getDimensions.js # netvisor_get_dimensions
│   │   └── getVoucher.js    # netvisor_get_voucher
│   │
│   ├── schemas/
│   │   ├── input.js         # Zod-skeemat tool-parametreille
│   │   └── output.js        # Zod-skeemat API-vastauksille
│   │
│   ├── transformers/
│   │   ├── ledger.js        # Pääkirjan normalisointi
│   │   ├── accounts.js      # Tilikartan käsittely
│   │   ├── balances.js      # Saldojen käsittely
│   │   └── dimensions.js    # Dimensioiden käsittely
│   │
│   └── utils/
│       ├── retry.js         # Exponential backoff
│       └── format.js        # Virheviestien muotoilu
│
└── tests/
    ├── unit/
    │   ├── config.test.js
    │   ├── schemas.test.js
    │   ├── transformers.test.js
    │   └── retry.test.js
    │
    ├── integration/
    │   ├── tools.test.js
    │   └── client.test.js
    │
    └── fixtures/
        ├── ledger-response.json
        ├── accounts-response.json
        ├── balances-response.json
        └── dimensions-response.json
```

---

## 4. Task Decomposition

### Task-01: Projektin alustus ja konfiguraatio

**Toteuttaa:** AC-23, AC-24, AC-25, AC-26, AC-27  
**Arvioitu kesto:** 2h  
**Prioriteetti:** MVP (ensimmäisenä)

**Kuvaus:**
Luo projektirakenne, package.json, ja konfiguraation lataava moduuli. Konfiguraatio luetaan ympäristömuuttujista dotenv-kirjastolla.

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-01.1 | HP | Kaikki env-muuttujat asetettu → config-objekti palautetaan |
| TS-01.2 | ER | NETVISOR_CUSTOMER_ID puuttuu → ConfigurationError |
| TS-01.3 | ER | NETVISOR_CUSTOMER_KEY puuttuu → ConfigurationError |
| TS-01.4 | ER | NETVISOR_PARTNER_ID puuttuu → ConfigurationError |
| TS-01.5 | ER | NETVISOR_PARTNER_KEY puuttuu → ConfigurationError |
| TS-01.6 | HP | NETVISOR_TIMEOUT puuttuu → käytetään oletusta 120000 |
| TS-01.7 | HP | Config ei sisällä avaimia toString():ssa tai JSON.stringify():ssä |

**Koodirunko:**

```javascript
// src/config.js
const { z } = require('zod');
require('dotenv').config();

const ConfigSchema = z.object({
  customerId: z.string().min(1, 'NETVISOR_CUSTOMER_ID vaaditaan'),
  customerKey: z.string().min(1, 'NETVISOR_CUSTOMER_KEY vaaditaan'),
  partnerId: z.string().min(1, 'NETVISOR_PARTNER_ID vaaditaan'),
  partnerKey: z.string().min(1, 'NETVISOR_PARTNER_KEY vaaditaan'),
  timeout: z.number().default(120000),
  integrationName: z.string().default('Numbers Tilintarkastaja')
});

class ConfigurationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ConfigurationError';
  }
}

function loadConfig() {
  // TODO: Implementoi
}

module.exports = { loadConfig, ConfigurationError, ConfigSchema };
```

---

### Task-02: Custom Error -luokat

**Toteuttaa:** AC-28, AC-29, AC-31, AC-32, AC-33, AC-35, AC-38  
**Arvioitu kesto:** 1h  
**Prioriteetti:** MVP

**Kuvaus:**
Luo virheluokat eri virhetilanteille. Jokainen virheluokka sisältää suomenkielisen viestin ja mahdollisen retry-ohjeen.

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-02.1 | HP | AuthenticationError sisältää viestin tunnuksista |
| TS-02.2 | HP | RateLimitError sisältää odotusajan sekunteina |
| TS-02.3 | HP | TimeoutError ehdottaa pienempää aikaväliä |
| TS-02.4 | HP | ValidationError sisältää virheellisen kentän nimen |
| TS-02.5 | HP | NetworkError sisältää alkuperäisen virheen |
| TS-02.6 | HP | NotFoundError sisältää puuttuvan resurssin tunnisteen |

**Koodirunko:**

```javascript
// src/errors.js
class NetvisorError extends Error {
  constructor(message, code, details = {}) {
    super(message);
    this.name = 'NetvisorError';
    this.code = code;
    this.details = details;
  }

  toJSON() {
    return {
      error: this.name,
      code: this.code,
      message: this.message,
      details: this.details
    };
  }
}

class AuthenticationError extends NetvisorError { /* TODO */ }
class RateLimitError extends NetvisorError { /* TODO */ }
class TimeoutError extends NetvisorError { /* TODO */ }
class ValidationError extends NetvisorError { /* TODO */ }
class NetworkError extends NetvisorError { /* TODO */ }
class NotFoundError extends NetvisorError { /* TODO */ }

module.exports = {
  NetvisorError,
  AuthenticationError,
  RateLimitError,
  TimeoutError,
  ValidationError,
  NetworkError,
  NotFoundError
};
```

---

### Task-03: Zod-validointiskeemat

**Toteuttaa:** AC-02, AC-13, AC-34, AC-36, AC-37  
**Arvioitu kesto:** 3h  
**Prioriteetti:** MVP

**Kuvaus:**
Luo Zod-skeemat sekä tool-parametrien validointiin (input) että API-vastausten validointiin (output). Skeemat dokumentoivat odotetut tietorakenteet.

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-03.1 | HP | Validi organizationId "1234567-8" → läpäisee |
| TS-03.2 | ER | Virheellinen organizationId "123" → ValidationError |
| TS-03.3 | HP | Validi startDate "2024-01-01" → läpäisee |
| TS-03.4 | ER | Virheellinen startDate "01-01-2024" → ValidationError |
| TS-03.5 | HP | Validi balanceDates "2024-01-01,2024-12-31" → läpäisee |
| TS-03.6 | ER | Virheellinen balanceDates "2024-01-01" (yksi pvm) → ValidationError |
| TS-03.7 | HP | API-vastaus validilla rakenteella → parsitaan onnistuneesti |
| TS-03.8 | EC | API-vastaus puuttuvalla kentällä → käytetään oletusarvoa |
| TS-03.9 | ER | API-vastaus täysin väärällä rakenteella → UnexpectedResponseError |

**Koodirunko:**

```javascript
// src/schemas/input.js
const { z } = require('zod');

const OrganizationIdSchema = z.string()
  .regex(/^\d{7}-\d$/, 'Y-tunnus muodossa 1234567-8');

const DateSchema = z.string()
  .regex(/^\d{4}-\d{2}-\d{2}$/, 'Päivämäärä muodossa YYYY-MM-DD');

const BalanceDatesSchema = z.string()
  .regex(/^\d{4}-\d{2}-\d{2},\d{4}-\d{2}-\d{2}$/, 
    'Päivämäärät muodossa YYYY-MM-DD,YYYY-MM-DD');

const GetLedgerInputSchema = z.object({
  organizationId: OrganizationIdSchema,
  startDate: DateSchema,
  endDate: DateSchema,
  accountNumberStart: z.number().optional(),
  accountNumberEnd: z.number().optional()
});

// TODO: Lisää muut skeemat

module.exports = {
  OrganizationIdSchema,
  DateSchema,
  BalanceDatesSchema,
  GetLedgerInputSchema
};
```

```javascript
// src/schemas/output.js
const { z } = require('zod');

const VoucherLineSchema = z.object({
  netvisorKey: z.number(),
  lineSum: z.number(),
  accountNumber: z.number(),
  vatPercent: z.number().optional().default(0),
  vatCode: z.string().optional().default('NONE'),
  description: z.string().optional().default(''),
  accountDimension: z.object({
    value: z.string(),
    attr: z.object({ netvisorkey: z.number() })
  }).optional(),
  dimension: z.array(z.object({
    dimensionName: z.string(),
    dimensionItem: z.string()
  })).optional()
});

const VoucherSchema = z.object({
  status: z.enum(['valid', 'invalidated', 'deleted']),
  netvisorKey: z.number(),
  voucherDate: z.string(),
  voucherNumber: z.number(),
  voucherDescription: z.string().optional().default(''),
  voucherClass: z.string(),
  voucherLine: z.array(VoucherLineSchema)
});

// TODO: Lisää AccountBalance, Account, Dimension skeemat

module.exports = { VoucherSchema, VoucherLineSchema };
```

---

### Task-04: Retry-logiikka (Exponential Backoff)

**Toteuttaa:** AC-29, AC-30  
**Arvioitu kesto:** 2h  
**Prioriteetti:** MVP

**Kuvaus:**
Implementoi withRetry-wrapper joka käsittelee rate limiting -tilanteet exponential backoff -strategialla (60s → 120s → 300s).

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-04.1 | HP | Ensimmäinen kutsu onnistuu → palautetaan tulos |
| TS-04.2 | HP | Ensimmäinen epäonnistuu (429), toinen onnistuu → palautetaan tulos |
| TS-04.3 | HP | Kolme epäonnistumista (429) → RateLimitError |
| TS-04.4 | HP | Ensimmäinen odotus 60s, toinen 120s, kolmas 300s |
| TS-04.5 | EC | Ei-429 virhe → ei retry:tä, heitetään heti |
| TS-04.6 | HP | onRetry-callback kutsutaan joka retry-kerralla |

**Koodirunko:**

```javascript
// src/utils/retry.js
const { RateLimitError } = require('../errors');

const DEFAULT_DELAYS = [60000, 120000, 300000]; // 1min, 2min, 5min

async function withRetry(operation, options = {}) {
  const {
    maxRetries = 3,
    delays = DEFAULT_DELAYS,
    onRetry = () => {},
    isRetryable = (error) => error.status === 429 || 
                            error.message?.includes('rate limit')
  } = options;

  // TODO: Implementoi
}

function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

module.exports = { withRetry, delay };
```

---

### Task-05: NetvisorApiClient Wrapper

**Toteuttaa:** AC-05, AC-25, AC-27  
**Arvioitu kesto:** 2h  
**Prioriteetti:** MVP

**Kuvaus:**
Luo wrapper NetvisorApiClient:lle joka käyttää konfiguraatiota, lisää retry-logiikan, ja varmistaa ettei tunnuksia vuoda.

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-05.1 | HP | createClient(orgId) → palauttaa toimivan clientin |
| TS-05.2 | HP | Client käyttää konfiguraation tunnuksia |
| TS-05.3 | HP | Client käyttää konfiguraation timeoutia |
| TS-05.4 | ER | API-virhe 401 → AuthenticationError |
| TS-05.5 | ER | API-virhe 429 → retry-logiikka aktivoituu |
| TS-05.6 | ER | Verkkovirhe → NetworkError |
| TS-05.7 | HP | Client.toString() ei paljasta tunnuksia |

**Koodirunko:**

```javascript
// src/client.js
const { NetvisorApiClient } = require('@rantalainen/netvisor-api-client');
const { loadConfig } = require('./config');
const { withRetry } = require('./utils/retry');
const { AuthenticationError, NetworkError } = require('./errors');

function createClient(organizationId) {
  const config = loadConfig();
  
  const client = new NetvisorApiClient({
    integrationName: config.integrationName,
    customerId: config.customerId,
    customerKey: config.customerKey,
    partnerId: config.partnerId,
    partnerKey: config.partnerKey,
    organizationId,
    timeout: config.timeout
  });

  // TODO: Wrappaa metodit retry-logiikalla
  return client;
}

module.exports = { createClient };
```

---

### Task-06: Pääkirjan transformer

**Toteuttaa:** AC-03  
**Arvioitu kesto:** 2h  
**Prioriteetti:** MVP

**Kuvaus:**
Transformoi Netvisorin pääkirjavastaus normalisoituun muotoon. Erottaa debet/kredit, parsii dimensiot, lisää tilikartan nimet.

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-06.1 | HP | lineSum > 0 → debit = lineSum, credit = "0" |
| TS-06.2 | HP | lineSum < 0 → debit = "0", credit = abs(lineSum) |
| TS-06.3 | HP | lineSum = 0 → debit = "0", credit = "0" |
| TS-06.4 | HP | accountDimension → dimensions.costCenter |
| TS-06.5 | HP | dimension[].dimensionName = "Projekti" → dimensions.project |
| TS-06.6 | HP | dimension[].dimensionName = tuntematon → dimensions.custom |
| TS-06.7 | EC | Puuttuva description → tyhjä string |
| TS-06.8 | HP | Summat palautetaan stringeinä (ei Decimal-objekteina) |

**Koodirunko:**

```javascript
// src/transformers/ledger.js

function transformVoucher(voucher, accountMap) {
  return voucher.voucherLine.map(line => transformLine(voucher, line, accountMap));
}

function transformLine(voucher, line, accountMap) {
  const lineSum = line.lineSum || 0;
  const account = accountMap.get(line.accountNumber);
  
  return {
    // Tosite
    voucherNetvisorKey: voucher.netvisorKey,
    voucherDate: voucher.voucherDate,
    voucherNumber: voucher.voucherNumber,
    voucherDescription: voucher.voucherDescription || '',
    voucherClass: voucher.voucherClass,
    
    // Rivi
    lineNetvisorKey: line.netvisorKey,
    accountNetvisorKey: line.accountNumber, // Netvisorissa accountNumber on int
    accountNumber: String(line.accountNumber),
    accountName: account?.name || '',
    debit: lineSum > 0 ? String(lineSum) : '0',
    credit: lineSum < 0 ? String(Math.abs(lineSum)) : '0',
    description: line.description || '',
    
    // ALV
    vatPercent: line.vatPercent || 0,
    vatCode: line.vatCode || 'NONE',
    
    // Dimensiot
    dimensions: extractDimensions(line)
  };
}

function extractDimensions(line) {
  // TODO: Implementoi peer review:n suositusten mukaan
}

module.exports = { transformVoucher, transformLine, extractDimensions };
```

---

### Task-07: Tilisaldojen transformer

**Toteuttaa:** AC-15, AC-16, AC-17  
**Arvioitu kesto:** 2h  
**Prioriteetti:** MVP

**Kuvaus:**
Transformoi Netvisorin tilisaldovastaus, lisää accountNumber mappingilla. **KRIITTINEN:** netvisorKey ≠ accountNumber!

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-07.1 | HP | v4.0.0+ AccountBalances wrapper käsitellään oikein |
| TS-07.2 | HP | netvisorKey mappaa accountNumber:iin accountMapin kautta |
| TS-07.3 | HP | accountName lisätään accountMapista |
| TS-07.4 | EC | Tuntematon netvisorKey → accountNumber = null, varoitus |
| TS-07.5 | HP | Saldot palautetaan stringeinä |
| TS-07.6 | HP | Useita balanceDates → kaikki saldopäivät mukana |

**Koodirunko:**

```javascript
// src/transformers/balances.js

function transformBalances(apiResponse, accountMap) {
  // HUOM: v4.0.0 muutti vastausrakennetta!
  // Vanha: apiResponse.accountBalances
  // Uusi:  apiResponse.AccountBalances.accountBalances
  
  const balances = apiResponse.AccountBalances?.accountBalances 
                || apiResponse.accountBalances 
                || [];
  
  return balances.map(item => {
    const netvisorKey = item.account?.attr?.netvisorkey;
    const account = accountMap.get(netvisorKey);
    
    if (!account) {
      console.warn(`Tuntematon netvisorKey: ${netvisorKey}`);
    }
    
    return {
      netvisorKey,
      accountNumber: account?.number || null,
      accountName: account?.name || '',
      balances: (item.account?.accountbalance || []).map(bal => ({
        date: bal.attr?.date,
        debit: String(bal.debet || 0),
        credit: String(bal.kredit || 0),
        balance: String(bal.balance || 0)
      }))
    };
  });
}

module.exports = { transformBalances };
```

---

### Task-08: Dimensioiden transformer

**Toteuttaa:** AC-19, AC-20  
**Arvioitu kesto:** 1h  
**Prioriteetti:** MVP

**Kuvaus:**
Transformoi Netvisorin dimensiolista strukturoituun muotoon: costCenters, projects, custom.

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-08.1 | HP | Dimension name "Kustannuspaikka" → costCenters-listaan |
| TS-08.2 | HP | Dimension name "Projekti" → projects-listaan |
| TS-08.3 | HP | Tuntematon dimension name → custom-objektiin |
| TS-08.4 | EC | Tyhjä dimensiolista → tyhjät taulukot |
| TS-08.5 | HP | Hierarkinen dimensio (fatherId) säilytetään |

**Koodirunko:**

```javascript
// src/transformers/dimensions.js

function transformDimensions(dimensionList) {
  const result = {
    costCenters: [],
    projects: [],
    custom: {}
  };

  for (const dim of dimensionList) {
    const name = dim.name?.toLowerCase() || '';
    const items = (dim.dimensionDetails?.dimensionDetail || []).map(d => ({
      netvisorKey: d.netvisorKey,
      name: d.name,
      isHidden: d.isHidden,
      level: d.level,
      parentId: d.fatherId || null
    }));

    if (name.includes('kustannuspaikka') || name.includes('cost center')) {
      result.costCenters = items;
    } else if (name.includes('projekti') || name.includes('project')) {
      result.projects = items;
    } else {
      result.custom[dim.name] = items;
    }
  }

  return result;
}

module.exports = { transformDimensions };
```

---

### Task-09: MCP Tool - netvisor_get_ledger

**Toteuttaa:** AC-01, AC-02, AC-03, AC-04, AC-05, AC-06  
**Arvioitu kesto:** 3h  
**Prioriteetti:** MVP

**Kuvaus:**
Implementoi MCP-tool pääkirjan hakuun. Yhdistää validoinnin, API-kutsun, ja transformaation.

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-09.1 | HP | Validi kutsu → palauttaa transformoidut rivit |
| TS-09.2 | HP | Tool on rekisteröity MCP-serveriin |
| TS-09.3 | ER | Virheellinen organizationId → ValidationError-viesti |
| TS-09.4 | ER | Virheellinen startDate → ValidationError-viesti |
| TS-09.5 | ER | API 401 → "Tarkista Netvisor-tunnukset" |
| TS-09.6 | EC | Tyhjä vastaus → tyhjä lista, ei virhettä |
| TS-09.7 | HP | accountNumberStart/End filtteröi vastauksen |

**Koodirunko:**

```javascript
// src/tools/getLedger.js
const { createClient } = require('../client');
const { GetLedgerInputSchema } = require('../schemas/input');
const { transformVoucher } = require('../transformers/ledger');
const { ValidationError } = require('../errors');

const definition = {
  name: 'netvisor_get_ledger',
  description: 'Hae pääkirja (tositteet ja rivit) Netvisorista',
  inputSchema: {
    type: 'object',
    properties: {
      organizationId: { type: 'string', description: 'Y-tunnus (1234567-8)' },
      startDate: { type: 'string', description: 'Alkupäivä (YYYY-MM-DD)' },
      endDate: { type: 'string', description: 'Loppupäivä (YYYY-MM-DD)' },
      accountNumberStart: { type: 'number', description: 'Tilinumero alku (valinnainen)' },
      accountNumberEnd: { type: 'number', description: 'Tilinumero loppu (valinnainen)' }
    },
    required: ['organizationId', 'startDate', 'endDate']
  }
};

async function handler(params) {
  // 1. Validoi parametrit
  const validation = GetLedgerInputSchema.safeParse(params);
  if (!validation.success) {
    throw new ValidationError(validation.error.issues[0].message);
  }
  
  const { organizationId, startDate, endDate, accountNumberStart, accountNumberEnd } = validation.data;
  
  // 2. Hae tilikartta (accountNumber mapping)
  // TODO: Kutsu getAccounts ensin
  
  // 3. Hae pääkirja
  const client = createClient(organizationId);
  const vouchers = await client.accounting.accountingLedger({
    startDate,
    endDate,
    accountNumberStart,
    accountNumberEnd,
    voucherStatus: 1 // Vain voimassaolevat
  });
  
  // 4. Transformoi
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

### Task-10: MCP Tool - netvisor_get_accounts

**Toteuttaa:** AC-07, AC-08, AC-09, AC-10, AC-11  
**Arvioitu kesto:** 2h  
**Prioriteetti:** MVP

**Kuvaus:**
Implementoi MCP-tool tilikartan hakuun. Palauttaa myös netvisorKey → accountNumber mappingin.

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-10.1 | HP | Validi kutsu → palauttaa tilikartan |
| TS-10.2 | HP | Vastaus sisältää companyDefaultAccounts |
| TS-10.3 | HP | Vastaus sisältää mapping: netvisorKey → {number, name} |
| TS-10.4 | HP | Inaktiiviset tilit mukana isActive: false -merkinnällä |
| TS-10.5 | HP | accountType erottelee account/accountgroup |

---

### Task-11: MCP Tool - netvisor_get_balances

**Toteuttaa:** AC-12, AC-13, AC-14, AC-15, AC-16, AC-17  
**Arvioitu kesto:** 3h  
**Prioriteetti:** MVP

**Kuvaus:**
Implementoi MCP-tool tilisaldojen hakuun. **KRIITTINEN:** Käyttää accountMap:ia netvisorKey → accountNumber mappingiin.

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-11.1 | HP | Validi kutsu → palauttaa saldot accountNumber:lla |
| TS-11.2 | HP | intervalType=3 → kuukausisaldot |
| TS-11.3 | HP | intervalType=4 → vuosisaldot |
| TS-11.4 | ER | Virheellinen balanceDates → ValidationError |
| TS-11.5 | HP | **KRIITTINEN:** netvisorKey ei sekoitu accountNumber:iin |
| TS-11.6 | HP | Hakee tilikartan ensin mappingia varten |

---

### Task-12: MCP Tool - netvisor_get_dimensions

**Toteuttaa:** AC-18, AC-19, AC-20, AC-21, AC-22  
**Arvioitu kesto:** 2h  
**Prioriteetti:** MVP

**Kuvaus:**
Implementoi MCP-tool dimensioiden hakuun.

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-12.1 | HP | Validi kutsu → palauttaa strukturoidut dimensiot |
| TS-12.2 | HP | showHidden=true → piilotetut mukana |
| TS-12.3 | HP | showHidden=false (oletus) → vain näkyvät |
| TS-12.4 | EC | Tyhjä dimensiolista → tyhjät taulukot |

---

### Task-13: MCP Tool - netvisor_get_voucher

**Toteuttaa:** AC-39, AC-40, AC-41, AC-42  
**Arvioitu kesto:** 2h  
**Prioriteetti:** MVP

**Kuvaus:**
Implementoi MCP-tool yksittäisen tositteen hakuun.

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-13.1 | HP | Validi netvisorKey → palauttaa tositteen |
| TS-13.2 | HP | Vastaus sisältää kaikki rivit |
| TS-13.3 | ER | Olematon netvisorKey → NotFoundError |
| TS-13.4 | ER | Virheellinen netvisorKey tyyppi → ValidationError |

---

### Task-14: MCP Server -integraatio

**Toteuttaa:** Kaikki AC-xx jotka liittyvät tool-rekisteröintiin  
**Arvioitu kesto:** 3h  
**Prioriteetti:** MVP (viimeisenä)

**Kuvaus:**
Kokoa kaikki toolit yhteen, rekisteröi MCP-serveriin, ja varmista että kokonaisuus toimii Claude Desktopin kanssa.

**Test Scenarios:**

| TS-ID | Type | Scenario |
|-------|------|----------|
| TS-14.1 | HP | Server käynnistyy ilman virheitä |
| TS-14.2 | HP | Kaikki 5 toolia näkyvät Claudelle |
| TS-14.3 | HP | Tool-kutsu ohjautuu oikealle handlerille |
| TS-14.4 | HP | Virhe toolissa palautetaan MCP-virheenä |
| TS-14.5 | ER | Puuttuva konfiguraatio estää käynnistyksen |

**Koodirunko:**

```javascript
// index.js
const { Server } = require('@modelcontextprotocol/sdk/server/index.js');
const { StdioServerTransport } = require('@modelcontextprotocol/sdk/server/stdio.js');

const getLedger = require('./src/tools/getLedger');
const getAccounts = require('./src/tools/getAccounts');
const getBalances = require('./src/tools/getBalances');
const getDimensions = require('./src/tools/getDimensions');
const getVoucher = require('./src/tools/getVoucher');

const tools = [getLedger, getAccounts, getBalances, getDimensions, getVoucher];

async function main() {
  const server = new Server({
    name: 'netvisor-mcp',
    version: '1.0.0'
  }, {
    capabilities: {
      tools: {}
    }
  });

  // Rekisteröi toolit
  server.setRequestHandler('tools/list', async () => ({
    tools: tools.map(t => t.definition)
  }));

  server.setRequestHandler('tools/call', async (request) => {
    const { name, arguments: args } = request.params;
    const tool = tools.find(t => t.definition.name === name);
    
    if (!tool) {
      throw new Error(`Unknown tool: ${name}`);
    }
    
    try {
      const result = await tool.handler(args);
      return { content: [{ type: 'text', text: JSON.stringify(result, null, 2) }] };
    } catch (error) {
      return { 
        content: [{ type: 'text', text: JSON.stringify(error.toJSON?.() || { error: error.message }) }],
        isError: true 
      };
    }
  });

  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main().catch(console.error);
```

---

## 5. Konfiguraatio

### 5.1 Ympäristömuuttujat

```bash
# .env.example
NETVISOR_CUSTOMER_ID=your_customer_id
NETVISOR_CUSTOMER_KEY=your_customer_key
NETVISOR_PARTNER_ID=your_partner_id
NETVISOR_PARTNER_KEY=your_partner_key
NETVISOR_TIMEOUT=120000
```

### 5.2 Claude Desktop -konfiguraatio

```json
// claude_desktop_config.json
{
  "mcpServers": {
    "netvisor": {
      "command": "node",
      "args": ["/path/to/netvisor-mcp/index.js"],
      "env": {
        "NETVISOR_CUSTOMER_ID": "xxx",
        "NETVISOR_CUSTOMER_KEY": "xxx",
        "NETVISOR_PARTNER_ID": "xxx",
        "NETVISOR_PARTNER_KEY": "xxx"
      }
    }
  }
}
```

---

## 6. Traceability Matrix (täydennetty)

| REQ | AC | Task | Test Scenario | Status |
|-----|-----|------|---------------|--------|
| REQ-01 | AC-01 | Task-09 | TS-09.2 | 🔲 |
| REQ-01 | AC-02 | Task-03, Task-09 | TS-03.1-TS-03.4, TS-09.3, TS-09.4 | 🔲 |
| REQ-01 | AC-03 | Task-06, Task-09 | TS-06.1-TS-06.8, TS-09.1 | 🔲 |
| REQ-01 | AC-04 | Task-03, Task-09 | TS-03.4, TS-09.4 | 🔲 |
| REQ-01 | AC-05 | Task-05, Task-09 | TS-05.4-TS-05.6, TS-09.5 | 🔲 |
| REQ-01 | AC-06 | Task-09 | TS-09.6 | 🔲 |
| REQ-02 | AC-07 | Task-10 | TS-10.1 | 🔲 |
| REQ-02 | AC-08 | Task-10 | TS-10.1, TS-10.4, TS-10.5 | 🔲 |
| REQ-02 | AC-09 | Task-10 | TS-10.2 | 🔲 |
| REQ-02 | AC-10 | Task-10 | TS-10.3 | 🔲 |
| REQ-02 | AC-11 | Task-10 | TS-10.4 | 🔲 |
| REQ-03 | AC-12 | Task-11 | TS-11.1 | 🔲 |
| REQ-03 | AC-13 | Task-03, Task-11 | TS-03.5, TS-03.6, TS-11.4 | 🔲 |
| REQ-03 | AC-14 | Task-11 | TS-11.2, TS-11.3 | 🔲 |
| REQ-03 | AC-15 | Task-07, Task-11 | TS-07.2, TS-07.3, TS-11.1 | 🔲 |
| REQ-03 | AC-16 | Task-07, Task-11 | TS-07.2, TS-11.5 | 🔲 |
| REQ-03 | AC-17 | Task-07 | TS-07.1 | 🔲 |
| REQ-04 | AC-18 | Task-12 | TS-12.1 | 🔲 |
| REQ-04 | AC-19 | Task-08, Task-12 | TS-08.1-TS-08.3, TS-12.1 | 🔲 |
| REQ-04 | AC-20 | Task-08 | TS-08.1-TS-08.3 | 🔲 |
| REQ-04 | AC-21 | Task-12 | TS-12.2, TS-12.3 | 🔲 |
| REQ-04 | AC-22 | Task-08, Task-12 | TS-08.4, TS-12.4 | 🔲 |
| REQ-05 | AC-23 | Task-01 | TS-01.1 | 🔲 |
| REQ-05 | AC-24 | Task-01 | TS-01.2-TS-01.5 | 🔲 |
| REQ-05 | AC-25 | Task-01, Task-05 | TS-01.7, TS-05.7 | 🔲 |
| REQ-05 | AC-26 | Task-09-13 | Kaikki tool-testit | 🔲 |
| REQ-05 | AC-27 | Task-01, Task-05 | TS-01.6, TS-05.3 | 🔲 |
| REQ-06 | AC-28 | Task-02, Task-05 | TS-02.1, TS-05.4 | 🔲 |
| REQ-06 | AC-29 | Task-02, Task-04 | TS-02.2, TS-04.2, TS-04.4 | 🔲 |
| REQ-06 | AC-30 | Task-04 | TS-04.3 | 🔲 |
| REQ-06 | AC-31 | Task-02 | TS-02.1 (ei retry) | 🔲 |
| REQ-06 | AC-32 | Task-02 | TS-02.3 | 🔲 |
| REQ-06 | AC-33 | Task-02, Task-05 | TS-02.5, TS-05.6 | 🔲 |
| REQ-07 | AC-34 | Task-03 | TS-03.1-TS-03.6 | 🔲 |
| REQ-07 | AC-35 | Task-02, Task-03 | TS-02.4, TS-03.2, TS-03.4, TS-03.6 | 🔲 |
| REQ-07 | AC-36 | Task-03 | TS-03.7 | 🔲 |
| REQ-07 | AC-37 | Task-03 | TS-03.8 | 🔲 |
| REQ-07 | AC-38 | Task-02, Task-03 | TS-02.5, TS-03.9 | 🔲 |
| REQ-08 | AC-39 | Task-13 | TS-13.1 | 🔲 |
| REQ-08 | AC-40 | Task-13 | TS-13.1 | 🔲 |
| REQ-08 | AC-41 | Task-13 | TS-13.2 | 🔲 |
| REQ-08 | AC-42 | Task-02, Task-13 | TS-02.6, TS-13.3 | 🔲 |

---

## 7. Definition of Done

### Task-taso
- [ ] Kaikki test scenariot katettu testeillä
- [ ] Testit läpäisevät (jest)
- [ ] Koodi noudattaa CommonJS-syntaksia
- [ ] JSDoc-kommentit julkisissa funktioissa

### Feature-taso (MVP)
- [ ] Kaikki 14 taskia valmis
- [ ] Integraatiotestit sandbox-ympäristössä läpäisevät
- [ ] Claude Desktop tunnistaa kaikki 5 toolia
- [ ] Dokumentaatio päivitetty (README.md)

---

## 8. Toteutusjärjestys

```
┌─────────────────────────────────────────────────────────────────┐
│  VAIHE 1: Infrastruktuuri (Task-01 – Task-05)                  │
│  ─────────────────────────────────────────────────────────────  │
│  Task-01: Konfiguraatio          ← Ensimmäisenä                │
│  Task-02: Error-luokat                                          │
│  Task-03: Zod-skeemat                                           │
│  Task-04: Retry-logiikka                                        │
│  Task-05: Client wrapper                                        │
│                                                                 │
│  VAIHE 2: Transformerit (Task-06 – Task-08)                    │
│  ─────────────────────────────────────────────────────────────  │
│  Task-06: Pääkirja-transformer                                  │
│  Task-07: Saldo-transformer       ← KRIITTINEN (netvisorKey)   │
│  Task-08: Dimensio-transformer                                  │
│                                                                 │
│  VAIHE 3: MCP Tools (Task-09 – Task-14)                        │
│  ─────────────────────────────────────────────────────────────  │
│  Task-10: getAccounts             ← Ensin (mapping tarvitaan)  │
│  Task-09: getLedger               ← Tarvitsee accountMap       │
│  Task-11: getBalances             ← Tarvitsee accountMap       │
│  Task-12: getDimensions                                         │
│  Task-13: getVoucher                                            │
│  Task-14: MCP Server              ← Viimeisenä                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Testausstrategia

### 9.1 Yksikkötestit (jest)

```javascript
// tests/unit/config.test.js
describe('loadConfig', () => {
  it('TS-01.1: returns config when all env vars set', () => {
    // Arrange
    process.env.NETVISOR_CUSTOMER_ID = 'test';
    // ...
    
    // Act
    const config = loadConfig();
    
    // Assert
    expect(config.customerId).toBe('test');
  });
  
  it('TS-01.2: throws ConfigurationError when CUSTOMER_ID missing', () => {
    // Arrange
    delete process.env.NETVISOR_CUSTOMER_ID;
    
    // Act & Assert
    expect(() => loadConfig()).toThrow(ConfigurationError);
  });
});
```

### 9.2 Integraatiotestit (sandbox)

```javascript
// tests/integration/tools.test.js
describe('netvisor_get_ledger', () => {
  it('fetches real data from sandbox', async () => {
    const result = await getLedger.handler({
      organizationId: 'SANDBOX_Y_TUNNUS',
      startDate: '2024-01-01',
      endDate: '2024-01-31'
    });
    
    expect(result.success).toBe(true);
    expect(result.data.rows).toBeInstanceOf(Array);
  });
});
```

### 9.3 Mock fixtures

```json
// tests/fixtures/ledger-response.json
[
  {
    "status": "valid",
    "netvisorKey": 12345,
    "voucherDate": "2024-01-15",
    "voucherNumber": 1,
    "voucherDescription": "Myyntilasku 001",
    "voucherClass": "Myyntilasku",
    "voucherLine": [
      {
        "netvisorKey": 67890,
        "lineSum": 1000.00,
        "accountNumber": 3000,
        "vatPercent": 25.5,
        "vatCode": "KOMY"
      },
      {
        "netvisorKey": 67891,
        "lineSum": -1000.00,
        "accountNumber": 1700,
        "vatPercent": 0,
        "vatCode": "NONE"
      }
    ]
  }
]
```

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.0 | 2025-12-18 | Ensimmäinen versio, 14 taskia, 60+ test scenariota |

---

## Liittyvät dokumentit

| Dokumentti | Yhteys |
|------------|--------|
| SPEC_Netvisor_MCP.md | Toiminnalliset vaatimukset |
| netvisor_peer_review.md | Alkuperäisten speksien arviointi |
| @rantalainen/netvisor-api-client | GitHub: rantalainen/netvisor-api-client |
| @modelcontextprotocol/sdk | MCP SDK dokumentaatio |

---

*Dokumentin laatija: Claude (ohjelmistoarkkitehti)*  
*Valmis Claude Code -toteutukseen*
