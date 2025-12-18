# PROCESS: Implementation Strategy (Hybridimalli)

> **Versio:** 1.1  
> **Päivitetty:** 2025-12-16  
> **Tiedosto:** docs/process/PROCESS_Implementation_Strategy.md  
> **Projekti:** Claude API -suunnittelutyökalu

---

## Tausta: Miksi Hybridimalli?

### Kaksi ääripäätä

| Lähestymistapa | Vahvuudet | Heikkoudet |
|----------------|-----------|------------|
| **"All Specs First"** | Kokonaiskuva selkeä, integraatio mietitty | Speksataan tyhjiössä, "unknown unknowns" paljastuvat vasta koodatessa |
| **"Foundations First"** | Opitaan tekemällä, nopea feedback | Integraatio voi jäädä miettimättä, refaktorointiriski |

### Hybridimalli yhdistää parhaat puolet

```
┌─────────────────────────────────────────────────────────────────────────┐
│  PERIAATE: "Koodaa perusta, speksaa integraatio, sitten koodaa loput"  │
└─────────────────────────────────────────────────────────────────────────┘

• Foundation-moduulit (ClaudeService, MemoryService) voidaan koodata
  kun niiden TECH_SPEC on valmis - ne eivät riipu muista moduuleista

• Integraatiokerros (ContextManager) PITÄÄ speksata ennen koodausta
  koska se määrittää miten palikat toimivat yhdessä

• Integraatiokerroksen SPEC voi paljastaa uusia vaatimuksia
  alemman tason moduuleille → "Refinement"-vaihe
```

---

## Dependency-driven sequencing

> **Periaate:** Seuraava askel määräytyy siitä, mikä komponentti TARVITSEE edellisen komponentin tietoja - ei siitä, mikä tuntuu turvalliselta.

### Opit Session #24:stä (ClaudeService-valmistuminen)

| Oppi | Selitys |
|------|---------|
| **TECH_SPEC ennen Refinementia** | Integraatiokerroksen TECH_SPEC paljastaa mitä leaf nodet oikeasti tarvitsevat. Refinement ilman tätä kontekstia on spekulatiivista. |
| **Dependency-tieto koodista** | CODE-vaiheen jälkeen CC tietää tarkalleen toteutetut rajapinnat ja konkreettiset riippuvuudet |
| **"Turvallinen" ≠ optimaalinen** | Spekulatiivinen review voi olla overhead ilman oikeaa kontekstia |

### Orchestration-roolit teknisessä järjestyksessä

| Tilanne | Parempi näkemys | Miksi |
|---------|-----------------|-------|
| Tekninen järjestys / riippuvuudet | **CC** | Näkee toteutetun koodin ja rajapinnat |
| Strategiset päätökset / riskit | **claude.ai** | Näkee kokonaiskuvan |
| "Mitä tarvitaan seuraavaksi?" | **CC** | Konkreettinen dependency-tieto koodista |
| Arkkitehtuuripäätökset | **claude.ai** | Laajempi konteksti |

> **Käytännössä:** CODE-vaiheessa CC:llä on usein parempi näkemys tekniseen järjestykseen koska se on juuri toteuttanut edellisen komponentin ja tietää tarkalleen mitä rajapintoja on käytettävissä.

---

## Arkkitehtuurin riippuvuussuunta

```
┌─────────────────────────────────────────────────────────────────┐
│  APPLICATION LAYER (riippuu kaikesta alla)                     │
│  ├── Chat UI                                                    │
│  └── SessionManager                                             │
├─────────────────────────────────────────────────────────────────┤
│  INTEGRATION LAYER ("Aivot" - yhdistää kaiken)                 │
│  └── ContextManager                                             │
├─────────────────────────────────────────────────────────────────┤
│  FOUNDATION LAYER (itsenäiset, ei keskinäisiä riippuvuuksia)   │
│  ├── ClaudeService    ← Wrappaa Claude API:n                   │
│  └── MemoryService    ← Wrappaa muistijärjestelmän             │
└─────────────────────────────────────────────────────────────────┘

RIIPPUVUUSSUUNTA: Ylhäältä alas (↓)
KOODAUSSUUNTA: Alhaalta ylös (↑)
```

---

## Toteutusvaiheet

### Vaihe 1: Foundation CODE + Integration SPEC (rinnakkain)

| Tehtävä | Moduuli | Tyyppi | Tarkoitus |
|---------|---------|--------|-----------|
| CODE ClaudeService | ClaudeService | CODE | Opitaan API:n todelliset rajoitteet |
| SPEC ContextManager | ContextManager | SPEC | Määritellään miten "aivot" käyttävät palasia |

**Miksi rinnakkain?**
- ClaudeService ei riipu ContextManagerista → turvallinen koodata
- ContextManager SPEC paljastaa vaatimukset muille moduuleille
- Säästetään aikaa tekemällä molemmat samanaikaisesti

**Onnistumiskriteerit:**
- [ ] ClaudeService läpäisee kaikki TECH_SPEC_01 testiskenaariot
- [ ] SPEC_03_ContextManager määrittelee selkeästi rajapinnat

---

### Vaihe 2: TECH_SPEC Integration (ContextManager)

| Tehtävä | Moduuli | Tyyppi | Tarkoitus |
|---------|---------|--------|-----------|
| TECH_RESEARCH ContextManager | ContextManager | RESEARCH | Teknologiatutkimus |
| TECH_SPEC ContextManager | ContextManager | TECH_SPEC | Tekninen suunnittelu |

**Miksi ennen Refinementia?**
- TECH_SPEC paljastaa mitä ContextManager *oikeasti* tarvitsee leaf nodeilta
- Refinement ilman tätä kontekstia on spekulatiivista
- CC:n näkemys Session #24: "Review ilman CM-kontekstia on overhead"

**Onnistumiskriteerit:**
- [ ] TECH_RESEARCH_03 dokumentoi teknologiavalinnat
- [ ] TECH_SPEC_03 määrittelee rajapinnat ClaudeService:en ja MemoryService:en
- [ ] Selkeät vaatimukset leaf nodeille tunnistettu

---

### Vaihe 3: Refinement (ehdollinen)

| Tehtävä | Moduuli | Tyyppi | Tarkoitus |
|---------|---------|--------|-----------|
| Tarkista TECH_SPEC_02 | MemoryService | REVIEW | Vaatiiko CM muutoksia muistirajapintaan? |
| Päivitä tarvittaessa | MemoryService | UPDATE | Lisää puuttuvat metodit/metadata |

**Milloin tarpeen?**
- TECH_SPEC_03 paljasti uusia vaatimuksia MemoryService:lle
- Rajapinta ei tue kaikkia ContextManagerin tarpeita

**Tarkistuskysymykset (TECH_SPEC_03:n pohjalta):**
1. Palauttaako MemoryService riittävän metadatan ContextManagerille?
2. Tukeeko semantic search token-budjetointia?
3. Onko rajapinta riittävän joustava prioriteettijärjestykselle?

**Jos ei muutoksia tarvita → Siirry suoraan Vaihe 4:ään.**

**Onnistumiskriteerit:**
- [ ] TECH_SPEC_02 arvioitu TECH_SPEC_03:n kontekstissa
- [ ] Muutokset dokumentoitu (jos tarpeen)
- [ ] Ei avoimia kysymyksiä rajapinnoista

---

### Vaihe 4: Foundation CODE (MemoryService)

| Tehtävä | Moduuli | Tyyppi | Tarkoitus |
|---------|---------|--------|-----------|
| CODE MemoryService | MemoryService | CODE | Toteutetaan kun vaatimukset ovat lukitut |

**Miksi vasta nyt?**
- Vaihe 2 on varmistanut että rajapinta on oikea
- Vältytään turhalta refaktoroinnilta
- ContextManager SPEC on antanut "kuluttajan näkökulman"

**Onnistumiskriteerit:**
- [ ] MemoryService läpäisee kaikki TECH_SPEC_02 testiskenaariot
- [ ] Integraatiotestit ClaudeServicen kanssa toimivat

---

### Vaihe 5: Integration CODE (ContextManager)

| Tehtävä | Moduuli | Tyyppi | Tarkoitus |
|---------|---------|--------|-----------|
| CODE ContextManager | ContextManager | CODE | "Aivojen" toteutus |

**Miksi vasta nyt?**
- Foundation-palikat ovat valmiita ja testattuja
- Tiedetään tarkalleen mitä ClaudeService ja MemoryService tarjoavat
- Voidaan integroida oikeisiin toteutuksiin, ei oletuksiin

**Onnistumiskriteerit:**
- [ ] ContextManager orkestroi moduuleja oikein
- [ ] Token-budjetointi toimii käytännössä
- [ ] End-to-end testit läpäisevät

---

### Vaihe 6: Application Layer

| Tehtävä | Moduuli | Tyyppi | Tarkoitus |
|---------|---------|--------|-----------|
| SPEC SessionManager | SessionManager | SPEC→CODE | Istunnonhallinta |
| SPEC Chat UI | Chat UI | SPEC→CODE | Käyttöliittymä |

**Miksi viimeisenä?**
- Kaikki alitason palikat ovat valmiita
- UI voi hyödyntää todellista toiminnallisuutta
- Voidaan testata koko ketju

---

## Päätöksentekokriteerit

### Milloin moduuli on valmis koodattavaksi?

```
┌─────────────────────────────────────────────────────────────────┐
│  CHECKLIST: Onko moduuli valmis CODE-vaiheeseen?               │
├─────────────────────────────────────────────────────────────────┤
│  [ ] TECH_SPEC on valmis ja hyväksytty                         │
│  [ ] Kaikki riippuvuudet ovat joko:                            │
│      - Koodattu ja testattu, TAI                               │
│      - Ulkoisia palveluita (API:t), TAI                        │
│      - Mocattavissa testausta varten                           │
│  [ ] Rajapinnat ylempiin moduuleihin on määritelty (SPEC)      │
│  [ ] Ei avoimia kysymyksiä jotka vaikuttavat toteutukseen      │
└─────────────────────────────────────────────────────────────────┘
```

### Milloin SPEC pitää tehdä ennen koodausta?

| Moduulityyppi | SPEC ensin? | Perustelu |
|---------------|:-----------:|-----------|
| **Foundation (leaf)** | Ei pakollinen | Ei ylöspäin riippuvuuksia |
| **Integration** | **KYLLÄ** | Määrittää miten palikat toimivat yhdessä |
| **Application** | Suositeltava | Käyttäjäkokemus vaatii suunnittelua |

---

## Riskit ja mitigaatio

| Riski | Todennäköisyys | Vaikutus | Mitigaatio |
|-------|:--------------:|:--------:|------------|
| ContextManager vaatii muutoksia ClaudeServiceen | Matala | Keskisuuri | Black Box -rajapinta mahdollistaa laajennuksen |
| MemoryService refinement on suuri | Matala | Keskisuuri | TECH_SPEC_02 on jo kattava |
| Integraatiotestit paljastavat ongelmia | Keskisuuri | Matala | TDD + vaiheittainen integraatio |

---

## Yhteys muihin prosesseihin

| Prosessi | Yhteys |
|----------|--------|
| PROCESS_Code.md | TDD-workflow, commit-käytännöt CODE-vaiheissa |
| PROCESS_SPEC_Writing.md | SPEC-vaiheiden suoritus |
| PROCESS_Testing.md | Testiskenaariot ja DoD-kriteerit |
| KEHITYSLOKI.md | Tiivistelmä ja seuranta |

---

## Alkuperä

Tämä strategia syntyi keskustelussa Claude (claude.ai) + Gemini 2.5 Pro + käyttäjä (2025-12-15). 

**Geminin "Foundations First" -argumentti:**
- Foundation-moduulit ovat "leaf nodes" - eivät riipu muista
- "Unknown unknowns" paljastuvat vasta koodatessa
- Black Box -periaate mahdollistaa rajapinnan laajentamisen

**Clauden vasta-argumentti:**
- Integraatiokerros (ContextManager) EI saa koodata ilman SPECiä
- MemoryService voi vaatia refinementin CM:n tarpeiden perusteella
- "Integration Blindness" -riski ilman kokonaissuunnitelmaa

**Synteesi → Hybridimalli:**
- Koodaa Foundation-moduulit (turvallista)
- Speksaa Integration-kerros (pakollista)
- Refinement-vaihe varmistaa yhteensopivuuden
- Application-kerros viimeisenä

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.1 | 2025-12-16 | **Orchestration-opit:** Dependency-driven sequencing, TECH_SPEC ennen Refinementia, 6-vaiheinen malli |
| 1.0 | 2025-12-15 | Ensimmäinen versio, Hybridimallin dokumentointi |

---

## Liittyvät dokumentit

| Dokumentti | Yhteys |
|------------|--------|
| KEHITYSLOKI.md | Tiivistelmä Toteutusstrategiasta |
| PROCESS_Code.md | TDD-workflow CODE-vaiheissa |
| PROCESS_SPEC_Writing.md | SPEC-vaiheiden prosessi |
| TECH_SPEC_01_ClaudeService.md | Ensimmäinen CODE-vaiheeseen menevä moduuli |
| TECH_SPEC_02_MemoryService.md | Refinement-vaiheessa tarkistettava |
| SPEC_03_ContextManager.md | Integraatiokerroksen määrittely |

---

*Dokumentti on osa Claude API -suunnittelutyökalun prosessidokumentaatiota.*
