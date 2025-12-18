# PROCESS_Research_Methodology

> **Versio:** 1.1  
> **Päivitetty:** 2025-12-05  
> **Tarkoitus:** Yleinen tutkimusmetodologia kaikkeen tutkimukseen  
> **Käyttökohteet:** Market Research, Module Research, TECH_Research

---

## Muutokset v1.0 → v1.1

| Muutos | Kuvaus |
|--------|--------|
| ✅ **Tutkimuksen valmistelu** | Aiempi tieto, tutkimussuunnitelma ennen hakuja |
| ✅ **Falsifiointi** | Etsi aktiivisesti kumoavaa tietoa |
| ✅ **Saturaatio** | Milloin tutkimus on riittävä? |
| ✅ **Vertaileva validointi** | Milloin tarvitaan, milloin ei |
| ✅ **Hakuesimerkit** | Konkreettiset hakutermimallit |

---

## Milloin käyttää tätä ohjetta

Tämä dokumentti sisältää **yleiset tutkimuksen metataidot**. Käytä AINA kun teet tutkimusta, riippumatta kontekstista.

| Tutkimustyyppi | Kontekstiohje | Tämä dokumentti |
|----------------|---------------|-----------------|
| Market Research (Phase 0) | PROCESS_Market_Research.md | ✅ Metodologia |
| Module Research (Phase 2) | PROCESS_SPEC_Writing.md | ✅ Metodologia |
| TECH_Research (Phase 8) | PROCESS_SPEC_Writing.md | ✅ Metodologia |

---

## Tutkimuksen syvyysarviointi

Ennen tutkimuksen aloittamista, arvioi tarvittava syvyys:

| Tilanne | Prosessin syvyys | Perustelu |
|---------|------------------|-----------|
| **Ensimmäinen tutkimus** aiheesta | Täysi prosessi | Laatu vaatii perusteellisuutta |
| **Jatkotutkimus** tunnetusta aiheesta | Kevyempi prosessi | Rakennetaan olemassa olevan päälle |
| **Nopea faktatarkistus** | Minimaalinen | Ei tarvitse täyttä metodologiaa |

> **Oletusarvo:** Perusteellisuus. Oikotiet riskeeraavat kriittisten oivallusten ohittamisen.

---

## Tutkimuksen valmistelu

**Ennen hakujen aloittamista, tee nämä:**

### Aiempi tieto (Prior Knowledge)

- [ ] Mitä käyttäjä jo tietää aiheesta?
- [ ] Mitä oletuksia tehdään?
- [ ] Onko aiempaa tutkimusta tästä projektissa?
- [ ] Onko aiheeseen liittyviä dokumentteja (esim. edellisen prosessivaiheen dokumentaatio)?

> **Miksi tärkeä:** Dokumentoi lähtötilanne, jotta näet mitä opit. 
> Tunnista oletukset, jotta voit haastaa ne.

### Tutkimussuunnitelma (kevyt versio)

Ennen hakuja, määrittele:

```
1. TUTKIMUSKYSYMYKSET
   - Mitä haluan tietää?
   - Mihin kysymyksiin tarvitsen vastauksen?

2. LÄHDETYYPIT (kontekstin mukaan)
   - Mitkä lähteet ovat todennäköisimmin hyödyllisiä?
   - Ks. "Lähteiden hierarkia" -osio

3. HAKUSTRATEGIA
   - Millä termeillä aloitan?
   - Mitä kilpailijoita/projekteja tutkin?

4. LOPETUSKRITEERIT (jos tiedossa)
   - Milloin tutkimus on riittävä?
   - Mitä pitää löytyä?
```

> **Huom:** Lopetuskriteerit eivät aina ole tiedossa etukäteen. 
> Jos tiedät mitä etsit (esim. "miten persistent memory toteutetaan"), 
> määrittele kriteerit. Jos et, iteroi tutkimuksen edetessä.

---

## Tutkimuskysymysten muodostaminen

### Kolme kysymystasoa

| Taso | Tyyppi | Esimerkki |
|------|--------|-----------|
| **Perus** | Mitä? Miten? | "Miten streaming toteutetaan?" |
| **Vertaileva** | Mikä parhaiten? | "Redis vs SQLite - kumpi parempi?" |
| **Syventävä** | Miksi? Entä jos? | "Mitä tapahtuu edge caseissa?" |

### Syventävien kysymysten checklist

Jokaisen peruskysymyksen jälkeen, kysy:

- [ ] "Mitä edge caseja en ole miettinyt?"
- [ ] "Mikä voisi epäonnistua tuotannossa?"
- [ ] "Miten muut ovat ratkaisseet saman ongelman?"
- [ ] "Onko tutkimusta/dataa joka tukee/kumoaa oletukseni?"
- [ ] "Mitä en tiedä mitä en tiedä?" (unknown unknowns)

---

## Lähteiden hierarkia (kontekstisidonnainen)

### Lähdetyypit

| Lähdetyyppi | Kuvaus | Esimerkki |
|-------------|--------|-----------|
| **GitHub-projektit** | Avoin lähdekoodi, toteutusesimerkit | MemGPT, LangChain |
| **API-dokumentaatiot** | Viralliset rajapintakuvaukset | Anthropic API docs |
| **Kilpailevat tuotteet** | Dokumentaatio, käyttöohjeet, APIt | Netvisor, Cursor, Fennoa |
| **Ammattilaiskeskustelut** | Foorumit, blogit, insightit | Reddit, HN, Dev.to |
| **Metatiedot** | Alustan omat ohjeet ja best practices | Anthropic docs, Claude.ai ohjeet |
| **Akateeminen tutkimus** | Tieteelliset julkaisut | arXiv, Google Scholar |

### Kontekstin mukainen priorisointi

**Valitse lähteiden painotus tutkimuskontekstin mukaan:**

| Tutkimuskonteksti | Ensisijainen | Toissijainen | Täydentävä |
|-------------------|--------------|--------------|------------|
| **Tekninen toteutus** | GitHub-projektit, API docs | Ammattilaiskeskustelut | Akateeminen |
| **Arkkitehtuuri** | Blogit, keskustelut, kilpailijat | GitHub | Akateeminen |
| **Uusi teknologia** | Akateeminen, viralliset docs | GitHub | Keskustelut |
| **Integraatiot** | Kilpailevat tuotteet, API docs | GitHub | Blogit |
| **Best practices** | Ammattilaiskeskustelut, blogit | GitHub | Kilpailijat |

### Lähteiden arviointikriteerit

| Kriteeri | Kysymys |
|----------|---------|
| **Tyyppi** | Mikä lähdetyyppi? (ks. yllä) |
| **Luotettavuus** | Kuka kirjoitti? Mikä organisaatio? |
| **Relevanssi** | Vastaako suoraan tutkimuskysymykseen? |
| **Tuoreus** | Milloin julkaistu? Onko tieto vanhentunut? |
| **Sovellettavuus** | Onko konteksti vastaava meidän projektiin? |

---

## Vertaileva validointi

### Milloin tarvitaan?

**Suunnittelupäätöksissä** (ei faktakysymyksissä), etsi vähintään 2 eri toteutusta:

- Mikä toimii hyvin ensimmäisessä?
- Mikä on tehty paremmin toisessa?
- Mitä virheitä voimme välttää?

**Esimerkki:** Netvisor vs Fennoa - sama moduuli, eri lähestymistavat. Toinen kömpelö, toinen moderni → opimme mitä välttää ja mitä kopioida.

### Milloin EI tarvita?

| Tilanne | Esimerkki | Miksi yksi lähde riittää? |
|---------|-----------|---------------------------|
| **Virallinen dokumentaatio** | "Miten kutsun Claude API:a?" | Anthropic docs on auktoritatiivinen |
| **Standardi/protokolla** | "Miten OAuth flow toimii?" | RFC/standardi määrittelee |
| **Yksinkertainen fakta** | "Tukeeko SQLite-vec HNSW:tä?" | Kyllä/ei -vastaus |

### Nyrkkisääntö

```
FAKTA tai STANDARDI → yksi lähde riittää
PÄÄTÖS tai SUUNNITTELU → vertailu hyödyllinen
```

---

## Web Search -strategia

### Hakujen vaiheistus

```
VAIHE 1: Yleiskatsaus (2-3 hakua)
├── Laaja haku aiheesta
├── "[aihe] best practices"
└── "[aihe] architecture patterns"

VAIHE 2: Toteutusesimerkit (3-5 hakua)
├── "github [aihe] implementation"
├── "[kilpailija] [aihe] how to"
└── "[aihe] tutorial"

VAIHE 3: Ongelmat ja reunatapaukset (2-3 hakua)
├── "[aihe] common pitfalls"
├── "[aihe] edge cases"
└── "[aihe] mistakes to avoid"

VAIHE 4: Standardit ja konventiot (1-2 hakua)
├── "[aihe] standards protocols"
└── "[aihe] conventions"
```

### Hakutermien esimerkkejä

| Hakutyyppi | Esimerkki |
|------------|-----------|
| **Käyttäjäpalaute** | `site:reddit.com [aihe] frustrations` |
| **GitHub-projektit** | `github [aihe] implementation` |
| **Vertailut** | `[A] vs [B] comparison` |
| **Ongelmat** | `[aihe] "doesn't work"` tai `[aihe] issues` |
| **Hacker News** | `site:news.ycombinator.com [aihe]` |
| **Stack Overflow** | `site:stackoverflow.com [aihe] [virhe]` |

### Hakujen optimointi

| Periaate | Selitys |
|----------|---------|
| **Laajasta kapeaan** | Aloita yleisellä haulla, tarkenna tulosten perusteella |
| **Vältä toistoa** | Dokumentoi tehdyt haut, älä toista samoja |
| **Lue kokonaan** | Käytä web_fetch koko artikkeliin, älä luota snippetteihin |
| **GitHub-haut erikseen** | "github [aihe]" löytää toteutusesimerkkejä |

### "Read Full Source" -sääntö

> **Älä koskaan luota pelkkään hakutuloksen snippettiin.** Hae aina koko artikkeli/dokumentti web_fetch-työkalulla ennen kuin teet johtopäätöksiä.

Poikkeus: Jos snippet selvästi vastaa yksinkertaiseen faktakysymykseen (esim. "mikä on X:n versio").

---

## Falsifiointi

> **Älä etsi vain vahvistusta - etsi aktiivisesti tietoa joka KUMOAA 
> alkuperäisen hypoteesisi.**

### Miksi tärkeä?

Ihmisillä (ja AI:lla) on taipumus etsiä vahvistavaa tietoa (confirmation bias). Tämä johtaa:
- Huonoihin päätöksiin
- Yllätyksiin myöhemmin
- "Olisi pitänyt tietää" -tilanteisiin

### Käytännössä

Kun löydät vastauksen, kysy:

- [ ] "Mitä hakuja tekisin jos haluaisin todistaa tämän VÄÄRÄKSI?"
- [ ] "Onko kukaan eri mieltä? Miksi?"
- [ ] "Missä tilanteissa tämä EI toimisi?"

**Esimerkki:**
```
OLETUS: "SQLite on paras valinta muistitietokantaan"

FALSIFIOINTIHAUT:
- "SQLite limitations large scale"
- "SQLite vs PostgreSQL when to use"
- "SQLite problems production"

TULOS: Löytyi että SQLite ei skaalaudu hyvin 
       samanaikaisiin kirjoituksiin → huomioitava suunnittelussa
```

---

## Saturaatio (milloin lopettaa?)

### Tunnusmerkit

Tutkimus on riittävä kun:

- [ ] Uudet lähteet eivät enää tuota uutta tietoa
- [ ] Samat vastaukset/patternit toistuvat eri lähteissä
- [ ] Tutkimuskysymyksiin on vastattu
- [ ] Tiedät tarpeeksi tehdäksesi päätöksen

### Käytä saturaatiota "linssinä"

Jokaisen hakukierroksen jälkeen kysy:

> "Tuottiko tämä haku oikeasti uutta tietoa vai toistoa?"

Jos 2-3 peräkkäistä hakua ei tuota uutta → tutkimus on todennäköisesti riittävä.

### Huomio

Saturaatio EI tarkoita että tiedät KAIKEN - se tarkoittaa että tiedät TARPEEKSI tähän päätökseen. Voit aina palata tutkimaan lisää myöhemmin.

---

## Laadunvarmistus

### ⚠️ "Research Slop" -varoitus

AI voi tuottaa uskottavan kuuloista mutta epätarkkaa tietoa. Varmista:

- [ ] Väite löytyy oikeasta lähteestä (ei vain "yleisesti tiedetään")
- [ ] Lähde on luotettava ja ajantasainen
- [ ] Kriittisissä kohdissa: useampi lähde tukee samaa väitettä

### Citation Protocol

Jokainen väite tarvitsee lähteen:

```markdown
✅ OIKEIN:
"Anthropic suosittelee exponential backoff -strategiaa API-kutsuissa [1]."

[1] https://docs.anthropic.com/...

❌ VÄÄRIN:
"Yleisesti suositellaan exponential backoff -strategiaa."
```

### Epävarmuuden dokumentointi

> **Dokumentoi mitä EI löytynyt yhtä huolellisesti kuin mitä löytyi.**

```markdown
## Aukot tutkimuksessa

- Ei löytynyt vertailua X vs Y tuotantoympäristössä
- Suorituskykydataa ei saatavilla yli 100K dokumentin datasetillä
- Turvallisuusauditoinnista ei julkista tietoa
```

---

## Dokumentointi tutkimuksen aikana

### Mitä dokumentoidaan

| Kategoria | Sisältö |
|-----------|---------|
| **Lähtötilanne** | Mitä tiedettiin ennen tutkimusta, oletukset |
| **Hakustrategiat** | Käytetyt hakutermit, lähteet |
| **Lähteiden arviointi** | Tyyppi, luotettavuus, relevanssi |
| **Ristiriidat** | Lähteet jotka ovat eri mieltä |
| **Aukot** | Mitä EI löytynyt |
| **Yllätykset** | Odottamattomat löydökset, kumotut oletukset |
| **Jatkokysymykset** | Uudet tutkimustarpeet jotka nousivat esiin |

### RESEARCH-dokumentin rakenne

```markdown
# RESEARCH_XX_[Nimi]

> **Versio:** 1.0
> **Tutkimuspäivä:** YYYY-MM-DD
> **Konteksti:** [Market Research / Module Research / TECH_Research]

## Executive Summary
[2-3 kappaletta: Keskeiset löydökset]

## Lähtötilanne
[Mitä tiedettiin ennen? Mitkä olivat oletukset?]

## Tutkimuskysymykset
[Mitä tutkittiin?]

## Lähteet ja metodologia
[Mitä lähteitä käytettiin? Miksi? Mitä hakuja tehtiin?]

## Löydökset
[Järjestetty tutkimuskysymysten mukaan]

## Ristiriidat ja epävarmuudet
[Missä lähteet olivat eri mieltä? Mitä ei löytynyt?]

## Kumotut oletukset
[Mitkä alkuperäiset oletukset osoittautuivat vääriksi?]

## Johtopäätökset ja suositukset
[Mitä tämä tarkoittaa projektille?]

## Jatkotutkimustarpeet
[Mitä pitäisi tutkia lisää?]

## Lähteet
[Numeroitu lista kaikista lähteistä]
```

---

## Iterointi ja kriittinen arviointi

### Iteroinnin periaate

Ennen tutkimuksen päättämistä, tee **aina** yksi itsenäinen iteraatio:

- [ ] Tarkista kriittisesti: Mitä puuttuu?
- [ ] Mitkä oletukset jäivät testaamatta?
- [ ] Onko teknisiä edellytyksiä joita ei tutkittu?
- [ ] Tehtiinkö falsifiointihakuja?
- [ ] **Ehdota aina vähintään yksi parannus** - lähes aina on jotain

### Kriittisen arvioinnin kysymykset

- Pitäisikö jotain aluetta syventää?
- Pitäisikö tutkimuskysymystä muokata?
- Onko uusia lähteitä joita pitäisi konsultoida?
- Syntyikö jatkotutkimuskysymyksiä?

---

## Deep Thinking -strategia

### Milloin käyttää

| Vaihe | Deep Thinking | Perustelu |
|-------|---------------|-----------|
| **Tiedonhaku** | OFF (tai oletus) | Ei vaikuta hakutuloksiin |
| **Synteesi** | ON | Auttaa monimuuttujayhtälöissä |
| **Johtopäätökset** | ON | Parempi analyysi ja priorisointi |

### Käytännössä

Kun siirryt tiedonhausta synteesiin/johtopäätöksiin:

> "Tiedonhaku valmis. Laita Deep Thinking PÄÄLLE ennen synteesivaihetta."

---

## Onnistumiskriteerit

### ✅ Hyvä tutkimus

- Lähtötilanne ja oletukset dokumentoitu
- Tutkimuskysymyksiin vastattu kattavasti
- Lähteet dokumentoitu ja arvioitu
- Kontekstiin sopivat lähteet priorisoitu
- Falsifiointia tehty (etsitty kumoavaa tietoa)
- Ristiriidat ja aukot tunnistettu
- Saturaatio saavutettu (uudet haut eivät tuota uutta)
- Vähintään yksi iteraatiokierros tehty

### ❌ Huono tutkimus

- Ei tiedetä mitä oletettiin alussa
- Pinnallinen (vain 2-3 lähdettä, ei analyysiä)
- Lähteet puuttuvat tai heikkolaatuisia
- Kontekstiin sopimattomat lähteet
- Ei falsifiointia (vain vahvistava tieto)
- Ei kriittistä arviointia
- Epävarmuutta ei dokumentoitu
- "Research slop" - uskottavaa mutta tarkistamatonta

---

## Muistisäännöt

> **"Perusteellisuus > Nopeus"** - Uusissa aiheissa oikotiet maksavat myöhemmin.

> **"Dokumentoi lähtötilanne"** - Mitä tiesit ennen? Mitkä olivat oletukset?

> **"Konteksti ohjaa lähteitä"** - Valitse lähdetyypit tutkimuskontekstin mukaan.

> **"Laajasta kapeaan"** - Aloita yleisellä haulla, tarkenna iteratiivisesti.

> **"Lue kokonaan"** - Snippetit eivät riitä, hae koko artikkeli.

> **"Falsifioi"** - Etsi aktiivisesti tietoa joka kumoaa oletuksesi.

> **"Tarkkaile saturaatiota"** - Tuottavatko uudet haut vielä uutta tietoa?

> **"Dokumentoi aukot"** - Mitä EI löytynyt on yhtä tärkeää kuin mitä löytyi.

> **"Vertaile päätöksissä"** - Fakta = yksi lähde riittää, päätös = vertailu hyödyllinen.

> **"Iteroi aina"** - Tee vähintään yksi kriittinen tarkistuskierros.

> **"Lähde jokaiselle väitteelle"** - Ei "yleisesti tiedetään" -väitteitä.

> **"Deep Thinking synteesiin"** - Laita päälle kun siirryt johtopäätöksiin.

---

## Liittyvät dokumentit

| Dokumentti | Yhteys |
|------------|--------|
| **PROCESS_Market_Research.md** | Phase 0 konteksti (projektin aloitus) |
| **PROCESS_SPEC_Writing.md** | Module Research konteksti (Phase 2, 8) |
| **RESEARCH_Methodology_Comparison.md** | Metodologian validointi ja kehitysehdotukset |

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.1 | 2025-12-05 | Tutkimuksen valmistelu, falsifiointi, saturaatio, vertaileva validointi, hakuesimerkit |
| 1.0 | 2025-12-05 | Ensimmäinen versio - metatason tutkimusmetodologia |

---

*Tämä dokumentti on osa Claude API -suunnittelutyökalun prosessidokumentaatiota.*
