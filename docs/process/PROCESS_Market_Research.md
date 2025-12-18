# PROCESS_Market_Research

> **Versio:** 1.1  
> **Päivitetty:** 2025-12-14  
> **Edellinen:** v1.0 (2025-11-26)  
> **Tiedosto:** v1_1_PROCESS_Market_Research.md  
> **Tarkoitus:** Phase 0: Market Research and Idea Evaluation/Evolution  
> **Liittyy:** PROCESS_SPEC_Writing.md (jatkaa Phase 1:een)  

---

## Muutokset v1.0 → v1.1

| Muutos | Kuvaus |
|--------|--------|
| ✅ **Research Methodology viittaus** | Vaihe 2 viittaa PROCESS_Research_Methodology.md:iin |
| ✅ **Web Search lyhennetty** | Yleinen hakustrategia siirretty metodologiadokumenttiin |
| ✅ **Liittyvät dokumentit päivitetty** | Lisätty Research Methodology |

---

## Yleiskatsaus

**Phase 0** on valinnainen, mutta erittäin arvokas vaihe **ennen** teknistä suunnittelua. Sen tavoite on:

1. **Ymmärtää markkinaa** - Mitä muut tekevät oikein ja väärin?
2. **Validoida idea** - Onko tälle oikeasti tarvetta?
3. **Kehittää ideaa** - Miten idea voi evoluoitua tutkimuksen perusteella?
4. **Priorisoida MVP** - Mitkä ominaisuudet ovat kriittisimpiä?

---

## Milloin Phase 0 kannattaa käyttää?

### ✅ SUOSITELTU (Phase 0 hyödyllinen)

| Projektin tyyppi | Miksi? |
|------------------|--------|
| **Startup / uusi tuote** | Markkinaymmärrys kriittinen |
| **SaaS-sovellus** | Kilpailu kovaa, differentiaatio tärkeää |
| **B2C-sovellus** | Käyttäjätarpeet monimutkaisia |
| **Pitkäaikainen projekti** | Investointi kannattaa (6kk+) |
| **Tiimiprojekti** | Vision_Doc auttaa yhteisymmärryksessä |

### ⚠️ EHKÄ (harkitse tapauskohtaisesti)

| Projektin tyyppi | Huomio |
|------------------|--------|
| **Sisäinen työkalu** | Jos käyttäjäryhmä pieni ja tuttu, kevyempi research riittää |
| **Enterprise B2B** | Jos asiakkaan vaatimukset selvät, Phase 0 voi olla overkill |

### ❌ EI SUOSITELLA (Phase 0 liian raskas)

| Projektin tyyppi | Miksi? |
|------------------|--------|
| **Tekninen kirjasto** | Teknologia-driven ok, ei tarvita liiketoimintakontekstia |
| **Proof of concept** | Nopeutta tärkeämpi kuin perusteellisuus |
| **Hyvin määritelty replica** | Jos kopioidaan tunnettua tuotetta 1:1 |
| **<2 viikon projekti** | Token-budjetti ja aika ei kannata |

---

## Prosessin vaiheet

```
┌─────────────────────────────────────────────────────────────────┐
│              PHASE 0: MARKET RESEARCH & IDEA EVOLUTION          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. VALMISTELU       Määrittele research-scope                  │
│         │                                                       │
│         ▼                                                       │
│  2. TUTKIMUS         Web search + analyysi (~30-45 min)         │
│         │            → Katso PROCESS_Research_Methodology.md    │
│         │            → MARKET_RESEARCH_[Project].md             │
│         ▼                                                       │
│  3. SYNTEESI         Vision Doc kirjoitus                       │
│         │            → VISION_DOC_[Project].md                  │
│         ▼                                                       │
│  4. USER REVIEW      Käyttäjä hyväksyy vision                   │
│         │                                                       │
│         ▼                                                       │
│  5. PRIORISOINTI     MVP vs Phase 2/3 + Roadmap                 │
│         │            → Päivitetty Vision Doc                    │
│         ▼                                                       │
│  → Phase 1 (RESEARCH_XX) alkaa                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Vaihe 1: Valmistelu

### Claude kysyy käyttäjältä:

```
"Aloitetaanko projekti Market Research -vaiheella (Phase 0)?"

Tämä vie ~30-45 minuuttia ja ~20K tokenia, mutta antaa:
✅ Kilpailijakartan ja best practices
✅ Käyttäjätarpeiden priorisoinnin
✅ Vision dokumentin sidosryhmille
✅ Realistisen MVP-scopen

Jatketaanko?
```

### Jos käyttäjä hyväksyy → määrittele scope

**Claude kysyy:**

```
"Määrittele tutkimuksen laajuus:

1. Nopea (15-20 min)
   - 3-5 kilpailijaa
   - Pinnallinen analyysi
   - Lyhyt Vision Doc

2. Keskitaso (30-45 min) [SUOSITELTU]
   - 5-8 kilpailijaa
   - Käyttäjäfoorumit (Reddit, HN)
   - Kattava Vision Doc + Roadmap

3. Syvällinen (60-90 min)
   - 10+ kilpailijaa
   - Research papers
   - Yksityiskohtainen markkina-analyysi

Suositus: Valitse 2 (keskitaso) ensimmäisellä kerralla."
```

---

## Vaihe 2: Tutkimus

> ⚠️ **PAKOLLINEN:** Lue ensin [PROCESS_Research_Methodology.md](docs/process/v1_1_PROCESS_Research_Methodology.md)  
> Sisältää yleiset tutkimuksen metataidot: falsifiointi, saturaatio, lähdehierarkia, laadunvarmistus.

### Tutkimuskysymykset (Market Research -spesifiset)

Claude laatii tutkimuskysymykset järjestelmällisesti:

```
MARKET RESEARCH -MALLI:

1. KILPAILIJAT
   - Ketkä ovat suorat kilpailijat?
   - Mitä ominaisuuksia heillä on?
   - Mikä on heidän hintamalli?
   - Mitä he tekevät hyvin? Huonosti?

2. KÄYTTÄJÄTARPEET
   - Mitä käyttäjät valittavat nykyisistä ratkaisuista?
   - Mitkä ominaisuudet mainitaan useimmiten?
   - Mitä "would be nice to have" -asioita?

3. BEST PRACTICES
   - Mitkä design patternit ovat yleisiä?
   - Mitä teknologioita käytetään?
   - Mitkä arkkitehtuuripäätökset toistuvat?

4. TRENDIT
   - Mihin suuntaan ala kehittyy?
   - Mitkä teknologiat nousevat?
   - Mitä uusia käyttötapauksia ilmenee?
```

### Web Search -strategia

> **Yleiset hakustrategiat:** Katso PROCESS_Research_Methodology.md

**Market Research -spesifiset hakumallit:**

```
KILPAILIJAT (5-8 hakua):
- "[tuotekategoria] alternatives"
- "[kilpailija] vs alternatives"
- "best [tuotekategoria] tools"

KÄYTTÄJÄPALAUTE (3-5 hakua):
- "site:reddit.com [tuotekategoria] frustrations"
- "site:news.ycombinator.com [kilpailija]"
- "[tuotekategoria] pain points"

BEST PRACTICES (3-5 hakua):
- "[teknologia] best practices"
- "[arkkitehtuuri] patterns"
```

### Lähteiden analysointi

**Jokaisesta lähteestä poimi:**

```
KILPAILIJA-ANALYYSI:
┌─────────────────────────────────────────────────────────┐
│ Nimi:           MemGPT                                  │
│ URL:            https://memgpt.ai                       │
│ Ydinominaisuus: Hierarchical memory system              │
│ Hinnoittelu:    Free tier + $20/mo                      │
│                                                         │
│ ✅ Hyvin:       - Pitkä muisti                          │
│                 - Avoin lähdekoodi                      │
│                                                         │
│ ❌ Huonosti:    - Monimutkainen setup                   │
│                 - UI ei intuitiivinen                   │
│                                                         │
│ 💡 Opimme:      Hierarchical memory = hyvä idea,       │
│                 mutta UX pitää olla yksinkertainen      │
└─────────────────────────────────────────────────────────┘
```

### Dokumentointi

Tallenna kaikki löydökset → **MARKET_RESEARCH_[Project].md**

**Rakenne:**

```markdown
# MARKET_RESEARCH_Claude_Planning_Tool

> **Versio:** 1.0
> **Tutkimuspäivä:** 2025-11-26
> **Tutkija:** Claude (ohjattu käyttäjän toimesta)

## Executive Summary

[2-3 kappaletta: Keskeiset löydökset]

## Kilpailija-analyysi

### 1. MemGPT
[Yksityiskohtainen analyysi]

### 2. LangChain
[Yksityiskohtainen analyysi]

...

## Käyttäjätarpeet

### Priorisoitu lista (esiintymisfrekvenssin mukaan)

| Tarve | Frekvenssi | Lähde |
|-------|-----------|-------|
| Muisti sessionien välillä | 87% | Reddit, HN |
| Pitkä konteksti (>200K) | 76% | HN, Discord |
| Dokumentaation automaatio | 54% | Reddit |

## Best Practices

### Muistiarkkitehtuuri
- Semantic search > keyword search
- Markdown > proprietary formats
- Gradual decay (vanhat haalistuvat)

### UI/UX
- Session branching (Notion AI)
- Auto-save (Cursor)
- Keyboard shortcuts (suosittua devien keskuudessa)

## Teknologiatrendit

- Vector databases (Pinecone, Weaviate)
- Embedding models (OpenAI, Cohere)
- Long context models (Claude, GPT-4 Turbo)

## Suositukset

[Claude tiivistää: Mitä meidän pitäisi tehdä?]

---

*Lähteet: [Lista kaikista käytetyistä lähteistä]*
```

---

## Vaihe 3: Synteesi (Vision Doc)

### Vision Doc -rakenne

```markdown
# VISION_DOC_Claude_Planning_Tool

> **Versio:** 1.0
> **Päivitetty:** 2025-11-26
> **Status:** Draft / Reviewed / Approved

---

## 1. Ongelma (Problem Statement)

[Mikä on ongelma jota ratkomme?]

## 2. Ratkaisu (Solution)

[Miten ratkaisemme tämän?]

## 3. Kohderyhmä (Target Audience)

[Kenelle tämä on?]

## 4. Kilpailuetu (Competitive Advantage)

[Miksi valita meidän ratkaisu kilpailijoiden sijaan?]

## 5. MVP-scope

[Mitä rakennetaan ENSIKSI?]

## 6. Success Metrics

[Miten mittaamme onnistumisen?]

## 7. Development Roadmap

[Aikataulu ja priorisointi]

## 8. Avoimet kysymykset

[Mitä emme vielä tiedä?]

## 9. Appendix: Research Summary

[Linkki täyteen MARKET_RESEARCH dokumenttiin]
```

---

## Vaihe 4: User Review

### Claude esittää käyttäjälle:

```
"Vision Doc v1.0 valmis. Tarkista:

1. Onko Problem Statement oikea?
2. Puuttuuko kohderyhmästä joku tärkeä segmentti?
3. Onko MVP-scope realistinen?
4. Haluatko lisätä/poistaa jotain Phase 2/3:sta?

TÄRKEÄ: Vision Doc on sinun dokumenttisi. Voit hylätä tai muuttaa
mitä tahansa, vaikka tutkimus ehdottaisi muuta."
```

### Käyttäjä päättää:

```
Vaihtoehto A: Hyväksyy sellaisenaan
   → Phase 0 valmis, siirrytään Phase 1:een

Vaihtoehto B: Pyytää muutoksia
   → Claude päivittää Vision Doc → uusi review

Vaihtoehto C: Hylkää kokonaan
   → Phase 0 keskeytetään, siirrytään suoraan Phase 1:een
```

---

## Vaihe 5: Priorisointi & Roadmap

### Development Roadmap (tarkempi)

Jos Vision Doc hyväksytty → laaditaan tarkempi roadmap:

```markdown
# DEVELOPMENT_ROADMAP_Claude_Planning_Tool

## MVP (Phase 1) - Q1 2025

### Moduulit prioriteettijärjestyksessä:

1. **ClaudeService** (2 viikkoa)
2. **MemoryService** (3 viikkoa)
3. **ContextManager** (2 viikkoa)
4. **SessionManager** (2 viikkoa)
5. **Chat UI** (3 viikkoa)

**Yhteensä: ~12 viikkoa (3 kuukautta)**

## Phase 2 - Q2 2025

1. **Semantic Search** (2 viikkoa)
2. **Team Features** (3 viikkoa)
3. **Export System** (2 viikkoa)

## Phase 3 - Q3 2025

[Placeholder - määritellään Phase 2:n jälkeen]
```

---

## Token-budjetti

### Phase 0 kustannukset:

| Vaihe | Tokenit | Aika |
|-------|---------|------|
| Valmistelu | ~1K | 5 min |
| Tutkimus (web_search) | ~10-15K | 30-45 min |
| MARKET_RESEARCH kirjoitus | ~5K | 10 min |
| VISION_DOC kirjoitus | ~3K | 10 min |
| Review & iteraatio | ~2K | 10 min |
| **YHTEENSÄ** | **~20K** | **~75 min** |

---

## Onnistumisen kriteerit

### ✅ Hyvä Phase 0 -tutkimus sisältää:

```
1. KILPAILIJA-ANALYYSI:
   - Vähintään 5 relevanttia kilpailijaa
   - Yksityiskohtaiset +/- listat
   - Selkeät oppimislöydökset

2. KÄYTTÄJÄTARPEET:
   - Priorisoitu lista (frekvenssi)
   - Konkreettiset lähteet
   - Yllättäviä löydöksiä (ei vain itsestäänselvyyksiä)

3. VISION DOC:
   - Selkeä ongelma + ratkaisu
   - Realistinen MVP-scope
   - Mitattavat success metrics
   - Käyttäjän hyväksymä

4. ROADMAP:
   - Moduulit priorisoitu
   - Aikataulu-estimaatit
   - Riippuvuudet tunnistettu
```

### ❌ Huono Phase 0 -tutkimus:

```
- Pinnallinen (vain 2-3 kilpailijaa, ei analyysiä)
- Geneerinen (ei konkreettisia löydöksiä)
- Vision Doc liian abstrakti (ei MVP-scopea)
- Roadmap puuttuu tai epärealistinen
```

---

## Dokumenttien sijainnit

```
MARKET_RESEARCH_[Project].md  → docs/research/
VISION_DOC_[Project].md        → docs/
DEVELOPMENT_ROADMAP_[Project].md → docs/ (optional)
```

---

## Muistisäännöt

> **"Systematiikka > Nopeus"**  
> Älä tee kevyesti Phase 0:aa. 30 minuuttia tutkimusta säästää viikkoja väärään suuntaan rakentamista.

> **"Käyttäjä päättää"**  
> Research ehdottaa, Vision Doc dokumentoi, mutta käyttäjä tekee lopulliset päätökset.

> **"MVP on minimalistinen"**  
> Jos ominaisuus ei ratkaise ydinongelmaa, se on Phase 2/3:ssa.

> **"Roadmap on elävä"**  
> Vision Doc lukitaan, mutta Roadmap voi muuttua oppimisen myötä.

---

## Liittyvät dokumentit

| Dokumentti | Yhteys |
|------------|--------|
| **PROCESS_Research_Methodology.md** | Yleiset tutkimuksen metataidot (Vaihe 2) |
| **PROCESS_SPEC_Writing.md** | Phase 1 alkaa täältä (RESEARCH_XX) |
| **System_Prompt.md** | Ohjeet milloin käyttää Phase 0:aa |
| **INDEX.md** | Dokumenttien sijainnit |

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.1 | 2025-12-14 | Research Methodology viittaus, Web Search lyhennetty, DRY-refaktorointi |
| 1.0 | 2025-11-26 | Ensimmäinen versio - Phase 0 prosessiohjeet |

---

*Tämä dokumentti on osa Claude API -suunnittelutyökalun prosessidokumentaatiota.*
