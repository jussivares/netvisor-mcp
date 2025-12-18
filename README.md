# Netvisor MCP Server 🔌

> **MCP Server for Netvisor accounting integration - Claude Desktop compatible**

[![Node.js](https://img.shields.io/badge/Node.js-18+-green.svg)](https://nodejs.org/)
[![MCP](https://img.shields.io/badge/MCP-Compatible-blue.svg)](https://modelcontextprotocol.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 🎯 Mikä tämä on?

Netvisor MCP Server mahdollistaa **Claude Desktop** -sovelluksen suoran integraation **Netvisor**-taloushallintojärjestelmään. Tilintarkastaja voi pyytää Claudea hakemaan ja analysoimaan kirjanpitodataa luonnollisella kielellä.

```
┌─────────────────┐     MCP Protocol     ┌─────────────────┐     HTTPS     ┌─────────────┐
│  Claude Desktop │◄──────────────────►│  netvisor-mcp   │◄────────────►│  Netvisor   │
│  (Tilintark.)   │     (localhost)     │  (Node.js)      │              │  API        │
└─────────────────┘                     └─────────────────┘              └─────────────┘
```

---

## ✨ Ominaisuudet

### MVP Tools

| Tool | Kuvaus |
|------|--------|
| `netvisor_get_ledger` | Hae pääkirja (tositteet ja rivit) aikaväliltä |
| `netvisor_get_accounts` | Hae tilikartta ja oletustilit |
| `netvisor_get_balances` | Hae tilisaldot päivämäärille |
| `netvisor_get_dimensions` | Hae dimensiot (kustannuspaikat, projektit) |
| `netvisor_get_voucher` | Hae yksittäisen tositteen tiedot |

### Turvallisuus

- ✅ API-avaimet .env-tiedostossa (ei kovakoodattuna)
- ✅ Tunnukset eivät näy lokeissa tai virheviestissä
- ✅ Stateless - ei tallenna dataa pysyvästi
- ✅ Suomenkieliset virheviestit

---

## 🚀 Asennus

### 1. Kloonaa repo

```bash
git clone https://github.com/jussivares/netvisor-mcp.git
cd netvisor-mcp
```

### 2. Asenna riippuvuudet

```bash
npm install
```

### 3. Konfiguroi ympäristömuuttujat

```bash
cp .env.example .env
```

Muokkaa `.env`:

```bash
NETVISOR_CUSTOMER_ID=your_customer_id
NETVISOR_CUSTOMER_KEY=your_customer_key
NETVISOR_PARTNER_ID=your_partner_id
NETVISOR_PARTNER_KEY=your_partner_key
NETVISOR_TIMEOUT=120000
```

### 4. Konfiguroi Claude Desktop

Lisää `claude_desktop_config.json` -tiedostoon:

**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`  
**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "netvisor": {
      "command": "node",
      "args": ["C:\\path\\to\\netvisor-mcp\\index.js"],
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

### 5. Käynnistä Claude Desktop uudelleen

MCP Server käynnistyy automaattisesti kun Claude Desktop avataan.

---

## 💬 Käyttöesimerkkejä

### Pääkirjan haku

> "Hae Yritys Oy:n (1234567-8) pääkirja ajalta 1.1.2024 - 31.12.2024"

### Tilisaldot

> "Näytä tilien 3000-3999 saldot vuoden 2024 lopussa"

### Tilikartta

> "Listaa yrityksen tilikartta"

### Kustannuspaikat

> "Mitä kustannuspaikkoja yrityksellä on käytössä?"

---

## 🏗️ Projektirakenne

```
netvisor-mcp/
├── index.js                 # MCP Server entry point
├── package.json
├── .env.example
│
├── src/
│   ├── config.js            # Konfiguraation lataus
│   ├── client.js            # Netvisor API wrapper
│   ├── errors.js            # Custom error classes
│   │
│   ├── data/                # DataProvider-abstraktio
│   │   ├── DataProvider.js
│   │   ├── ApiDataProvider.js
│   │   └── index.js
│   │
│   ├── tools/               # MCP Tools
│   │   ├── getLedger.js
│   │   ├── getAccounts.js
│   │   ├── getBalances.js
│   │   ├── getDimensions.js
│   │   └── getVoucher.js
│   │
│   ├── schemas/             # Zod-validointi
│   │   ├── input.js
│   │   └── output.js
│   │
│   ├── transformers/        # Data normalisointi
│   │   ├── ledger.js
│   │   ├── accounts.js
│   │   ├── balances.js
│   │   └── dimensions.js
│   │
│   └── utils/
│       ├── retry.js
│       └── format.js
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
│
└── docs/
    ├── specs/
    │   └── SPEC_Netvisor_MCP.md
    ├── tech-specs/
    │   └── TECH_SPEC_Netvisor_MCP.md
    ├── process/
    └── templates/
```

---

## 🧪 Testaus

```bash
# Yksikkötestit
npm test

# Integraatiotestit (vaatii sandbox-tunnukset)
npm run test:integration

# Kattavuusraportti
npm run test:coverage
```

---

## 📖 Dokumentaatio

| Dokumentti | Kuvaus |
|------------|--------|
| [SPEC_Netvisor_MCP.md](docs/specs/SPEC_Netvisor_MCP.md) | Toiminnalliset vaatimukset |
| [TECH_SPEC_Netvisor_MCP.md](docs/tech-specs/TECH_SPEC_Netvisor_MCP.md) | Tekninen toteutussuunnitelma |
| [KEHITYSLOKI.md](KEHITYSLOKI.md) | Projektin edistyminen |

---

## 🗺️ Roadmap

### MVP (v1.0) 🟢

- [x] Projektin alustus
- [ ] Konfiguraation hallinta
- [ ] 5 MCP toolia
- [ ] Virheenkäsittely ja retry
- [ ] Zod-validointi

### Phase 2 🟡

- [ ] SQLite-välimuisti
- [ ] Tilikausien vertailu
- [ ] Cache status/refresh tools

### Phase 3 🔵

- [ ] Laskujen haku
- [ ] Audit trail
- [ ] GDPR-suojatut palkkatiedot

---

## 🤝 Kehitys

Tämä projekti käyttää **AI-avusteista kehitysmetodologiaa**:

- **TDD** (Test-Driven Development)
- **11-vaiheinen SPEC-prosessi**
- **Claude + Claude Code** -orkestrointi

Katso [docs/process/](docs/process/) prosessiohjeet.

---

## 📄 Lisenssi

MIT License

---

## 🙏 Kiitokset

- [Rantalainen/netvisor-api-client](https://github.com/rantalainen/netvisor-api-client) - Netvisor API client
- [Model Context Protocol](https://modelcontextprotocol.io/) - MCP specification
- [Anthropic](https://anthropic.com/) - Claude AI

---

*Projekti: Numbers Tilintarkastaja-Controller*
