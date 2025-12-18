# KEHITYSLOKI - Netvisor MCP Server

> **Versio:** 1.0  
> **Päivitetty:** 2025-12-18  
> **Projekti:** Netvisor MCP Server (Numbers Tilintarkastaja-Controller)

<!-- MUOKKAUSOHJE: Tee LISÄYKSIÄ ylös, älä kirjoita uudestaan. -->

---

## Projektin vaihe

```
[██░░░░░░░░] 20% - SPEC & TECH_SPEC valmis, CODE-vaihe alkamassa
```

**Nykyinen fokus:** Task-01 (Projektin alustus ja konfiguraatio)

---

## 🚀 Toteutusstrategia

> **Täysi dokumentaatio:** `docs/process/PROCESS_Implementation_Strategy.md`

### Toteutusjärjestys (TECH_SPEC v1.2)

```
┌─────────────────────────────────────────────────────────────────┐
│  VAIHE 1: Infrastruktuuri (Task-01 – Task-05b)                 │
│  ─────────────────────────────────────────────────────────────  │
│  Task-01: Konfiguraatio          ← SEURAAVA                    │
│  Task-02: Error-luokat                                          │
│  Task-03: Zod-skeemat                                           │
│  Task-04: Retry-logiikka                                        │
│  Task-05: Client wrapper                                        │
│  Task-05b: DataProvider                                         │
│                                                                 │
│  VAIHE 2: Transformerit (Task-06 – Task-08)                    │
│  ─────────────────────────────────────────────────────────────  │
│  Task-06: Pääkirja-transformer                                  │
│  Task-07: Saldo-transformer                                     │
│  Task-08: Dimensio-transformer                                  │
│                                                                 │
│  VAIHE 3: MCP Tools (Task-09 – Task-14)                        │
│  ─────────────────────────────────────────────────────────────  │
│  Task-10: getAccounts            ← Ensin (mapping tarvitaan)   │
│  Task-09: getLedger                                             │
│  Task-11: getBalances                                           │
│  Task-12: getDimensions                                         │
│  Task-13: getVoucher                                            │
│  Task-14: MCP Server             ← Viimeisenä                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Seuraava sessio

**Vaihe: 1 - Infrastruktuuri**

### Tehtävät:

| # | Tehtävä | Arvio | Status |
|:-:|---------|-------|:------:|
| 1 | Task-01: Projektin alustus (package.json, .env, config.js) | 2h | ▶ |
| 2 | Task-02: Custom Error -luokat | 1h | 🔲 |
| 3 | Task-03: Zod-validointiskeemat | 3h | 🔲 |

### Aloitusohje Claude Code:lle:

```
1. Lue TECH_SPEC_Netvisor_MCP.md (Task-01 osio)
2. Luo projektirakenne:
   - package.json
   - .env.example
   - src/config.js
   - tests/unit/config.test.js
3. TDD: Kirjoita testit ensin (TS-01.1 - TS-01.7)
4. Implementoi loadConfig()
```

---

## Odottavat tehtävät

### Phase 2 (SQLite-välimuisti)

- [ ] Task-15: SQLite schema
- [ ] Task-16: CachedDataProvider
- [ ] Task-17: Cache invalidation
- [ ] Task-18: netvisor_cache_status tool
- [ ] Task-19: netvisor_refresh_cache tool

---

## Sessiohistoria

<!-- UUSIN SESSIO AINA YLIMMÄKSI -->

### Session #1 (2025-12-18) - Projektin perustaminen

**Tavoite:** Perustaa netvisor-mcp repo ja dokumentaatio

**Saavutukset:**

- ✅ GitHub-repo luotu: `jussivares/netvisor-mcp`
- ✅ Prosessiohjeet kopioitu starter kitistä (9 kpl)
- ✅ Dokumenttitemplatet kopioitu (5 kpl)
- ✅ README.md luotu (asennus- ja käyttöohjeet)
- ✅ SYSTEM_PROMPT.md luotu (Claude-projektin ohjeet)
- ✅ INDEX.md luotu
- ✅ KEHITYSLOKI.md luotu

**Päätökset:**
- Käytetään ai-dev-starter-kit -pohjaa
- Node.js + CommonJS (MCP-yhteensopivuus)
- DataProvider-abstraktio Phase 2 -valmiutta varten

**Puuttuu vielä:**
- SPEC_Netvisor_MCP.md (käyttäjä tallentaa)
- TECH_SPEC_Netvisor_MCP.md (käyttäjä tallentaa)

---

## Task Status

| Task | Kuvaus | AC:t | Arvio | Status |
|------|--------|------|-------|--------|
| Task-01 | Konfiguraatio | AC-23–27 | 2h | 🔲 |
| Task-02 | Error-luokat | AC-28–33, 35, 38 | 1h | 🔲 |
| Task-03 | Zod-skeemat | AC-02, 13, 34–37 | 3h | 🔲 |
| Task-04 | Retry-logiikka | AC-29, 30 | 1.5h | 🔲 |
| Task-05 | Client wrapper | AC-05, 25, 27 | 2h | 🔲 |
| Task-05b | DataProvider | - | 2h | 🔲 |
| Task-06 | Ledger transformer | AC-03–06 | 2h | 🔲 |
| Task-07 | Balance transformer | AC-14–17 | 1.5h | 🔲 |
| Task-08 | Dimension transformer | AC-19–22 | 1.5h | 🔲 |
| Task-09 | getLedger tool | AC-01–06 | 2h | 🔲 |
| Task-10 | getAccounts tool | AC-07–11 | 1.5h | 🔲 |
| Task-11 | getBalances tool | AC-12–17 | 2h | 🔲 |
| Task-12 | getDimensions tool | AC-18–22 | 1.5h | 🔲 |
| Task-13 | getVoucher tool | AC-39–42 | 1.5h | 🔲 |
| Task-14 | MCP Server | - | 2h | 🔲 |

**Symbolit:** ✅ Valmis | 🔶 Työn alla | 🔲 Ei aloitettu | ▶ Seuraava

---

## Avoimet kysymykset

### Ratkaistu ✅

| Kysymys | Ratkaisu | Sessio |
|---------|----------|--------|
| Mikä stack? | Node.js + CommonJS | #1 |
| Miten cache? | DataProvider-abstraktio | #1 |

### Avoin 🔲

| Kysymys | Prioriteetti | Huom |
|---------|--------------|------|
| Sandbox-tunnukset testaukseen? | P1 | Tarvitaan integraatiotesteihin |

---

## Opitut asiat 🎓

| Sessio | Oppi | Toimenpide |
|--------|------|------------|
| #1 | ai-dev-starter-kit toimii hyvin uuden projektin pohjana | Dokumentoitu README:hen |

---

## Linkit

| Resurssi | URL |
|----------|-----|
| GitHub Repo | https://github.com/jussivares/netvisor-mcp |
| Netvisor API Client | https://github.com/rantalainen/netvisor-api-client |
| MCP Specification | https://modelcontextprotocol.io/ |

---

## Liittyvät dokumentit

| Dokumentti | Yhteys |
|------------|--------|
| INDEX.md | Dokumenttien navigointi |
| SPEC_Netvisor_MCP.md | Toiminnalliset vaatimukset |
| TECH_SPEC_Netvisor_MCP.md | Task decomposition |
| PROCESS_Implementation_Strategy.md | Toteutusstrategia |

---

*Dokumentti on osa Netvisor MCP Server -projektin dokumentaatiota.*
