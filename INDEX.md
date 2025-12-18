# INDEX - Netvisor MCP Server

> **Versio:** 1.0  
> **Päivitetty:** 2025-12-18  
> **Projekti:** Netvisor MCP Server (Numbers Tilintarkastaja-Controller)

---

## Projektin status

```
[██░░░░░░░░] 20% - TECH_SPEC valmis, CODE-vaihe alkamassa
```

**Seuraava askel:** Task-01 (Projektin alustus ja konfiguraatio)

---

## ⚡ Quick Reference

### Päivittäinen käyttö

| Tilanne | Dokumentti | Sijainti |
|---------|------------|----------|
| **Session aloitus** | KEHITYSLOKI | Kontekstissa |
| **Mistä löytyy X?** | INDEX (tämä) | Kontekstissa |
| **Vaatimukset?** | SPEC_Netvisor_MCP | docs/specs/ |
| **Taskit ja koodit?** | TECH_SPEC_Netvisor_MCP | docs/tech-specs/ |

### Skillit

| Tilanne | Skill |
|---------|-------|
| SPEC/RESEARCH | `spec-writing` |
| Tallennus | `document-updates` |
| Testaus | `testing` |
| Arkkitehtuuri | `systems-architecture` |

---

## 📐 Arkkitehtuuri

```
┌─────────────────┐     MCP Protocol     ┌─────────────────┐     HTTPS     ┌─────────────┐
│  Claude Desktop │◄──────────────────►│  netvisor-mcp   │◄────────────►│  Netvisor   │
│                 │     (localhost)     │  (Node.js)      │              │  API        │
└─────────────────┘                     └─────────────────┘              └─────────────┘
```

### MVP Tools (5 kpl)

| Tool | REQ | Status |
|------|-----|--------|
| `netvisor_get_ledger` | REQ-01 | 🔲 |
| `netvisor_get_accounts` | REQ-02 | 🔲 |
| `netvisor_get_balances` | REQ-03 | 🔲 |
| `netvisor_get_dimensions` | REQ-04 | 🔲 |
| `netvisor_get_voucher` | REQ-08 | 🔲 |

---

## 📚 Dokumenttihierarkia

```
netvisor-mcp/
│
├── README.md                        ← Asennus ja käyttöohjeet
├── KEHITYSLOKI.md                   ← Edistymisen seuranta
├── INDEX.md                         ← Tämä dokumentti
│
├── docs/
│   ├── claude-project/
│   │   └── SYSTEM_PROMPT.md         ← Claude-projektin ohjeet
│   │
│   ├── specs/
│   │   └── SPEC_Netvisor_MCP.md     ← Toiminnalliset vaatimukset
│   │
│   ├── tech-specs/
│   │   └── TECH_SPEC_Netvisor_MCP.md ← Tekniset määrittelyt, taskit
│   │
│   ├── process/                     ← Prosessiohjeet (9 kpl)
│   │   ├── PROCESS_Code.md
│   │   ├── PROCESS_Database_Management.md
│   │   ├── PROCESS_Debugging.md
│   │   ├── PROCESS_Document_Updates.md
│   │   ├── PROCESS_Implementation_Strategy.md
│   │   ├── PROCESS_Market_Research.md
│   │   ├── PROCESS_Research_Methodology.md
│   │   ├── PROCESS_SPEC_Writing.md
│   │   └── PROCESS_Testing.md
│   │
│   ├── templates/                   ← Dokumenttitemplatet (5 kpl)
│   │   ├── BRIEFING_TEMPLATE.md
│   │   ├── RESEARCH_TEMPLATE.md
│   │   ├── SPEC_TEMPLATE.md
│   │   ├── TECH_RESEARCH_TEMPLATE.md
│   │   └── TECH_SPEC_TEMPLATE.md
│   │
│   └── briefings/                   ← Claude Code briefingit
│       └── (luodaan tarvittaessa)
│
├── src/                             ← Lähdekoodi (luodaan Task-01:ssä)
└── tests/                           ← Testit (luodaan Task-01:ssä)
```

---

## 🔧 Task Status (TECH_SPEC)

### Vaihe 1: Infrastruktuuri

| Task | Kuvaus | Arvio | Status |
|------|--------|-------|--------|
| Task-01 | Projektin alustus ja konfiguraatio | 2h | 🔲 |
| Task-02 | Custom Error -luokat | 1h | 🔲 |
| Task-03 | Zod-validointiskeemat | 3h | 🔲 |
| Task-04 | Retry-logiikka | 1.5h | 🔲 |
| Task-05 | NetvisorApiClient wrapper | 2h | 🔲 |
| Task-05b | DataProvider Interface | 2h | 🔲 |

### Vaihe 2: Transformerit

| Task | Kuvaus | Arvio | Status |
|------|--------|-------|--------|
| Task-06 | Pääkirja-transformer | 2h | 🔲 |
| Task-07 | Saldo-transformer | 1.5h | 🔲 |
| Task-08 | Dimensio-transformer | 1.5h | 🔲 |

### Vaihe 3: MCP Tools

| Task | Kuvaus | Arvio | Status |
|------|--------|-------|--------|
| Task-09 | netvisor_get_ledger | 2h | 🔲 |
| Task-10 | netvisor_get_accounts | 1.5h | 🔲 |
| Task-11 | netvisor_get_balances | 2h | 🔲 |
| Task-12 | netvisor_get_dimensions | 1.5h | 🔲 |
| Task-13 | netvisor_get_voucher | 1.5h | 🔲 |
| Task-14 | MCP Server Entry Point | 2h | 🔲 |

**Yhteensä MVP:** ~24h

---

## 🎯 Vaatimukset (SPEC)

| REQ | Kuvaus | Prioriteetti | Status |
|-----|--------|--------------|--------|
| REQ-01 | Pääkirjan haku | 🟢 MVP | 🔲 |
| REQ-02 | Tilikartan haku | 🟢 MVP | 🔲 |
| REQ-03 | Tilisaldojen haku | 🟢 MVP | 🔲 |
| REQ-04 | Dimensioiden haku | 🟢 MVP | 🔲 |
| REQ-05 | Konfiguraation hallinta | 🟢 MVP | 🔲 |
| REQ-06 | Virheenkäsittely | 🟢 MVP | 🔲 |
| REQ-07 | Datan validointi | 🟢 MVP | 🔲 |
| REQ-08 | Yksittäisen tositteen haku | 🟢 MVP | 🔲 |
| REQ-09 | Inkrementaalinen haku | 🟡 Phase 2 | 🔲 |
| REQ-10 | Audit Trail | 🟡 Phase 2 | 🔲 |

**Symbolit:** ✅ Valmis | 🔶 Työn alla | 🔲 Ei aloitettu

---

## ⚠️ Kriittiset huomiot

### netvisorKey vs accountNumber

```
⚠️ NÄMÄ EI SAA SEKOITTUA!

netvisorKey  = Netvisorin sisäinen ID (esim. 847562)
accountNumber = Kirjanpitotilin numero (esim. "3000")
```

### API-avainten suojaus

```
❌ EI kovakoodata
❌ EI lokiteta  
❌ EI virheviesteihin
✅ Luetaan .env:stä
```

---

## Liittyvät dokumentit

| Dokumentti | Yhteys |
|------------|--------|
| **KEHITYSLOKI.md** | Päivittäinen edistymisen seuranta |
| **SPEC_Netvisor_MCP.md** | Toiminnalliset vaatimukset ja AC:t |
| **TECH_SPEC_Netvisor_MCP.md** | Task decomposition ja koodirungot |
| **SYSTEM_PROMPT.md** | Claude-projektin ohjeet |

---

*Dokumentti on osa Netvisor MCP Server -projektin dokumentaatiota.*
