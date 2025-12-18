# SPEC_XX: [Moduulin nimi]

> **Versio:** 1.0  
> **Päivitetty:** [PÄIVÄMÄÄRÄ]  
> **Status:** Draft / Review / Approved  
> **Perustuu:** RESEARCH_XX

<!-- 
KÄYTTÖOHJE:
1. Korvaa XX järjestysnumerolla (01, 02, 03...)
2. Korvaa [Moduulin nimi] moduulin oikealla nimellä
3. Täytä jokainen osio huolellisesti
4. Varmista että jokainen vaatimus on testattavissa
5. Poista tämä kommenttilohko kun olet valmis
-->

---

## 1. Yleiskatsaus

### 1.1 Tarkoitus

[Kuvaile moduulin päätehtävä 2-3 lauseella. Mitä ongelmaa se ratkaisee?]

### 1.2 Scope

**Sisältyy:**
- [Mitä moduuli tekee]
- [Mitä moduuli tekee]

**Ei sisälly:**
- [Mitä moduuli EI tee - rajaukset]
- [Mitä jätetään muille moduuleille]

### 1.3 Primitiivi

**Perusyksikkö:** [Mikä on moduulin keskeisin tietorakenne/käsite?]

```
Esimerkki: MemoryService:n primitiivi on MemoryItem
┌──────────────────────────────────────┐
│ MemoryItem                           │
├──────────────────────────────────────┤
│ id: UUID                             │
│ content: str                         │
│ category: Enum                       │
│ metadata: dict                       │
│ created_at: datetime                 │
└──────────────────────────────────────┘
```

---

## 2. Requirements & Acceptance Criteria

### REQ-01: [Vaatimuksen nimi] 🟢

[Vaatimuksen kuvaus - mitä järjestelmän pitää tehdä]

**Acceptance Criteria:**

| AC-ID | Kriteeri | Tyyppi |
|-------|----------|--------|
| AC-01 | [Mistä tiedän että tämä toimii?] | Functional |
| AC-02 | [Toinen kriteeri] | Functional |
| AC-03 | [Virhetilanne käsitellään oikein] | Error |

**Prioriteetti:** 🟢 MVP  
**Riippuvuudet:** [Muut moduulit/vaatimukset]

---

### REQ-02: [Toinen vaatimus] 🟢

[Kuvaus]

**Acceptance Criteria:**

| AC-ID | Kriteeri | Tyyppi |
|-------|----------|--------|
| AC-04 | [Kriteeri] | Functional |
| AC-05 | [Kriteeri] | Performance |

**Prioriteetti:** 🟢 MVP  
**Riippuvuudet:** REQ-01

---

### REQ-03: [Kolmas vaatimus] 🟡

[Kuvaus - Phase 2 ominaisuus]

**Acceptance Criteria:**

| AC-ID | Kriteeri | Tyyppi |
|-------|----------|--------|
| AC-06 | [Kriteeri] | Functional |

**Prioriteetti:** 🟡 Phase 2  
**Riippuvuudet:** REQ-01, REQ-02

---

## 3. API-määrittely (Black Box)

### 3.1 Julkinen rajapinta

```python
class [ModuuleName]:
    """
    [Moduulin kuvaus]
    
    Käyttöesimerkki:
        service = [ModuuleName](config)
        result = service.method(params)
    """
    
    def __init__(self, config: Config) -> None:
        """Alustaa moduulin annetulla konfiguraatiolla."""
        ...
    
    def method_one(self, param: Type) -> ReturnType:
        """
        [Metodin kuvaus]
        
        Args:
            param: [Parametrin kuvaus]
            
        Returns:
            [Palautusarvon kuvaus]
            
        Raises:
            ValueError: [Milloin heitetään]
        """
        ...
    
    def method_two(self, param: Type) -> ReturnType:
        """[Metodin kuvaus]"""
        ...
```

### 3.2 Tietorakenteet

```python
@dataclass
class [PrimaryDataClass]:
    """[Kuvaus]"""
    id: str
    field_one: Type
    field_two: Type
    created_at: datetime

class [EnumName](Enum):
    """[Kuvaus]"""
    VALUE_ONE = "value_one"
    VALUE_TWO = "value_two"
```

---

## 4. Data Model

### 4.1 Entiteetit

```
┌─────────────────────────────────────────────────────────────────┐
│  [Entity Name]                                                  │
├─────────────────────────────────────────────────────────────────┤
│  id: UUID (PK)                                                  │
│  field_one: VARCHAR(255)                                        │
│  field_two: TEXT                                                │
│  created_at: TIMESTAMP                                          │
│  updated_at: TIMESTAMP                                          │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Relaatiot

```
[Entity A] ──1:N──► [Entity B]
[Entity B] ──N:M──► [Entity C]
```

---

## 5. Edge Cases & Error Handling

### 5.1 Edge Cases

| Case | Syöte | Odotettu käytös |
|------|-------|-----------------|
| Tyhjä syöte | `""` | Palauttaa tyhjän listan |
| Null | `None` | Heittää `ValueError` |
| Erittäin pitkä syöte | 10MB teksti | Katkaistaan + varoitus |
| Erikoismerkit | Unicode, emojit | Käsitellään normaalisti |

### 5.2 Virhetilanteet

| Virhe | Trigger | Käsittely | Palautus |
|-------|---------|-----------|----------|
| `ValueError` | Virheellinen parametri | Validoi alussa | Selkeä virheilmoitus |
| `ConnectionError` | Ulkoinen palvelu alhaalla | Retry 3x | Fallback tai virhe |
| `TimeoutError` | Liian pitkä operaatio | 30s timeout | Keskeytä + ilmoita |

### 5.3 Virheviestit

```python
# Hyvä virheviesti
ValueError("Invalid category 'UNKNOWN'. Valid options: CONTEXT, DECISION, LEARNING")

# Huono virheviesti
ValueError("Invalid input")
```

---

## 6. Turvallisuus

### 6.1 Uhkamallit

| Uhka | Riski | Suojaus |
|------|-------|---------|
| SQL Injection | Korkea | Parametrisoidut kyselyt |
| API-avaimen vuoto | Kriittinen | Environment variables |
| DoS (liian isot pyynnöt) | Keskitaso | Rate limiting, max size |

### 6.2 Validointi

```python
def validate_input(data: str) -> str:
    """Validoi ja sanitoi käyttäjän syöte."""
    if len(data) > MAX_INPUT_SIZE:
        raise ValueError(f"Input exceeds maximum size of {MAX_INPUT_SIZE}")
    # Sanitoi...
    return sanitized_data
```

---

## 7. Suorituskyky

### 7.1 SLA-tavoitteet

| Operaatio | Tavoite | Max |
|-----------|---------|-----|
| Read | < 50ms | 200ms |
| Write | < 100ms | 500ms |
| Search | < 200ms | 1000ms |

### 7.2 Skaalautuvuus

| Metriikka | MVP | Target | Bottleneck |
|-----------|-----|--------|------------|
| Concurrent users | 1 | 10 | Tietokanta |
| Data volume | 10MB | 1GB | Indeksointi |
| Requests/sec | 10 | 100 | API rate limit |

---

## 8. Traceability Matrix (pohja)

*Täydennetään TECH_SPEC- ja CODE-vaiheissa.*

| REQ | AC | Task | Test Scenario | Test | Status |
|-----|-----|------|---------------|------|--------|
| REQ-01 | AC-01 | - | - | - | 🔲 |
| REQ-01 | AC-02 | - | - | - | 🔲 |
| REQ-01 | AC-03 | - | - | - | 🔲 |
| REQ-02 | AC-04 | - | - | - | 🔲 |
| REQ-02 | AC-05 | - | - | - | 🔲 |
| REQ-03 | AC-06 | - | - | - | 🔲 |

**Status:** ✅ Valmis | 🔶 Työn alla | 🔲 Ei aloitettu

---

## 9. Vaiheistus yhteenveto

| Prioriteetti | Vaatimukset | Perustelu |
|--------------|-------------|-----------|
| 🟢 MVP | REQ-01, REQ-02 | Ydinominaisuudet, välttämättömät |
| 🟡 Phase 2 | REQ-03 | Parantaa käytettävyyttä |
| 🔵 Phase 3 | - | Nice-to-have, myöhemmin |

---

## 10. Avoimet kysymykset

| # | Kysymys | Status | Päätös |
|---|---------|--------|--------|
| 1 | [Avoin kysymys] | 🔲 Avoin | - |
| 2 | [Toinen kysymys] | ✅ Ratkaistu | [Päätös ja perustelu] |

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.0 | [PÄIVÄMÄÄRÄ] | Ensimmäinen versio |

---

## Liittyvät dokumentit

| Dokumentti | Yhteys |
|------------|--------|
| RESEARCH_XX_[Nimi].md | Tutkimus joka johti tähän SPECiin |
| TECH_SPEC_XX_[Nimi].md | Tekninen toteutussuunnitelma |
| SPEC_YY_[Toinen].md | Riippuva moduuli |

---

*Dokumentti on osa [PROJEKTIN NIMI] -projektin dokumentaatiota.*
