# CODE Phase Process

> **Versio:** 1.7  
> **Päivitetty:** 2025-12-16  
> **Status:** Reviewed (Gemini)  
> **Lähteet:** Conventional Commits, Obra Superpowers, Uncle Bob TDD, PROCESS_Testing.md, PROCESS_SPEC_Writing.md

---

## Tarkoitus

Tämä dokumentti määrittelee CODE-vaiheen prosessin: miten TECH_SPEC muuttuu toimivaksi, testatuksi ja dokumentoiduksi koodiksi. Prosessi perustuu TDD-metodologiaan, atomic commits -käytäntöön ja systemaattiseen laadunvarmistukseen.

---

## Milloin käytetään

CODE-vaihe alkaa kun:
- TECH_SPEC on hyväksytty (vaihe 10 SPEC-prosessissa)
- Traceability Matrix on täydennetty Task → TS -tasolle
- Testiskenaariot (TS) on määritelty arrow-syntaksilla
- **Implementation Strategy tarkistettu** (→ `PROCESS_Implementation_Strategy.md`)

### ⚠️ Edellytykset: Tarkista ennen koodausta

```
┌─────────────────────────────────────────────────────────────────┐
│  CHECKLIST: Onko moduuli valmis CODE-vaiheeseen?               │
├─────────────────────────────────────────────────────────────────┤
│  [ ] TECH_SPEC on valmis ja hyväksytty                         │
│  [ ] Moduuli on oikeassa Hybridimallin vaiheessa               │
│      → Tarkista: docs/process/PROCESS_Implementation_Strategy.md│
│  [ ] Foundation-moduulit: Voidaan koodata heti                 │
│  [ ] Integration-moduulit: SPEC ensin, sitten TECH_SPEC + CODE │
│  [ ] Refinement tehty jos tarpeen (MemoryService)              │
└─────────────────────────────────────────────────────────────────┘
```

CODE-vaihe päättyy kun:
- Kaikki testit läpäisevät
- Definition of Done -kriteerit täyttyvät
- Traceability Matrix on täydennetty test_xxx() -tasolle
- Code review on hyväksytty

---

## TDD Workflow: RGRC-sykli

### Perusperiaate

**RGRC = Red-Green-Refactor-Commit**

```
┌─────────────────────────────────────────────────────────────┐
│  1. RED      → Kirjoita testi, katso sen EPÄONNISTUVAN     │
│  2. GREEN    → Kirjoita MINIMAALINEN koodi testin läpäisyyn │
│  3. REFACTOR → Siivoa koodi laadukkaaksi                    │
│  4. COMMIT   → Tallenna toimiva tila                        │
└─────────────────────────────────────────────────────────────┘
```

### Kriittiset säännöt

| Sääntö | Selitys | Rikkomisen seuraus |
|--------|---------|-------------------|
| **Testi ensin** | Älä kirjoita koodia ennen testiä | Poista koodi, aloita alusta |
| **Katso epäonnistuminen** | Testin PITÄÄ epäonnistua ensin | Testi ei testaa mitään |
| **Minimaalinen koodi** | Vain sen verran että testi läpäisee | Over-engineering |
| **Refactor vasta vihreällä** | Älä refaktoroi punaisen aikana | Sekava tila |
| **Commit toimivassa tilassa** | Testit läpäisevät AINA commitissa | Rikkinäinen historia |

### Task-granulariteetti

Obra Superpowers suosittelee **2-5 minuutin taskeja**. Tämä tarkoittaa:

- Yksi task = yksi pieni toiminnallisuus
- Yksi RGRC-sykli per task
- Helppo seurata ja debugata

**Esimerkki task-jaosta:**

```
TECH_SPEC Task: "Implement conversation storage"

Pilkotaan:
├── Task 1: Create Conversation model (2 min)
├── Task 2: Add save() method (3 min)
├── Task 3: Add load() method (3 min)
├── Task 4: Add list_all() method (2 min)
└── Task 5: Add delete() method (2 min)
```

### Uncle Bob's Three Laws

1. **Kirjoita production-koodia VAIN failaavan testin läpäisemiseksi**
2. **Kirjoita VAIN sen verran testiä että se failaa** (compilation failure = fail)
3. **Kirjoita VAIN sen verran production-koodia että yksi testi läpäisee**

---

## Commit-käytännöt

### Conventional Commits -formaatti

```
<type>[scope]: <description>

[optional body]

[optional footer]
```

### Pakolliset tyypit

| Tyyppi | Käyttö | SemVer |
|--------|--------|--------|
| `feat` | Uusi toiminnallisuus | MINOR |
| `fix` | Bugikorjaus | PATCH |
| `BREAKING CHANGE` | Rikkova muutos (footerissa tai `!`) | MAJOR |

### Suositellut lisätyypit

| Tyyppi | Käyttö |
|--------|--------|
| `test` | Testien lisäys/korjaus |
| `refactor` | Koodin uudelleenjärjestely (ei toiminnallinen muutos) |
| `docs` | Dokumentaatio |
| `style` | Formatointi (ei toiminnallinen muutos) |
| `chore` | Build, riippuvuudet, konfiguraatio |
| `perf` | Suorituskyvyn parantaminen |

### TDD-spesifiset commit-viestit

RGRC-syklin aikana käytä selkeitä viestejä:

```bash
# RED-vaihe (valinnainen, jos haluat tallentaa)
git commit -m "test(memory): add failing test for conversation save"

# GREEN+REFACTOR yhdessä (tyypillinen)
git commit -m "feat(memory): implement conversation save method"

# Pelkkä refaktorointi
git commit -m "refactor(memory): extract validation to helper"
```

### Atomic Commits

**Määritelmä:** Commit on atominen kun se:
1. Edustaa **yhtä loogista muutosta**
2. On mahdollisimman **pieni mutta täydellinen**
3. Jättää codebasen **toimivaan tilaan** (testit läpäisevät)
4. **Ei sekoita huolia** (esim. formatointi + logiikka erikseen)

**Hyödyt:**
- `git bisect` toimii (bug-etsintä)
- `git revert` yksinkertaista
- Code review helpompaa
- Selkeämpi historia

**Anti-pattern:**

```bash
# VÄÄRIN - liian monta asiaa
git commit -m "feat: add memory service, fix bug, update docs, refactor utils"

# OIKEIN - yksi asia kerrallaan
git commit -m "feat(memory): add MemoryService class"
git commit -m "fix(memory): handle empty conversation list"
git commit -m "docs(memory): add API documentation"
git commit -m "refactor(utils): extract date formatting"
```

---

## Code Review -prosessi

### Kaksitasoinen review

```
┌─────────────────────────────────────────────────────────────┐
│  TASO 1: Self-Review (Claude Code)                         │
│  ─────────────────────────────────                          │
│  Milloin: Jokaisen taskin jälkeen                           │
│  Fokus: Tekninen laatu, TDD-noudattaminen                   │
│  Tulos: Korjaukset ENNEN etenemistä                         │
├─────────────────────────────────────────────────────────────┤
│  TASO 2: Feature Review (Gemini)                           │
│  ─────────────────────────────────                          │
│  Milloin: Featuren valmistuttua                             │
│  Fokus: Arkkitehtuuri, kokonaisuus, edge caset              │
│  Tulos: Hyväksyntä tai korjauspyynnöt                       │
└─────────────────────────────────────────────────────────────┘
```

### Self-Review checklist (per task)

Claude Code tarkistaa jokaisen taskin jälkeen:

```markdown
## Self-Review Checklist

### TDD-noudattaminen
- [ ] Testi kirjoitettu ENNEN koodia
- [ ] Testi epäonnistui ensin (RED)
- [ ] Koodi minimaalinen testin läpäisyyn (GREEN)
- [ ] Refaktorointi tehty vihreällä (REFACTOR)

### Koodin laatu
- [ ] Type hints kaikissa funktioissa
- [ ] Docstring julkisissa metodeissa
- [ ] Ei hardkoodattuja arvoja
- [ ] Single Responsibility -periaate

### Testauksen laatu
- [ ] Testi testaa oikeaa asiaa
- [ ] Arrange-Act-Assert -rakenne
- [ ] Ei implementation details -testausta
- [ ] Edge caset huomioitu
```

### Severity-luokitus (Obra-malli)

Review-löydökset luokitellaan:

| Severity | Kuvaus | Toimenpide |
|----------|--------|------------|
| **CRITICAL** | Estää toiminnan, tietoturvaongelma | **Blokaa etenemisen** |
| **MAJOR** | Merkittävä ongelma, vaatii korjauksen | Korjaa ennen mergea |
| **MINOR** | Pieni parannus, ei estä | Korjaa kun ehdit |
| **INFO** | Huomio, ehdotus | Harkitse |

**Sääntö:** CRITICAL-löydökset estävät etenemisen seuraavaan taskiin.

### Feature Review (Gemini)

Kun feature on valmis, pyydä Gemini-review käyttäen tätä prompt templatea:

```markdown
Olen saanut featuren [NIMI] valmiiksi.

**TECH_SPEC:** [linkki tai tiedostonimi]
**Muutetut tiedostot:**
- src/[...].py
- tests/[...].py

**Testikattavuus:** [X]% (uusi koodi)

Tee Feature Review PROCESS_Code.md ohjeiden mukaisesti. 
Keskity erityisesti:
1. Arkkitehtuurin yhteensopivuus TECH_SPECin kanssa
2. API-rajapintojen selkeys
3. Virhetilanteiden käsittely
4. Testikattavuuden riittävyys
5. Dokumentaation ajantasaisuus

Käytä severity-luokitusta (CRITICAL/MAJOR/MINOR/INFO).
```

### Review-vastauksen rakenne

Gemini vastaa strukturoidusti:
```

---

## Definition of Done (DoD)

### Task-tason DoD

Jokainen task on "done" kun:

- [ ] Testi kirjoitettu ja läpäisee
- [ ] Koodi noudattaa type hints -käytäntöä
- [ ] Docstring julkisissa metodeissa (sis. REQ/AC/TS viittaukset)
- [ ] Self-review checklist läpäisty
- [ ] Commit tehty Conventional Commits -muodossa

### Feature-tason DoD

Feature on "done" kun:

- [ ] Kaikki taskit valmiita (task-DoD)
- [ ] Kaikki testit läpäisevät (`pytest` vihreä)
- [ ] Code coverage >80% (uudelle koodille)
- [ ] Static analysis läpäisee (`mypy`, `ruff`)
- [ ] Traceability Matrix päivitetty (batch)
- [ ] Gemini feature review hyväksytty
- [ ] API-dokumentaatio päivitetty
- [ ] KEHITYSLOKI päivitetty
- [ ] (Valinnainen) Coverage validation script ajettu (E4/E5, ks. KEHITYSLOKI.md)

### Moduuli-tason DoD

Moduuli on "done" kun:

- [ ] Kaikki featuret valmiita (feature-DoD)
- [ ] Integraatiotestit läpäisevät
- [ ] SPEC ja TECH_SPEC merkitty "Implemented"
- [ ] Traceability Matrix täydellinen (REQ → test_xxx)
- [ ] README/dokumentaatio ajan tasalla

---

## Traceability Matrix -päivitys

### Kaksitasoinen jäljitettävyys

| Taso | Mitä | Milloin |
|------|------|---------|
| **Task-taso** | Docstring sisältää REQ/AC/TS | Jokaisen testin kirjoituksen yhteydessä |
| **Feature-taso** | TECH_SPEC Matrix päivitetään | Batch-päivitys ennen Gemini-reviewtä |

**Miksi kaksi tasoa?**
- Task-taso (2-5 min) on liian nopea matriisin manuaaliseen päivitykseen
- Docstring on "code-level traceability" - elää koodin mukana
- Feature-tason batch-päivitys pitää TECH_SPECin ajan tasalla

### Task-taso: Docstring-linkitys

Jokainen testi dokumentoi jäljitettävyyden docstringissä:

```python
def test_conversation_save_creates_file():
    """
    Verify that saving a conversation creates a markdown file.
    
    REQ: REQ-001 (Conversation persistence)
    AC: AC-001 (Save creates file)
    TS: TS-001 (empty list → creates file with header)
    Type: Unit
    """
    # Arrange
    service = MemoryService()
    conversation = Conversation(messages=[])
    
    # Act
    result = service.save(conversation)
    
    # Assert
    assert result.success is True
    assert Path(result.filepath).exists()
```

### Feature-taso: Batch-päivitys TECH_SPECiin

Kun feature on valmis, päivitä TECH_SPEC.md:n Traceability Matrix:

```markdown
| REQ | AC | Task | TS | Testi |
|-----|-----|------|-----|-------|
| REQ-001 | AC-001 | Task-001 | TS-001 | ✅ test_conversation_save_creates_file |
| REQ-001 | AC-002 | Task-002 | TS-002 | ✅ test_conversation_load_returns_messages |
```

---

## Branch-strategia

### Solo/Small Team: Trunk-Based Development

Koska tämä on yhden henkilön projekti AI-avustajien kanssa, käytämme **trunk-based development** -mallia:

```
main (trunk)
  │
  ├── Commit: feat(memory): add MemoryService
  ├── Commit: test(memory): add save tests
  ├── Commit: feat(memory): implement save
  └── ... (suorat commitit mainiin)
```

### Milloin feature branch?

Feature branch vain kun:
1. Kokeellinen muutos joka voi epäonnistua
2. Iso refaktorointi joka kestää useita sessioita
3. Halutaan erillinen review ennen mergea

```bash
# Feature branch tarvittaessa
git checkout -b feature/experimental-vector-search
# ... työskentely ...
git checkout main
git merge feature/experimental-vector-search
git branch -d feature/experimental-vector-search
```

### Branch-päätöslogiikka (Obra-malli)

Kun feature/branch on valmis, valitse:

| Vaihtoehto | Milloin | Komento |
|------------|---------|---------|
| **Merge** | Valmis, testattu | `git merge && git branch -d` |
| **PR** | Halutaan dokumentoitu review | Luo PR GitHubissa |
| **Keep** | Jatketaan myöhemmin | Jätä branch |
| **Discard** | Ei toiminut, hylätään | `git branch -D` |

---

## Workflow-yhteenveto

### Päivittäinen kehitysrytmi

```
1. ALOITUS
   ├── git pull
   ├── Lue KEHITYSLOKI (missä jäätiin)
   ├── Lataa konteksti:
   │   ├── TECH_SPEC_XX.md (taskit, testiskenaariot)
   │   ├── ARCHITECTURE_OVERVIEW.md (rajapinnat)
   │   └── (Valinnainen) API_REFERENCE.md
   └── Valitse seuraava task TECH_SPECistä

2. TASK-SYKLI (toista per task)
   ├── RED: Kirjoita testi, aja, katso FAIL
   ├── GREEN: Kirjoita koodi, aja, katso PASS
   ├── REFACTOR: Siivoa, aja, varmista PASS
   ├── COMMIT: git commit -m "feat/test/fix..."
   └── SELF-REVIEW: Tarkista checklist

3. FEATURE VALMIS
   ├── Aja kaikki testit (pytest)
   ├── Aja static analysis (mypy, ruff)
   ├── Päivitä Traceability Matrix (batch)
   ├── Pyydä Gemini review (ks. prompt template)
   ├── Korjaa löydökset
   └── Päivitä KEHITYSLOKI

4. LOPETUS
   ├── git push
   ├── Päivitä KEHITYSLOKI (seuraavat askeleet)
   └── Tee hand-off seuraavalle sessiolle
```

### Komentojen pikaohje

```bash
# Testien ajo
pytest                          # Kaikki testit
pytest tests/unit/              # Vain unit-testit
pytest -v --tb=short            # Verbose, lyhyt traceback
pytest --cov=src --cov-report=term-missing  # Coverage

# Static analysis
mypy src/                       # Type checking
ruff check src/                 # Linting
ruff format src/                # Formatting

# Git
git status                      # Tila
git add -p                      # Stage osittain (atomic commits)
git commit -m "type(scope): message"
git push
```

---

## Anti-patternit

### TDD Anti-patternit

| Anti-pattern | Ongelma | Ratkaisu |
|--------------|---------|----------|
| **Code First** | Koodi ennen testiä | Poista koodi, aloita RED |
| **Test After** | Testi koodin jälkeen | Sama - ei TDD |
| **Green Bar Fever** | Ohitetaan REFACTOR | Pakota refaktorointivaihe |
| **Mock Overuse** | Liikaa mockeja | Testaa oikeita integraatioita |
| **Testing Implementation** | Testaa sisäistä toteutusta | Testaa käyttäytymistä |

### Commit Anti-patternit

| Anti-pattern | Ongelma | Ratkaisu |
|--------------|---------|----------|
| **Mega Commit** | Liian monta muutosta | Pilko atomic commiteiksi |
| **Broken Commit** | Testit eivät läpäise | Commit vain vihreällä |
| **Vague Message** | "fix stuff", "updates" | Käytä Conventional Commits |
| **Mixed Concerns** | Formatointi + logiikka | Erillisiin committeihin |

---

## Käytännön työnkulku (Validated Session #19)

> **Huom:** Tämä osio perustuu ensimmäiseen validoituun koodaussessioon (Task-01, Task-02, Task-03).

### Orchestration-malli: claude.ai + Claude Code

```
┌─────────────────────────────────────────────────────────────────┐
│  ROOLI           │ TEHTÄVÄT                                    │
├──────────────────┼─────────────────────────────────────────────┤
│  claude.ai       │ Orchestration, briefingit, testien ajo,    │
│  (projektipäällikkö) │ code review, git push                   │
├──────────────────┼─────────────────────────────────────────────┤
│  Claude Code     │ TDD-koodaus, iterointi, self-review        │
│  (kehittäjä)     │                                             │
├──────────────────┼─────────────────────────────────────────────┤
│  Desktop Commander│ pytest-ajo, git status, file ops          │
│  (työkalut)      │                                             │
└─────────────────────────────────────────────────────────────────┘
```

### Orchestration-roolit päätöksenteossa

> **Oppi Session #24:stä:** CODE-vaiheessa CC:llä on usein parempi näkemys tekniseen järjestykseen koska se on juuri toteuttanut edellisen komponentin ja tietää tarkalleen mitä rajapintoja on käytettävissä.

| Päätöstyyppi | Ensisijainen | Miksi |
|--------------|--------------|-------|
| Tekninen järjestys | **CC** | Näkee toteutetut rajapinnat ja riippuvuudet |
| "Mitä koodataan seuraavaksi?" | **CC** | Konkreettinen dependency-tieto koodista |
| Strategiset riskit | **claude.ai** | Näkee kokonaiskuvan |
| Arkkitehtuuripäätökset | **claude.ai** | Laajempi konteksti |

**Käytännössä:**
- Kun CC ehdottaa eri järjestystä kuin claude.ai, kuuntele CC:n perusteluja
- CC:n "dependency-driven" näkemys voi olla tarkempi kuin "turvallinen" valinta
- Spekulatiivinen review ilman kontekstia voi olla turhaa overheadia

### Trunk-Based Development

**Päätös:** Työskentele suoraan `main`-branchissa.

| Tilanne | Työtapa | Perustelu |
|---------|---------|-----------|
| Pienet taskit (1-3h) | Suoraan main | Nopea, testit varmistavat |
| Riskialtis muutos | Feature branch | Voidaan peruuttaa |
| Kokeilu | Feature branch | Ei sotke mainia |

**Ei worktreeta** - aiheuttaa permission-ongelmia.

### Workflow: Task-toteutus

```bash
# 1. Valmistelu
git pull                              # Ajantasaisuus

# 2. Briefing Claude Codelle
# - Luo BRIEFING_*.md (konteksti, TS:t, koodipohja)
# - Anna Claude Codelle

# 3. Claude Code koodaa (TDD)
# - RED → GREEN → REFACTOR

# 4. Testien ajo (claude.ai Desktop Commanderilla)
python -m pytest tests/unit/ -v

# 5. Bugien korjaus (jos tarpeen)
# - edit_block pieniin korjauksiin
# - Tai pyydä Claude Codea korjaamaan

# 6. Commit + Push
git add [files]
git commit -m "feat(scope): message"
git push
```

### Briefing-template (tiivistelmä)

```markdown
# Claude Code Briefing: [Moduuli] Task-XX

## 1. Konteksti
- Projektin polku, aiemmat taskit

## 2. Test Scenariot (taulukko)
| TS-ID | Type | Scenario | REQ | AC |

## 3. Toteutettava koodi (TECH_SPEC:stä)

## 4. Onnistumiskriteerit
- [ ] Tiedostot luotu
- [ ] Testit läpäisevät
- [ ] Commit-viesti valmis

## ⚠️ Muistilista
- [ ] Kaikki importit mukana (myös testeissä!)
- [ ] Työskentele mainissa (ei worktreeta)
- [ ] `git pull` ennen aloitusta
- [ ] `pytest tests/` ennen committia
```

### Briefing ja TDD-yhteensopivuus

Briefing voi sisältää eri tasoja valmiutta:

| Briefing sisältää | TDD-yhteensopivuus | CC:n tehtävä |
|-------------------|-------------------|--------------|
| Test Scenario -kuvaukset (TS-XX.X) | ✅ Täysin OK | Kirjoita testikoodi, aja RED |
| Valmis testikoodi | ✅ OK | Aja testi ENSIN, varmista RED |
| Valmis testi + koodi | ❌ Ei TDD | Vältä tätä! |

**Sääntö:** Jos briefing sisältää valmiin testikoodin, CC:n ENSIMMÄINEN tehtävä on ajaa testi ja varmistaa että se EPÄONNISTUU (RED). Vasta sitten kirjoitetaan production-koodi.

```
Briefing contains test code?
  └─ YES → Run test first, verify RED, then write code
  └─ NO → Write test yourself (normal TDD)
```

---

## Obra Superpowers Skills (Claude Code)

Claude Code voi aktivoida nämä skillit automaattisesti kehityksen aikana:

| Tilanne | Skill | Aktivoituu kun |
|---------|-------|----------------|
| TDD-sykli | `test-driven-development` | Testien kirjoitus |
| Bugi/virhe | `systematic-debugging` | Virhe havaittu |
| Syvät stack-virheet | `root-cause-tracing` | Jäljitys tarpeen |
| Ennen committia | `verification-before-completion` | Quality gate |

### Quality Gate: Ennen JOKAISTA committia

CC:n tulee varmistaa:

- [ ] Testit läpäisevät (tuore ajo, ei välimuistista)
- [ ] Ei uusia varoituksia
- [ ] Muutokset vastaavat tarkoitusta
- [ ] Commit-viesti Conventional Commits -muodossa

**Katso:** CLAUDE.md Skills Policy (sallitut/kielletyt skillit)

---

## Liittyvät dokumentit

| Dokumentti | Yhteys |
|------------|--------|
| **PROCESS_Implementation_Strategy.md** | **Hybridimalli: Milloin moduuli on valmis CODE-vaiheeseen** |
| PROCESS_Testing.md | TDD-sykli, testipyramidi, docstring-rakenne |
| PROCESS_SPEC_Writing.md | Vaihe 11 (CODE), Traceability Matrix, **E4/E5 määrittelyt** |
| ARCHITECTURE_OVERVIEW.md | Moduulirakenne, rajapinnat |
| TECH_SPEC_*.md | Task-lista, testiskenaariot |
| KEHITYSLOKI.md | Päivittäinen seuranta, **E4/E5 kehityspolku** |

---

## Automaattiset validointiskriptit (kehityspolku)

Nämä skriptit toteutetaan CODE-vaiheessa kun perustoiminnallisuus on valmis.

| ID | Nimi | Tarkoitus | Sijainti |
|----|------|-----------|----------|
| **E4** | SPECTest Coverage Validation | Tarkistaa että kaikki AC:t on katettu testeillä | KEHITYSLOKI.md, PROCESS_SPEC_Writing.md |
| **E5** | AC→Task Mapping Validation | Varmistaa että jokainen AC on linkitetty taskiin | KEHITYSLOKI.md, PROCESS_SPEC_Writing.md |
| **E11** | Arrange-Act-Assert ohje | Claude Code -ohje testirakenteiden noudattamiseen | **PROCESS_Testing.md (4.3.1)** |

**Käyttö Feature-tason DoD:ssä:**
```bash
# Kun E4/E5 on toteutettu:
python scripts/validate_spec_coverage.py   # E4
python scripts/validate_ac_task_mapping.py  # E5
```

---

## Lähteet

1. **Conventional Commits** - conventionalcommits.org
2. **Obra Superpowers** - github.com/obra/superpowers (v3.5.1)
3. **Uncle Bob's TDD** - Three Laws of TDD, RGRC cycle
4. **Atomic Commits** - gitbybit.com, kennyballou.com
5. **Trunk-Based Development** - trunkbaseddevelopment.com

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.7 | 2025-12-16 | **Orchestration-roolit päätöksenteossa:** CC:n dependency-driven näkemys, Session #24 opit |
| 1.6 | 2025-12-16 | **Obra Skills Integration:** Skills-osio CC:lle, Briefing+TDD-yhteensopivuus, Quality Gate |
| 1.5 | 2025-12-15 | **Session #19 opit:** Orchestration-malli, Trunk-Based Development, Briefing-template |
| 1.4 | 2025-12-15 | E11-viittaus päivitetty: KEHITYSLOKI → PROCESS_Testing.md (4.3.1) |
| 1.3 | 2025-12-15 | Lisätty PROCESS_Implementation_Strategy viittaus, edellytykset-checklist |
| 1.2 | 2025-12-14 | E4/E5/E11 kehityspolku -osio, selkeät viittaukset sijainteihin |
| 1.1 | 2025-12-14 | Gemini review: kontekstin lataus, traceability 2-taso, review template, E4/E5 DoD |
| 1.0 | 2025-12-14 | Ensimmäinen versio: TDD, Commits, Review, DoD, Branch |

---

*Dokumentti luotu: 2025-12-14*

