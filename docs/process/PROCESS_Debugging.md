# PROCESS_Debugging.md

> **Versio:** 1.3  
> **Päivitetty:** 2025-12-16  
> **Tiedosto:** docs/process/PROCESS_Debugging.md  
> **Projekti:** Claude API -suunnittelutyökalu  
> **Lähde:** Obra Superpowers (systematic-debugging, root-cause-tracing, defense-in-depth)

---

## 1. Overview

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Core Principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

```
SYSTEMAATTINEN (15-30 min)     vs.    ARVAAMINEN (2-3 tuntia)
────────────────────────────────────────────────────────────
✅ 95% first-time fix rate            ❌ 40% first-time fix rate
✅ Ei uusia bugeja                    ❌ Uusia bugeja yleisiä
✅ Ymmärrys kasvaa                    ❌ "Se vain toimii nyt"
```

**Violating the letter of this process is violating the spirit of debugging.**

### Claude Code Skills

This process is implemented in Obra Superpowers skills that CC can activate automatically:

| Skill | Purpose | Activation |
|-------|---------|------------|
| `systematic-debugging` | 4-phase debugging process | When error/bug encountered |
| `root-cause-tracing` | Backward tracing technique | When tracing needed |
| `defense-in-depth` | Multi-layer validation | When implementing fixes |

**See:** CLAUDE.md Skills Policy for allowed/forbidden skills.

---

## 2. Iron Law

```
┌─────────────────────────────────────────────────────────────┐
│       NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST       │
└─────────────────────────────────────────────────────────────┘
```

If you haven't completed Phase 1, you cannot propose fixes.

### 2.1 The 3+ Fix Rule

| Korjausyritykset | Toimenpide |
|------------------|------------|
| 1-2 | Jatka Phase 1:stä, analysoi uudella tiedolla |
| **3+** | **STOP. Kyseenalaista arkkitehtuuri.** |

#### Tärkeä tarkennus: "Sama bugi"

| Tilanne | Toimenpide |
|---------|------------|
| 3+ korjausta **SAMAAN BUGIIN** | ⚠️ STOP - Kyseenalaista arkkitehtuuri |
| 3+ korjausta **ERI bugeihin** | ✅ Normaali TDD - jatka |

**"Sama bugi" tarkoittaa:**
- Sama virheilmoitus toistuu
- Korjaus rikkoo jotain muuta
- Symptomi siirtyy mutta ei poistu

**Esimerkki:**
```
Bugi: "Token count returns negative"

Fix 1: Add abs() → Breaks when count is None
Fix 2: Add None check → Breaks streaming  
Fix 3: Restructure calculation → Still negative in edge case

→ STOP! Tämä on arkkitehtuuriongelma, ei bugi.
  Keskustele orchestratorin kanssa.
```

**Pattern indicating architectural problem:**
- Each fix reveals new shared state/coupling
- Fixes require "massive refactoring"
- Each fix creates new symptoms elsewhere

**Kun 3+ Fix Rule laukeaa:**
1. STOP - älä yritä korjausta #4
2. Keskustele käyttäjän kanssa
3. Kysy: Onko pattern fundamentaalisesti väärä?
4. Harkitse refaktorointia vs. lisäkorjauksia
5. Tämä ei ole enää "bugi" - se on väärä abstraktio tai suunnitteluvirhe

---

## 3. When to Use

### 3.1 Triggerit - Käytä AINA näissä tilanteissa

| Tilanne | Esimerkki |
|---------|-----------|
| Test failure | `pytest` palauttaa FAILED |
| Unexpected behavior | "Tämän pitäisi toimia mutta..." |
| Build failure | Kompilointivirheet, import errors |
| Performance problem | Hidas suoritus, muistivuoto |
| Integration issue | API palauttaa odottamattomia arvoja |
| Virhe syvällä stackissa | Stack trace osoittaa monen tason läpi |

### 3.2 Erityisen tärkeää käyttää kun

- **Kiire painaa** - Emergencyt houkuttelevat arvaamaan
- **"Nopea korjaus" näyttää ilmeiseltä** - Ilmeiset ratkaisut ovat harvoin oikeita
- **Olet jo yrittänyt useita korjauksia** - Merkki systemaattisuuden puutteesta
- **Et täysin ymmärrä ongelmaa** - Jos et voi selittää, et voi korjata

### 3.3 Älä ohita vaikka

| Tekosyy | Totuus |
|---------|--------|
| "Ongelma on yksinkertainen" | Yksinkaisillakin bugeilla on juurisyy |
| "Ei ole aikaa prosessille" | Systemaattinen on NOPEAMPI kuin arvaaminen |
| "Korjaan nopeasti, tutkin myöhemmin" | "Myöhemmin" ei tule koskaan |

---

## 4. The Four Phases

```
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│  PHASE 1         │    │  PHASE 2         │    │  PHASE 3         │    │  PHASE 4         │
│  ROOT CAUSE      │ →  │  PATTERN         │ →  │  HYPOTHESIS      │ →  │  IMPLEMENTATION  │
│  INVESTIGATION   │    │  ANALYSIS        │    │  TESTING         │    │                  │
└──────────────────┘    └──────────────────┘    └──────────────────┘    └──────────────────┘
     Ymmärrä               Vertaa                 Testaa                  Korjaa
```

**You MUST complete each phase before proceeding to the next.**

### 4.1 Phase 1: Root Cause Investigation

**BEFORE attempting ANY fix:**

#### Step 1: Read Error Messages Carefully

```python
# ❌ Älä ohita
Traceback (most recent call last):
  File "memory_service.py", line 42, in store
    self._repository.save(item)
  File "repository.py", line 18, in save
    cursor.execute(query, params)
sqlite3.IntegrityError: UNIQUE constraint failed: memories.id

# ✅ Lue huolellisesti
# - Virhetyyppi: IntegrityError (ei yleinen Error)
# - Sijainti: repository.py, rivi 18
# - Syy: UNIQUE constraint, memories.id
# - Päätelmä: ID ei ole uniikki - tutki ID-generointia
```

Älä skippaa stack tracea. Kirjaa rivinumerot, tiedostopolut, virhekoodit.

#### Step 2: Reproduce Consistently

- Voitko toistaa virheen luotettavasti?
- Mitkä ovat tarkat askeleet?
- Tapahtuuko joka kerta?
- **Jos ei toistettavissa → kerää lisää dataa, älä arvaa**

#### Step 3: Check Recent Changes

```bash
# Mitä muuttui?
git diff HEAD~5 --stat
git log --oneline -10

# Milloin alkoi?
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
```

#### Step 4: Gather Evidence (Multi-Component Systems)

**When system has multiple components:**

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Layer 1    │ →  │  Layer 2    │ →  │  Layer 3    │
│  API Call   │    │  Service    │    │  Database   │
└─────────────┘    └─────────────┘    └─────────────┘
     ↓                   ↓                  ↓
   Log IN/OUT          Log IN/OUT        Log IN/OUT
```

```python
# Lisää diagnostiikka JOKAISEEN rajapintaan
def api_endpoint(request):
    logger.debug(f"=== API Layer ===")
    logger.debug(f"IN: {request.json}")
    
    result = service.process(request.json)
    
    logger.debug(f"OUT: {result}")
    return result

def service_process(data):
    logger.debug(f"=== Service Layer ===")
    logger.debug(f"IN: {data}")
    
    result = repository.save(data)
    
    logger.debug(f"OUT: {result}")
    return result
```

**Run once → Analyze → Identify failing layer → Investigate that layer**

#### Step 5: Add Stack Traces When Needed

Jos et löydä syytä, lisää instrumentointia **ennen** kaatuvaa operaatiota:

```python
import traceback
import sys

def problematic_operation(directory: str):
    # Debug logging - käytä stderr testeissä!
    print(f"DEBUG problematic_operation:", file=sys.stderr)
    print(f"  directory: {directory}", file=sys.stderr)
    print(f"  cwd: {os.getcwd()}", file=sys.stderr)
    print(f"  stack:\n{''.join(traceback.format_stack())}", file=sys.stderr)
    
    # Actual operation
    do_something(directory)
```

**Tärkeää:** Älä käytä normaalia loggeria testeissä - se voi niellä viestit. Käytä `stderr`.

### 4.2 Phase 2: Pattern Analysis

#### Step 1: Find Working Examples

```python
# Etsi toimiva esimerkki SAMASTA koodikannasta
# ❌ Älä: Googlaa random esimerkkiä
# ✅ Tee: Etsi miten vastaava asia toimii jo projektissa

# Esim: Jos MemoryService.store() ei toimi,
# katso miten ConfigService.save() toimii
```

#### Step 2: Compare Against References

- Lue referenssitoteutus KOKONAAN
- Älä silmäile - lue jokainen rivi
- Ymmärrä pattern täysin ennen soveltamista

#### Step 3: Identify Differences

| Toimiva koodi | Rikkinäinen koodi | Ero |
|---------------|-------------------|-----|
| `await db.commit()` | `db.commit()` | async puuttuu |
| `with transaction:` | suora kutsu | context manager |
| `validate(input)` | ei validointia | input check |

Älä oleta "tuolla ei ole väliä". Listaa kaikki erot.

#### Step 4: Understand Dependencies

- Mitä muita komponentteja tarvitaan?
- Mitä asetuksia, konfiguraatiota, ympäristömuuttujia?
- Mitä oletuksia koodi tekee?

### 4.3 Phase 3: Hypothesis and Testing

#### Step 1: Form Single Hypothesis

```markdown
## Hypoteesi #1

**Väite:** Juurisyy on X koska Y.

**Evidenssi:**
- Virheviesti viittaa Z:aan
- git diff näyttää muutoksen kohdassa W

**Testi:** Jos muutan A → B, virheen pitäisi kadota.
```

#### Step 2: Test Minimally

- **YKSI** muutos kerrallaan
- Pienin mahdollinen muutos hypoteesin testaamiseksi
- Älä korjaa useita asioita samalla

#### Step 3: Verify Before Continuing

| Tulos | Toimenpide |
|-------|------------|
| Toimii ✅ | → Phase 4 |
| Ei toimi ❌ | → Kumoa hypoteesi, palaa Step 1. Älä jätä "testikorjausta" koodiin. |
| Osittain | → Analysoi mikä osa toimi |

#### Step 4: When You Don't Know

- Sano ääneen: "En ymmärrä X:ää"
- Älä teeskentele tietäväsi
- Pyydä apua tai tutki lisää

### 4.4 Phase 4: Implementation

#### Step 1: Create Failing Test FIRST

```python
def test_store_handles_duplicate_id(memory_service):
    """
    REQ: REQ-01 (Memory Storage)
    AC: AC-01 (Store returns unique ID)
    TS: TS-01.X (Bugfix - duplicate ID handling)
    Type: Regression / Edge Case
    """
    # Arrange
    item1 = MemoryItem(content="first")
    item2 = MemoryItem(content="second")
    
    # Act
    memory_service.store(item1)
    result = memory_service.store(item2)  # Tämä kaatui
    
    # Assert
    assert result.id != item1.id  # Uniikki ID
```

**Kriittinen:** Sinulla ON OLTAVA testi joka epäonnistuu ennen korjausta (RED).

#### Step 2: Implement Single Fix

- Korjaa ROOT CAUSE, ei oiretta
- **YKSI** muutos kerrallaan
- Ei "samalla kun olen täällä" -parannuksia
- Ei niputtaa refaktorointia mukaan

#### Step 3: Defense-in-Depth

Kun juurisyy on löytynyt ja korjattu, lisää validointia **jokaiseen kerrokseen** jonka läpi virhe kulki:

**Milloin käyttää:**

| Tilanne | 4 kerrosta? | Perustelu |
|---------|-------------|-----------|
| **Bugfix** | ✅ Kyllä | Bugi pääsi läpi → validointi puuttui |
| **Security-related** | ✅ Kyllä | Aina kaikki kerrokset |
| **Uusi feature (TDD)** | 🟡 Valinnainen | TDD kattaa perustapaukset |
| **Refaktorointi** | ❌ Ei | Testit jo olemassa |

**Periaate:** Defense-in-Depth on **reaktiivinen** työkalu bugeihin, ei **proaktiivinen** vaatimus kaikkeen koodiin.

**Kerrokset (kun käytetään):**

```python
# Layer 1: Project.create() validates directory
def create(name: str, directory: str) -> Project:
    if not directory:
        raise ValueError("Directory cannot be empty")
    ...

# Layer 2: WorkspaceManager validates not empty
def initialize(self, directory: str):
    if not os.path.isabs(directory):
        raise ValueError(f"Expected absolute path: {directory}")
    ...

# Layer 3: Environment guard
def git_init(directory: str):
    if os.environ.get("NODE_ENV") == "test":
        if not directory.startswith(tempfile.gettempdir()):
            raise RuntimeError("Refusing git init outside tmpdir in test")
    ...
```

Tämä varmistaa, että sama virhe ei pääse läpi tulevaisuudessa.

#### Step 4: Verify Fix

- [ ] Testi menee läpi? (GREEN)
- [ ] Muut testit edelleen vihreällä?
- [ ] Ongelma oikeasti ratkaistu?

#### Step 5: If Fix Doesn't Work

```
Korjausyritykset: 1  → Palaa Phase 1, analysoi uudella tiedolla
Korjausyritykset: 2  → Palaa Phase 1, analysoi uudella tiedolla
Korjausyritykset: 3+ → STOP! Kyseenalaista arkkitehtuuri (ks. 2.1)
```

---

## 5. Root Cause Tracing

### 5.1 Core Principle

```
Bug ilmenee syvällä          Älä korjaa täällä!
         ↓                          ↓
┌─────────────────┐          ┌─────────────────┐
│  test_xxx()     │          │                 │
│       ↓         │          │                 │
│  Service.call() │    ←──   │  Jäljitä        │
│       ↓         │          │  taaksepäin     │
│  Repository()   │          │                 │
│       ↓         │          │                 │
│  💥 ERROR 💥    │          │                 │
└─────────────────┘          └─────────────────┘
                                    ↓
                             Korjaa LÄHTEELLÄ
```

**NEVER fix just where the error appears. Trace back to find the original trigger.**

### 5.2 The Tracing Process

#### Example: Git Init Wrong Directory

```
1. OBSERVE SYMPTOM
   Error: git init failed in /Users/jesse/project/packages/core

2. FIND IMMEDIATE CAUSE
   await execFileAsync('git', ['init'], { cwd: projectDir });

3. ASK: WHAT CALLED THIS?
   WorktreeManager.createSessionWorktree(projectDir, sessionId)
     → called by Session.initializeWorkspace()
     → called by Session.create()
     → called by test at Project.create()

4. KEEP TRACING UP
   projectDir = '' (empty string!)
   Empty string as cwd resolves to process.cwd()
   That's the source code directory!

5. FIND ORIGINAL TRIGGER
   const context = setupCoreTest(); // Returns { tempDir: '' }
   Project.create('name', context.tempDir); // Accessed before beforeEach!

6. FIX AT SOURCE
   Made tempDir a getter that throws if accessed before beforeEach
```

### 5.3 Test Pollution Detection

Jos testit vuotavat tilaa toisilleen (esim. `.git` ilmestyy source-kansioon), käytä bisektio-tekniikkaa:

```bash
# Ajaa testit yksi kerrallaan, pysähtyy ensimmäiseen "saastuttajaan"
./find-polluter.sh '.git' 'src/**/*.test.ts'
```

Tämä auttaa löytämään mikä testi jättää jälkiä.

---

## 6. AI-Assisted Debugging (Claude Code)

### 6.1 Hallusinaatioriski

AI:na sinulla on riski **hallusinoida korjauksia** - ehdottaa koodimuutoksia jotka "voisivat toimia" ilman todellista ymmärrystä ongelmasta. Tämä on erityisen vaarallista debuggauksessa.

### 6.2 Säännöt Claude Code:lle

#### Sääntö 1: Älä ehdota koodia ennen diagnoosia

```
❌ VÄÄRIN:
"Kokeillaan muuttaa X, se saattaa auttaa."
"Lisätään try-catch tähän."

✅ OIKEIN:
"Analysoin virheen. Stack trace osoittaa riville 42 tiedostossa repository.py. 
Jäljitän taaksepäin: kuka kutsui tätä funktiota ja millä arvoilla?"
```

#### Sääntö 2: Pyydä lupaa instrumentointiin

Jos et näe virheen syytä, älä arvaa. Sen sijaan:
- Ehdota diagnostiikan lisäämistä
- Pyydä käyttäjältä lupa `edit_block`:lla lisätä loggausta
- Analysoi tulokset ennen korjausehdotusta

#### Sääntö 3: Tunnusta tietämättömyys

```
✅ "En ymmärrä miksi tämä virhe tapahtuu. Tarvitsen lisää tietoa:
   1. Voitko ajaa testin uudelleen verbose-modessa?
   2. Mikä on muuttujan X arvo tässä kohtaa?"
```

#### Sääntö 4: Seuraa 3+ Fix Rule:a

Jos olet ehdottanut 2 korjausta jotka eivät toimineet:
- STOP
- Ilmoita käyttäjälle: "Kaksi korjausyritystä epäonnistui. 3+ Fix Rule:n mukaan meidän pitäisi kyseenalaistaa lähestymistapa."
- Kysy: "Haluatko jatkaa vai pitäisikö miettiä arkkitehtuuria uudelleen?"

### 6.3 Human-in-the-Loop Signals

**Watch for these redirections from user:**

| User Signal | Meaning | Action |
|-------------|---------|--------|
| "Is that not happening?" | Oletit ilman verifiointia | Palaa Phase 1 |
| "Will it show us...?" | Olisit pitänyt lisätä diagnostiikkaa | Lisää loggausta |
| "Stop guessing" | Ehdotit korjausta ilman ymmärrystä | Palaa Phase 1 |
| "We're stuck?" (frustrated) | Lähestymistapa ei toimi | Kyseenalaista arkkitehtuuri |

**When you see these: STOP. Return to Phase 1.**

### 6.4 When to Escalate to Human

```
┌─────────────────────────────────────────────────────────────┐
│  ESCALATE TO HUMAN WHEN:                                    │
├─────────────────────────────────────────────────────────────┤
│  • 3+ Fix Rule triggered                                    │
│  • Root cause is in external dependency                     │
│  • Fix requires architectural decision                      │
│  • Uncertainty about business logic                         │
│  • Evidence is contradictory                                │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. Integration with TDD and Traceability

### 7.1 Milloin RED on "Debug"?

TDD:n RED-vaihe ja debugging ovat eri asioita:

**Normaali TDD:**
- Kirjoitat testin → Se epäonnistuu odotetusti (AssertionError)
- Tämä on **tarkoituksellista** - kirjoita koodi ja mene GREEN:iin

**Debugging-tilanne:**
1. Testi epäonnistuu **väärällä tavalla** (Crash/Exception, ei AssertionError)
2. Koodi on jo kirjoitettu, mutta testi ei mene läpi
3. Uusi koodi rikkoo vanhan testin (**Regressio**)
4. Virhe tulee yllättävästä paikasta (syvällä stackissa)

Näissä tilanteissa **älä kirjoita lisää koodia** - käynnistä debugging-prosessi.

### 7.2 Bugfix Traceability (PAKOLLINEN)

Bugin luonne määrittää käsittelytavan:

#### Case A: Regression (Vanha toiminnallisuus hajosi)

```bash
# Linkitä suoraan olemassa olevaan testiskenaarioon
git commit -m "fix(memory): fix regression in save logic (ref TS-01.2)"
```

- Testi on jo olemassa TECH_SPECissä
- Korjaa koodi, testi menee vihreälle
- Commit viittaa olemassa olevaan TS:ään

#### Case B: New Edge Case (Spesifikaatio oli puutteellinen)

```markdown
1. PÄIVITÄ TECH_SPEC ENSIN
   
   Lisää uusi testiskenaario:
   | TS-01.5 | Null bytes in content | Edge Case |

2. KIRJOITA FAILING TEST
   
   def test_store_handles_null_bytes(...):
       """TS: TS-01.5 (Null bytes in content)"""

3. KORJAA JA COMMIT
   
   git commit -m "fix(memory): handle null bytes (ref TS-01.5)"
```

**Miksi:** TECH_SPEC pysyy totuuden lähteenä. Jos bugikorjaus on vain Git-historiassa, se katoaa ajan myötä.

### 7.3 Commit Message Format

```
fix(<scope>): <description> (ref <TS-reference>)

# Esimerkkejä:
fix(memory): fix regression in save logic (ref TS-01.2)
fix(memory): handle null bytes (ref TS-01.5)
fix(context): correct token calculation (ref TS-03.1)
```

---

## 8. Post-mortem Documentation

### 8.1 When to Document

| Dokumentoi | Älä dokumentoi |
|------------|----------------|
| 3+ Fix Rule laukesi | Kirjoitusvirheet |
| Juurisyy oli yllättävä | Triviaalit logiikkavirheet |
| Opettavainen kokemus | RED-vaiheen normaalit fixaukset |
| Arkkitehtuuripäätös syntyi | Rutiinikorjaukset |

### 8.2 Where to Document

**KEHITYSLOKI.md - session merkintään:**

```markdown
### Sessio #XX (YYYY-MM-DD) - [Otsikko]

**Debugging Insight:** [3+ Fix Rule / Lessons Learned]
- Ongelma: [Kuvaus]
- Juurisyy: [Mikä oikeasti aiheutti]
- Ratkaisu: [Miten korjattiin]
- Oppi: [Mitä opimme tulevaisuutta varten]
```

### 8.3 Example

```markdown
**Debugging Insight:** 3+ Fix Rule - ID Generation

- Ongelma: Duplicate ID errors satunnaisesti testeissä
- Korjausyritykset:
  1. UUID format fix → ei auttanut
  2. Timestamp lisäys → ei auttanut  
  3. Random suffix → ei auttanut
- Juurisyy: Testit käyttivät samaa seed-arvoa, UUID mock oli deterministinen
- Ratkaisu: Refaktoroitiin ID-generaatio injektoitavaksi riippuvuudeksi
- Oppi: Testien deterministisyys voi aiheuttaa yllättäviä ongelmia
```

---

## 9. Red Flags - STOP and Follow Process

### 9.1 Warning Thoughts

If you catch yourself thinking:

| Thought | Reality |
|---------|---------|
| "Quick fix for now, investigate later" | "Later" never comes |
| "Just try changing X and see if it works" | Guessing, not debugging |
| "Add multiple changes, run tests" | Can't isolate what worked |
| "Skip the test, I'll manually verify" | Untested fixes don't stick |
| "It's probably X, let me fix that" | Seeing symptoms ≠ understanding |
| "Pattern says X but I'll adapt it differently" | Partial understanding = bugs |
| "One more fix attempt" (after 2+ failures) | 3+ = architectural problem |

**ALL of these mean: STOP. Return to Phase 1.**

### 9.2 Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple, don't need process" | Simple issues have root causes too |
| "Emergency, no time for process" | Systematic is FASTER than thrashing |
| "Just try this first, then investigate" | First fix sets the pattern |
| "I'll write test after confirming fix works" | Untested fixes don't stick |
| "Multiple fixes at once saves time" | Can't isolate what worked |
| "Reference too long, I'll adapt the pattern" | Partial understanding = bugs |

---

## 10. Quick Reference

### 10.1 Phase Summary

| Phase | Key Activities | Success Criteria |
|-------|----------------|------------------|
| **1. Root Cause** | Read errors, reproduce, check changes, gather evidence | Understand WHAT and WHY |
| **2. Pattern** | Find working examples, compare | Identify differences |
| **3. Hypothesis** | Form theory, test minimally | Confirmed or new hypothesis |
| **4. Implementation** | Create test, fix, defense-in-depth, verify | Bug resolved, tests pass |

### 10.2 Debugging Checklist

```
□ Phase 1: Root Cause Investigation
  □ Read error message completely
  □ Reproduce consistently
  □ Check recent changes (git diff)
  □ Add diagnostics if multi-component
  □ Add stack traces if needed

□ Phase 2: Pattern Analysis
  □ Find working example in codebase
  □ Compare working vs broken
  □ List all differences

□ Phase 3: Hypothesis Testing
  □ Form single, specific hypothesis
  □ Test with minimal change
  □ Verify result (don't leave test code)

□ Phase 4: Implementation
  □ Write failing test FIRST (RED)
  □ Implement single fix
  □ Add defense-in-depth validations
  □ Verify all tests pass (GREEN)
  □ Update TECH_SPEC if new edge case
  □ Commit with proper reference
```

### 10.3 Fix Counter

```
Fix attempt #1: ❌ → Return to Phase 1
Fix attempt #2: ❌ → Return to Phase 1  
Fix attempt #3: ❌ → STOP! Question architecture.
```

### 10.4 TDD vs Debugging Quick Check

```
Testi failaa...
  └─ Odotetusti (AssertionError, uusi testi)?
       └─ Kyllä → Normaali TDD, kirjoita koodi
       └─ Ei (Crash/Exception/Regressio) → DEBUGGING PROCESS
```

---

## 11. Related Documents

| Document | Relationship |
|----------|--------------|
| PROCESS_Code.md | TDD workflow, commit conventions |
| PROCESS_Testing.md | Test structure, fixtures, DoD |
| KEHITYSLOKI.md | Post-mortem documentation |
| TECH_SPEC_*.md | Traceability (TS references) |

---

## Change History

| Version | Date | Changes |
|---------|------|---------|
| 1.3 | 2025-12-16 | **Clarifications:** "Sama bugi" tarkennus 3+ Fix Rule:een, Defense-in-Depth "when to use" taulukko |
| 1.2 | 2025-12-16 | **Skills Integration:** CC skills -viittaus Overview-osioon, defense-in-depth lisätty lähteisiin |
| 1.1 | 2025-12-15 | Gemini-parannukset: "hallusinoida korjauksia" -varoitus, "Milloin RED on Debug?" -erottelu, Defense-in-Depth integroitu Phase 4:ään |
| 1.0 | 2025-12-14 | Initial version based on Obra Superpowers systematic-debugging and root-cause-tracing skills |

---

*This document is part of the Claude API Planning Tool project documentation.*
