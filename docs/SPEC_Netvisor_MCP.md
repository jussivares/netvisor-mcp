# SPEC_Netvisor_MCP

> **Versio:** 1.0  
> **Päivitetty:** 2025-12-18  
> **Status:** Draft  
> **Projekti:** Numbers Tilintarkastaja-Controller  
> **Perustuu:** Peer Review 2025-12-18, netvisor_integration_spec.md, netvisor_tech_spec.md

---

## 1. Yleiskatsaus

### 1.1 Tarkoitus

Tämä dokumentti määrittelee **Netvisor MCP Server** -komponentin, joka mahdollistaa Claude Desktop -sovelluksen suoran integraation Netvisor-taloushallintojärjestelmään. MCP (Model Context Protocol) -palvelin toimii lokaalisti tilintarkastajan koneella ja tarjoaa Claude-assistentille työkalut kirjanpitodatan hakemiseen ja analysointiin.

### 1.2 Scope

**Sisältyy MVP:hen:**
- Pääkirjan haku (accountingLedger)
- Tilikartan haku (accountList)
- Tilisaldojen haku (getAccountBalance)
- Dimensioiden/kustannuspaikkojen haku (dimensionList)
- Yksittäisen tositteen haku
- Virheenkäsittely ja rate limiting
- Datan validointi (Zod)

**Ei sisälly MVP:hen (jatkokehitys):**
- Audit trail -lokitus
- GDPR-suojattu palkkatietojen haku
- Laskujen haku (salesInvoiceList, purchaseInvoiceList)
- Datan kirjoitus Netvisoriin
- Usean yrityksen samanaikainen käsittely

### 1.3 Ei-tavoitteet

- Tämä komponentti EI korvaa Netvisorin käyttöliittymää
- Tämä komponentti EI tallenna dataa pysyvästi (stateless)
- Tämä komponentti EI tee automaattisia kirjauksia

### 1.4 Arkkitehtuurikonteksti

```
┌─────────────────┐     MCP Protocol     ┌─────────────────┐     HTTPS     ┌─────────────┐
│  Claude Desktop │◄──────────────────►│  netvisor-mcp   │◄────────────►│  Netvisor   │
│  (Tilintark.)   │     (localhost)     │  (Node.js)      │              │  API        │
└─────────────────┘                     └─────────────────┘              └─────────────┘
                                               │
                                               ▼
                                        ┌─────────────────┐
                                        │  .env           │
                                        │  - CUSTOMER_ID  │
                                        │  - CUSTOMER_KEY │
                                        │  - PARTNER_ID   │
                                        │  - PARTNER_KEY  │
                                        └─────────────────┘
```

---

## 2. Requirements & Acceptance Criteria

### REQ-01: Pääkirjan haku 🟢

MCP Server tarjoaa työkalun pääkirjan (accountingLedger) hakemiseen Netvisorista. Tilintarkastaja voi pyytää Claudea hakemaan tilikauden tositteet ja rivit analysoitavaksi.

**Acceptance Criteria:**

| AC-ID | Kriteeri | Tyyppi |
|-------|----------|--------|
| AC-01 | Tool `netvisor_get_ledger` on rekisteröity ja näkyy Claudelle | Functional |
| AC-02 | Parametrit `startDate`, `endDate`, `organizationId` validoidaan ennen API-kutsua | Functional |
| AC-03 | Vastaus sisältää tositteet ja rivit normalisoituna (debet/kredit eroteltuna) | Functional |
| AC-04 | Virheellinen päivämäärämuoto palauttaa selkeän virheviestin | Error |
| AC-05 | API-virhe (401, 429, 500) käsitellään ja palautetaan ymmärrettävä virheviesti | Error |
| AC-06 | Tyhjä vastaus (ei tositteita) palautetaan tyhjänä listana, ei virheenä | Edge Case |

**Prioriteetti:** 🟢 MVP  
**Riippuvuudet:** REQ-05 (Konfiguraatio), REQ-06 (Virheenkäsittely)

---

### REQ-02: Tilikartan haku 🟢

MCP Server tarjoaa työkalun tilikartan (accountList) hakemiseen. Tilikartta sisältää kaikki yrityksen kirjanpitotilit numeroineen ja nimineen, sekä oletustilit (myyntisaamiset, ostovelat jne.).

**Acceptance Criteria:**

| AC-ID | Kriteeri | Tyyppi |
|-------|----------|--------|
| AC-07 | Tool `netvisor_get_accounts` on rekisteröity ja näkyy Claudelle | Functional |
| AC-08 | Vastaus sisältää tilit: netvisorKey, number, name, accountType, isActive | Functional |
| AC-09 | Vastaus sisältää oletustilit (companyDefaultAccounts) | Functional |
| AC-10 | Palautetaan mapping `netvisorKey → accountNumber` muiden toolien käyttöön | Functional |
| AC-11 | Inaktiiviset tilit sisältyvät vastaukseen merkittynä (isActive: false) | Functional |

**Prioriteetti:** 🟢 MVP  
**Riippuvuudet:** REQ-05 (Konfiguraatio)

---

### REQ-03: Tilisaldojen haku 🟢

MCP Server tarjoaa työkalun tilisaldojen (getAccountBalance) hakemiseen. Saldot voidaan hakea yksittäiselle päivälle tai aikavälille eri tarkkuuksilla (päivä, viikko, kuukausi, vuosi).

**Acceptance Criteria:**

| AC-ID | Kriteeri | Tyyppi |
|-------|----------|--------|
| AC-12 | Tool `netvisor_get_balances` on rekisteröity ja näkyy Claudelle | Functional |
| AC-13 | Parametri `balanceDates` (pakollinen) validoidaan muodossa "YYYY-MM-DD,YYYY-MM-DD" | Functional |
| AC-14 | Parametri `intervalType` (0-4) määrittää tarkkuuden | Functional |
| AC-15 | Vastaus sisältää tilit netvisorKey:llä JA accountNumber:lla (mapping REQ-02:sta) | Functional |
| AC-16 | **KRIITTINEN:** netvisorKey EI sekoitu accountNumber:iin filtteröinnissä | Functional |
| AC-17 | v4.0.0+ vastausrakenne (AccountBalances wrapper) käsitellään oikein | Functional |

**Prioriteetti:** 🟢 MVP  
**Riippuvuudet:** REQ-02 (Tilikartta netvisorKey-mappingiin), REQ-05 (Konfiguraatio)

---

### REQ-04: Dimensioiden haku 🟢

MCP Server tarjoaa työkalun dimensioiden (dimensionList) hakemiseen. Dimensiot sisältävät kustannuspaikat, projektit ja muut seurantakohteet.

**Acceptance Criteria:**

| AC-ID | Kriteeri | Tyyppi |
|-------|----------|--------|
| AC-18 | Tool `netvisor_get_dimensions` on rekisteröity ja näkyy Claudelle | Functional |
| AC-19 | Vastaus sisältää kaikki dimensiotyypit: kustannuspaikat, projektit, muut | Functional |
| AC-20 | Dimensiot palautetaan strukturoituna: { costCenters: [], projects: [], custom: {} } | Functional |
| AC-21 | Parametri `showHidden` (oletus: false) mahdollistaa piilotettujen näyttämisen | Functional |
| AC-22 | Tyhjä dimensiolista palautetaan tyhjinä taulukoina, ei virheenä | Edge Case |

**Prioriteetti:** 🟢 MVP  
**Riippuvuudet:** REQ-05 (Konfiguraatio)

---

### REQ-05: Konfiguraation hallinta 🟢

MCP Server lukee Netvisor API -tunnukset ympäristömuuttujista (.env) turvallisesti. Tunnuksia ei koskaan kovakoodata tai lokiteta.

**Acceptance Criteria:**

| AC-ID | Kriteeri | Tyyppi |
|-------|----------|--------|
| AC-23 | Konfiguraatio luetaan ympäristömuuttujista: NETVISOR_CUSTOMER_ID, NETVISOR_CUSTOMER_KEY, NETVISOR_PARTNER_ID, NETVISOR_PARTNER_KEY | Functional |
| AC-24 | Puuttuva pakollinen ympäristömuuttuja estää käynnistyksen selkeällä virheviestillä | Error |
| AC-25 | Tunnukset EI näy lokeissa, virheviestissä tai Claude-vastauksissa | Security |
| AC-26 | organizationId (Y-tunnus) annetaan tool-kutsussa, ei konfiguraatiossa | Functional |
| AC-27 | Timeout-asetus konfiguroidaan (oletus: 120s) | Functional |

**Prioriteetti:** 🟢 MVP  
**Riippuvuudet:** -

---

### REQ-06: Virheenkäsittely ja Rate Limiting 🟢

MCP Server käsittelee Netvisor API:n virheet gracefully ja toteuttaa älykkään retry-logiikan rate limiting -tilanteissa.

**Acceptance Criteria:**

| AC-ID | Kriteeri | Tyyppi |
|-------|----------|--------|
| AC-28 | HTTP 401 (Unauthorized) palauttaa virheen "Tarkista Netvisor-tunnukset" | Error |
| AC-29 | HTTP 429 (Rate Limited) triggeröi exponential backoff: 60s → 120s → 300s | Error |
| AC-30 | Retry-yrityksiä tehdään maksimissaan 3 kpl | Error |
| AC-31 | HTTP 500 (Server Error) palautetaan käyttäjälle ilman retry:tä | Error |
| AC-32 | Timeout (>120s) käsitellään ja ehdotetaan pienempää aikaväliä | Error |
| AC-33 | Verkkovirhe (ECONNREFUSED, ETIMEDOUT) palautetaan selkeänä viestinä | Error |

**Prioriteetti:** 🟢 MVP  
**Riippuvuudet:** -

---

### REQ-07: Datan validointi 🟢

MCP Server validoi sekä sisääntulevan datan (tool-parametrit) että Netvisorista saadun datan (API-vastaukset) Zod-skeemoilla.

**Acceptance Criteria:**

| AC-ID | Kriteeri | Tyyppi |
|-------|----------|--------|
| AC-34 | Tool-parametrit validoidaan Zod-skeemalla ennen API-kutsua | Functional |
| AC-35 | Virheellinen parametri palauttaa kentän nimen ja odotetun muodon | Error |
| AC-36 | API-vastaus validoidaan Zod-skeemalla (defensive parsing) | Functional |
| AC-37 | Puuttuva kenttä API-vastauksessa käsitellään oletusarvolla tai selkeällä virheellä | Error |
| AC-38 | Odottamaton API-vastausrakenne (breaking change) logitetaan ja palautetaan virhe | Error |

**Prioriteetti:** 🟢 MVP  
**Riippuvuudet:** -

---

### REQ-08: Yksittäisen tositteen haku 🟢

MCP Server tarjoaa työkalun yksittäisen tositteen yksityiskohtien hakemiseen netvisorKey:n perusteella.

**Acceptance Criteria:**

| AC-ID | Kriteeri | Tyyppi |
|-------|----------|--------|
| AC-39 | Tool `netvisor_get_voucher` on rekisteröity ja näkyy Claudelle | Functional |
| AC-40 | Parametri `netvisorKey` (pakollinen) identifioi tositteen | Functional |
| AC-41 | Vastaus sisältää tositteen täydet tiedot: päivä, numero, selite, rivit, liitteet | Functional |
| AC-42 | Olematon netvisorKey palauttaa virheen "Tositetta ei löydy" | Error |

**Prioriteetti:** 🟢 MVP  
**Riippuvuudet:** REQ-05 (Konfiguraatio)

---

### REQ-09: Inkrementaalinen pääkirjan haku 🟡

Suurten yritysten pääkirja voi sisältää satoja tuhansia rivejä. MCP Server tukee pääkirjan hakemista pienemmissä erissä (pagination by date range).

**Acceptance Criteria:**

| AC-ID | Kriteeri | Tyyppi |
|-------|----------|--------|
| AC-43 | Tool `netvisor_get_ledger` tukee parametria `batchDays` (oletus: 90) | Functional |
| AC-44 | Suuri aikaväli pilkotaan automaattisesti pienempiin pyyntöihin | Functional |
| AC-45 | Jokaisen erän jälkeen palautetaan välitulos Claudelle | Functional |
| AC-46 | Käyttäjä voi keskeyttää haun erien välissä | Functional |

**Prioriteetti:** 🟡 Phase 2  
**Riippuvuudet:** REQ-01 (Pääkirjan haku)

---

### REQ-10: Audit Trail 🟡

MCP Server tallentaa lokitiedoston jokaisesta datahaku-operaatiosta tilintarkastuksen jäljitettävyyttä varten.

**Acceptance Criteria:**

| AC-ID | Kriteeri | Tyyppi |
|-------|----------|--------|
| AC-47 | Jokainen API-kutsu logitetaan: aikaleima, organizationId, endpoint, parametrit | Functional |
| AC-48 | Vastauksesta tallennetaan: rivimäärä, SHA-256 hash, kesto | Functional |
| AC-49 | Lokitiedosto on JSON-muodossa, helposti parsiittava | Functional |
| AC-50 | Lokit säilytetään erillään API-tunnuksista | Security |

**Prioriteetti:** 🟡 Phase 2  
**Riippuvuudet:** -

---

## 3. API-määrittely (MCP Tools)

### 3.1 Tool: netvisor_get_ledger

```typescript
{
  name: "netvisor_get_ledger",
  description: "Hae pääkirja (tositteet ja rivit) Netvisorista",
  inputSchema: {
    type: "object",
    properties: {
      organizationId: { type: "string", description: "Y-tunnus (1234567-8)" },
      startDate: { type: "string", description: "Alkupäivä (YYYY-MM-DD)" },
      endDate: { type: "string", description: "Loppupäivä (YYYY-MM-DD)" },
      accountNumberStart: { type: "number", description: "Tilinumero alku (valinnainen)" },
      accountNumberEnd: { type: "number", description: "Tilinumero loppu (valinnainen)" }
    },
    required: ["organizationId", "startDate", "endDate"]
  }
}
```

### 3.2 Tool: netvisor_get_accounts

```typescript
{
  name: "netvisor_get_accounts",
  description: "Hae tilikartta Netvisorista",
  inputSchema: {
    type: "object",
    properties: {
      organizationId: { type: "string", description: "Y-tunnus (1234567-8)" }
    },
    required: ["organizationId"]
  }
}
```

### 3.3 Tool: netvisor_get_balances

```typescript
{
  name: "netvisor_get_balances",
  description: "Hae tilisaldot Netvisorista",
  inputSchema: {
    type: "object",
    properties: {
      organizationId: { type: "string", description: "Y-tunnus (1234567-8)" },
      balanceDates: { type: "string", description: "Päivämäärät (YYYY-MM-DD,YYYY-MM-DD)" },
      intervalType: { 
        type: "number", 
        enum: [0, 1, 2, 3, 4],
        description: "0=erilliset päivät, 1=päivä, 2=viikko, 3=kuukausi, 4=vuosi" 
      }
    },
    required: ["organizationId", "balanceDates"]
  }
}
```

### 3.4 Tool: netvisor_get_dimensions

```typescript
{
  name: "netvisor_get_dimensions",
  description: "Hae dimensiot (kustannuspaikat, projektit) Netvisorista",
  inputSchema: {
    type: "object",
    properties: {
      organizationId: { type: "string", description: "Y-tunnus (1234567-8)" },
      showHidden: { type: "boolean", description: "Näytä piilotetut (oletus: false)" }
    },
    required: ["organizationId"]
  }
}
```

### 3.5 Tool: netvisor_get_voucher

```typescript
{
  name: "netvisor_get_voucher",
  description: "Hae yksittäisen tositteen tiedot",
  inputSchema: {
    type: "object",
    properties: {
      organizationId: { type: "string", description: "Y-tunnus (1234567-8)" },
      netvisorKey: { type: "number", description: "Tositteen Netvisor-tunniste" }
    },
    required: ["organizationId", "netvisorKey"]
  }
}
```

---

## 4. Data Model

### 4.1 Primitiivi: LedgerRow

Järjestelmän perusyksikkö on **LedgerRow** - yksittäinen pääkirjarivi.

```typescript
interface LedgerRow {
  // Tosite-tiedot
  voucherNetvisorKey: number;    // Tositteen ID
  voucherDate: string;           // "2024-01-15"
  voucherNumber: number;         // Tositenumero
  voucherDescription: string;    // Tositteen selite
  voucherClass: string;          // "Myyntilasku", "Ostolasku", "Muistio"
  
  // Rivi-tiedot
  lineNetvisorKey: number;       // Rivin ID
  accountNetvisorKey: number;    // Tilin Netvisor-tunniste
  accountNumber: string;         // Tilinumero "3000"
  accountName: string;           // Tilin nimi
  debit: string;                 // Debet-summa stringinä (serialisoitavissa)
  credit: string;                // Kredit-summa stringinä
  description: string;           // Rivin selite
  
  // ALV
  vatPercent: number;
  vatCode: string;               // "KOMY", "EUMY", "NONE" jne.
  
  // Dimensiot
  dimensions: {
    costCenter?: string;
    project?: string;
    product?: string;
    customer?: string;
    custom?: Record<string, string>;
  };
}
```

### 4.2 Account (Tili)

```typescript
interface Account {
  netvisorKey: number;
  number: string;                // "3000"
  name: string;                  // "Myynti"
  accountType: 'account' | 'accountgroup';
  isActive: boolean;
  isCumulative: boolean;         // true = tasetili
  isNaturalNegative: boolean;    // true = luontaisesti negatiivinen
}
```

### 4.3 AccountBalance (Tilisaldo)

```typescript
interface AccountBalance {
  netvisorKey: number;
  accountNumber: string;         // Lisätty mappingilla
  accountName: string;           // Lisätty mappingilla
  balances: {
    date: string;                // "2024-12-31"
    debit: string;
    credit: string;
    balance: string;             // Nettosaldo
  }[];
}
```

### 4.4 Dimension (Dimensio)

```typescript
interface DimensionCategory {
  netvisorKey: number;
  name: string;                  // "Kustannuspaikka", "Projekti"
  isHidden: boolean;
  items: {
    netvisorKey: number;
    name: string;                // "Hallinto", "Myynti"
    isHidden: boolean;
    level: number;
    parentId?: number;
  }[];
}
```

---

## 5. Edge Cases & Error Handling

### 5.1 Tunnetut edge caset

| Tilanne | Käsittely |
|---------|-----------|
| Tyhjä tilikausi (ei tositteita) | Palauta tyhjä lista, ei virhettä |
| Erittäin suuri pääkirja (>100k riviä) | Ehdota inkrementaalista hakua |
| Tulevaisuuden päivämäärä | Salli, Netvisor palauttaa tyhjän |
| Väärä Y-tunnus | API palauttaa virheen, välitä käyttäjälle |
| Tili ilman saldoja | Sisällytä 0-saldoisena |
| Dimensio ilman arvoja | Palauta tyhjä items-lista |
| Tuntematon VAT-koodi | Säilytä alkuperäinen, älä kaada |

### 5.2 Virheviestien muotoilu

Kaikki virheviestit palautetaan suomeksi, selkokielisesti:

```javascript
const ERROR_MESSAGES = {
  AUTH_FAILED: "Netvisor-tunnistautuminen epäonnistui. Tarkista API-avaimet.",
  RATE_LIMITED: "Netvisor rajoittaa pyyntöjä. Odotetaan {seconds} sekuntia...",
  TIMEOUT: "Pyyntö aikakatkaistiin. Kokeile pienempää aikaväliä.",
  NOT_FOUND: "Tositetta {key} ei löytynyt.",
  INVALID_DATE: "Virheellinen päivämäärä '{value}'. Käytä muotoa YYYY-MM-DD.",
  INVALID_ORG_ID: "Virheellinen Y-tunnus '{value}'. Käytä muotoa 1234567-8.",
  NETWORK_ERROR: "Verkkovirhe: Netvisoriin ei saada yhteyttä.",
  UNEXPECTED: "Odottamaton virhe: {message}"
};
```

---

## 6. Turvallisuus

### 6.1 API-avainten suojaus

| Uhka | Suojaus |
|------|---------|
| Avaimet kovakoodattuna | ❌ Kielletty, luetaan .env:stä |
| Avaimet lokeissa | ❌ Ei logiteta koskaan |
| Avaimet virheviestissä | ❌ Maskataan aina |
| Avaimet Claude-vastauksessa | ❌ Ei välitetä Claudelle |

### 6.2 Datan suojaus

| Uhka | Suojaus |
|------|---------|
| Väärän yrityksen data | organizationId pakollinen joka kutsussa |
| Datan persistointi | MCP Server on stateless, ei tallenna |
| Man-in-the-middle | HTTPS Netvisor-API:iin |

---

## 7. Suorituskyky

### 7.1 SLA-tavoitteet

| Metriikka | Tavoite |
|-----------|---------|
| Tilikartan haku | < 2s |
| Kuukauden pääkirja (keskikokoinen yritys) | < 10s |
| Vuoden pääkirja (keskikokoinen yritys) | < 60s |
| Tilisaldot (vuositaso) | < 5s |

### 7.2 Bottleneckit

- **Netvisor API latenssi:** ~200-500ms per kutsu
- **Suuret vastaukset:** Pääkirja voi olla megatavuja
- **Rate limiting:** Netvisor rajoittaa pyyntömäärää

---

## 8. Traceability Matrix (pohja)

*Täydennetään TECH_SPEC- ja CODE-vaiheissa.*

| REQ | AC | Task | Test Scenario | Test | Status |
|-----|-----|------|---------------|------|--------|
| REQ-01 | AC-01 | - | - | - | 🔲 |
| REQ-01 | AC-02 | - | - | - | 🔲 |
| REQ-01 | AC-03 | - | - | - | 🔲 |
| REQ-01 | AC-04 | - | - | - | 🔲 |
| REQ-01 | AC-05 | - | - | - | 🔲 |
| REQ-01 | AC-06 | - | - | - | 🔲 |
| REQ-02 | AC-07 | - | - | - | 🔲 |
| REQ-02 | AC-08 | - | - | - | 🔲 |
| REQ-02 | AC-09 | - | - | - | 🔲 |
| REQ-02 | AC-10 | - | - | - | 🔲 |
| REQ-02 | AC-11 | - | - | - | 🔲 |
| REQ-03 | AC-12 | - | - | - | 🔲 |
| REQ-03 | AC-13 | - | - | - | 🔲 |
| REQ-03 | AC-14 | - | - | - | 🔲 |
| REQ-03 | AC-15 | - | - | - | 🔲 |
| REQ-03 | AC-16 | - | - | - | 🔲 |
| REQ-03 | AC-17 | - | - | - | 🔲 |
| REQ-04 | AC-18 | - | - | - | 🔲 |
| REQ-04 | AC-19 | - | - | - | 🔲 |
| REQ-04 | AC-20 | - | - | - | 🔲 |
| REQ-04 | AC-21 | - | - | - | 🔲 |
| REQ-04 | AC-22 | - | - | - | 🔲 |
| REQ-05 | AC-23 | - | - | - | 🔲 |
| REQ-05 | AC-24 | - | - | - | 🔲 |
| REQ-05 | AC-25 | - | - | - | 🔲 |
| REQ-05 | AC-26 | - | - | - | 🔲 |
| REQ-05 | AC-27 | - | - | - | 🔲 |
| REQ-06 | AC-28 | - | - | - | 🔲 |
| REQ-06 | AC-29 | - | - | - | 🔲 |
| REQ-06 | AC-30 | - | - | - | 🔲 |
| REQ-06 | AC-31 | - | - | - | 🔲 |
| REQ-06 | AC-32 | - | - | - | 🔲 |
| REQ-06 | AC-33 | - | - | - | 🔲 |
| REQ-07 | AC-34 | - | - | - | 🔲 |
| REQ-07 | AC-35 | - | - | - | 🔲 |
| REQ-07 | AC-36 | - | - | - | 🔲 |
| REQ-07 | AC-37 | - | - | - | 🔲 |
| REQ-07 | AC-38 | - | - | - | 🔲 |
| REQ-08 | AC-39 | - | - | - | 🔲 |
| REQ-08 | AC-40 | - | - | - | 🔲 |
| REQ-08 | AC-41 | - | - | - | 🔲 |
| REQ-08 | AC-42 | - | - | - | 🔲 |

---

## 9. Vaiheistus yhteenveto

| Prioriteetti | Vaatimukset | Perustelu |
|--------------|-------------|-----------|
| 🟢 MVP | REQ-01 – REQ-08 | Ydinominaisuudet tilintarkastukseen |
| 🟡 Phase 2 | REQ-09, REQ-10 | Skaalautuvuus ja jäljitettävyys |
| 🔵 Phase 3 | Laskut, palkkatiedot, GDPR | Laajempi toiminnallisuus |

---

## 10. Tulevat ominaisuudet (Phase 2)

### 10.1 SQLite-välimuisti

Phase 2:ssa toteutetaan **paikallinen SQLite-välimuisti** tilintarkastusdatalle. Tämä mahdollistaa:

| Ominaisuus | Hyöty tilintarkastajalle |
|------------|--------------------------|
| **Datan pysyvyys** | Sama tilikausi haetaan kerran, käytetään monta kertaa |
| **Vertailu edelliseen vuoteen** | Molemmat vuodet valmiina paikallisesti |
| **Offline-työskentely** | Analyysi toimii ilman nettiyhteyttä |
| **Nopeus** | SQLite-haku ~1ms vs API-haku ~5-30s |
| **Session-riippumattomuus** | Data säilyy Claude Desktop -uudelleenkäynnistysten yli |

### 10.2 Arkkitehtuurivalmius MVP:ssä

MVP:ssä toteutetaan **DataProvider-abstraktiokerros** joka mahdollistaa SQLite-välimuistin lisäämisen ilman refaktorointia:

```
MVP:                              Phase 2:
┌─────────────┐                   ┌─────────────┐
│  MCP Tools  │                   │  MCP Tools  │
└──────┬──────┘                   └──────┬──────┘
       │                                 │
       ▼                                 ▼
┌─────────────┐                   ┌─────────────┐
│ DataProvider│ (interface)       │ DataProvider│ (interface)
└──────┬──────┘                   └──────┬──────┘
       │                                 │
       ▼                                 ▼
┌─────────────┐                   ┌─────────────┐
│   API       │ (suora kutsu)     │   Cached    │ (SQLite + API)
│  Provider   │                   │  Provider   │
└──────┬──────┘                   └──────┬──────┘
       │                          ┌──────┴──────┐
       ▼                          ▼             ▼
┌─────────────┐             ┌─────────┐   ┌─────────────┐
│  Netvisor   │             │ SQLite  │   │  Netvisor   │
│    API      │             │ (cache) │   │    API      │
└─────────────┘             └─────────┘   └─────────────┘
```

**Miksi tämä on tärkeää:**
- Ei koodimuutoksia MCP Tools -kerrokseen Phase 2:ssa
- Vain uusi `CachedDataProvider`-luokka lisätään
- Testattavuus: `MockDataProvider` yksikkötesteissä

### 10.3 Phase 2 -työkalut (alustava)

| Tool | Kuvaus |
|------|--------|
| `netvisor_cache_status` | Näytä välimuistin tila: mitä dataa on tallennettuna |
| `netvisor_refresh_cache` | Päivitä välimuisti Netvisorista |
| `netvisor_compare_years` | Vertaa kahden tilikauden dataa |

### 10.4 Phase 2 -vaatimukset (alustava)

*Nämä täsmentyvät MVP-kokemusten perusteella.*

| REQ-ID | Kuvaus | Prioriteetti |
|--------|--------|--------------|
| REQ-11 | SQLite-välimuisti pääkirjalle | 🟡 Phase 2 |
| REQ-12 | Välimuistin tilan tarkistus | 🟡 Phase 2 |
| REQ-13 | Välimuistin manuaalinen päivitys | 🟡 Phase 2 |
| REQ-14 | Tilikausien vertailu | 🟡 Phase 2 |
| REQ-15 | Datan vanheneminen (TTL) | 🟡 Phase 2 |

---

## 11. Avoimet kysymykset

| # | Kysymys | Vaihtoehdot | Suositus |
|---|---------|-------------|----------|
| 1 | Miten käsitellään eri tilikausien vertailu? | A: Erillinen tool, B: Parametri nykyisiin | B: Lisätään `previousYear: boolean` |
| 2 | Tallennetaanko tilikartta välimuistiin? | A: Ei, haetaan aina, B: Session-cache | A: Stateless MVP, cache Phase 2 |
| 3 | Miten käsitellään monitilikausiset yritykset? | A: Käyttäjä antaa päivät, B: Haetaan tilikaudet ensin | A: Yksinkertaisin MVP:hen |

---

## 12. Sanasto

| Termi | Selitys |
|-------|---------|
| **MCP** | Model Context Protocol - Anthropicin protokolla Claude-integraatioihin |
| **netvisorKey** | Netvisorin sisäinen ID (esim. 847562), EI tilinumero |
| **accountNumber** | Kirjanpitotilin numero (esim. "3000") |
| **Y-tunnus** | Suomalainen yritystunnus (esim. "1234567-8") |
| **Dimensio** | Seurantakohde: kustannuspaikka, projekti, tuote jne. |
| **Tosite** | Kirjanpidon tapahtuma (voucher) |
| **Pääkirja** | Kaikki tilikauden tositteet ja rivit (ledger) |

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.1 | 2025-12-18 | Lisätty osio 10: Tulevat ominaisuudet (Phase 2), DataProvider-abstraktio |
| 1.0 | 2025-12-18 | Ensimmäinen versio, perustuu Peer Review -löydöksiin |

---

## Liittyvät dokumentit

| Dokumentti | Yhteys |
|------------|--------|
| TECH_SPEC_Netvisor_MCP.md | Tekninen toteutussuunnitelma |
| TECH_SPEC_Netvisor_MCP_Addendum_v1_1.md | DataProvider-abstraktio (Phase 2 valmius) |
| netvisor_peer_review.md | Alkuperäisten speksien arviointi |
| netvisor_integration_spec.md | Lähtödokumentti (Tilintarkastaja-tiimi) |
| netvisor_tech_spec.md | Lähtödokumentti (Tilintarkastaja-tiimi) |

---

*Dokumentin laatija: Claude (ohjelmistoarkkitehti)*  
*Projekti: Numbers Tilintarkastaja-Controller*
