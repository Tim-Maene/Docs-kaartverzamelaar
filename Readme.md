# Propositie: Card Manager

> Status: **concept**, ter bespreking
> Opgesteld door: Tim Maene
> Datum: 17/10/2026

Alles wat als **[keuze]** is gemarkeerd, moeten we samen beslissen. Onderaan staat een overzicht van de besluiten die we nodig hebben.

---

## 1. Idee

Een kaartbeheerder voor verzamelkaarten, te beginnen met **Magic: The Gathering**. Gebruikers beheren hun collectie via een website en een mobiele app, en kunnen later kaarten scannen met de camera. Het project is bedoeld om te leren, samen iets af te maken en onze programmeervaardigheden te tonen.

## 2. Doelen

- Een werkende collectiebeheerder: zoeken, toevoegen, hoeveelheid, staat en foil bijhouden
- Een kaartscanner die kaarten herkent in de database
- Een architectuur die later meer spellen aankan
- Een nette, goed gedocumenteerde repo die we kunnen laten zien aan werkgevers of opleidingen

## 3. Buiten scope (voorlopig)

- Andere spellen dan Magic
- Marktplaats- of handelsfuncties
- Betalingen
- Realtime multiplayer

## 4. Functionaliteit per fase

| Fase | Inhoud |
|---|---|
| 1. Fundament | Monorepo, Docker Compose, PostgreSQL, API-skelet met OpenAPI |
| 2. Data | Importer voor Scryfall-bulkdata (kaarten, sets, prijzen) |
| 3. Web | Zoeken, kaartdetail, accounts, collectiebeheer |
| 4. App | Mobiele app met camera-toegang |
| 5. Scanner | Eerst OCR op de kaartnaam, later image matching op de exacte printing |
| 6. Extra's | Deckbouwer, prijsgeschiedenis, CSV-import/export, tweede spel |

Elke fase levert iets werkends op, zodat we steeds een resultaat hebben om te tonen.

## 5. Technische keuzes

| Onderdeel | Voorstel | Alternatieven |
|---|---|---|
| Web | Vue 3, Composition API, TypeScript, Vite, Pinia, Vue Router | n.v.t. |
| App | **[keuze]** Vue + Capacitor | React Native, Flutter, Quasar |
| API | **[keuze]** Go | NestJS (TypeScript), C# / ASP.NET Core |
| Database | PostgreSQL (later pgvector voor beeldmatching) | n.v.t. |
| Scanner | Python-microservice (OpenCV, FastAPI) | n.v.t. |
| Infra | Docker Compose, GitHub Actions | n.v.t. |
| Repo | Monorepo met pnpm workspaces | n.v.t. |

### 5.1 Keuze appframework

| Optie | Voordelen | Nadelen |
|---|---|---|
| **Vue + Capacitor** | Zelfde code als het web, één taal, makkelijkst vol te houden | Realtime beeldverwerking is zwakker dan native |
| **React Native** | Native UI, sterke camera- en realtime-ondersteuning | Ander framework dan het web, weinig codehergebruik |
| **Flutter** | Goede prestaties, uniforme UI, nieuwe taal (Dart) | Geen hergebruik met Vue, extra taal erbij |

**Voorstel:** Vue + Capacitor. Als de scanner later te traag blijkt, kan alleen het scannerscherm naar een native module worden verplaatst.

### 5.2 Keuze API-taal

| Optie | Voordelen | Nadelen |
|---|---|---|
| **Go** | Snel, eenvoudige taal, laat zien dat we verder kijken dan JavaScript | Types dubbel bijhouden (op te lossen met OpenAPI-codegeneratie) |
| **NestJS** | Types delen met de Vue-frontend, nette architectuur | Minder onderscheidend, overal dezelfde taal |
| **C# / ASP.NET Core** | Sterk getypeerd, uitstekende tooling | Wat zwaarder om mee te starten |

**Voorstel:** Go als we iets nieuws willen leren, NestJS als we snel willen vooruitgaan.

### 5.3 Proof of concept

Voor de risicovolle keuzes bouwt ieder een korte proof of concept (ongeveer een avond): camera openen, foto nemen en naar een dummy-API sturen, in de twee kandidaat-frameworks. Daarna kiezen we.

## 6. Datamodel (grove lijnen)

- **Spelonafhankelijke kern**: `users`, `collections`, `collection_entries` (hoeveelheid, staat, foil, taal) met een `game`-veld
- **Per spel een eigen kaarttabel of adapter** (`mtg_cards`, later bijvoorbeeld `pokemon_cards`), met een **importer per databron**
- Onderscheid tussen **kaart (oracle)** en **printing**: één Magic-kaart heeft vaak tientallen printings met verschillende sets en prijzen. Een collectie bestaat uit printings, niet uit kaartnamen.

## 7. De scanner

Drie aanpakken, oplopend in moeilijkheid:

1. **OCR op de kaartnaam** (Tesseract of Google ML Kit): makkelijk te starten, maar levert alleen de naam, niet de exacte printing, en is gevoelig voor licht en lettertypes.
2. **Image hashing** (OpenCV voor kaartdetectie en perspectiefcorrectie, dan pHash/dHash): goede middenweg, herkent de exacte printing.
3. **Embeddings + vectorzoeken** (bijvoorbeeld CLIP met pgvector): het robuust en indrukwekkendst, maar het moeilijkst.

**Voorstel:** eerst een app zonder scanner, dan OCR als snelle eerste versie, daarna image matching als upgrade.

## 8. Databron en juridische punten

- **Bron:** Scryfall, via de dagelijkse bulkdata, lokaal gecachet in onze eigen database.
- **Rate limits:** de zoek-endpoints zijn beperkt tot 2 requests per seconde, overige endpoints tot 10 per seconde. Bulkdata en bestanden op `*.scryfall.io` hebben geen rate limits.
- **Regels:** geen betaalmuur om Scryfall-data, geen herpublicatie of proxy van hun data, en niet impliceren dat data van een ander spel komt dan Magic. Lees de volledige voorwaarden na voor we iets publiek maken.
- **Kaartafbeeldingen** zijn eigendom van Wizards of the Coast. Kijk naar hun Fan Content Policy bij publieke release.
- **Geen Scryfall-data of kaartafbeeldingen in de repo.** Verwijs ernaar of laat de importer ze ophalen.
- **Licentie:** **[keuze]**, bijvoorbeeld **MIT** voor code en **CC BY 4.0** voor documentatie. Alternatief: AGPL als we niet willen dat iemand de app als gesloten dienst aanbiedt.

> Dit is algemene informatie en geen juridisch advies.

## 9. Risico's

| Risico | Aanpak |
|---|---|
| De scanner is moeilijker dan verwacht | Eerst een versie zonder scanner, dan OCR, dan pas image matching |
| Te veel technologieën voor drie mensen | Eén frontend-aanpak, proof of concept voor risicovolle keuzes |
| Motivatie in onze vrije tijd | Kleine mijlpalen, een werkend resultaat per fase |
| Wijzigingen of limieten bij Scryfall | Lokale kopie van de data, bulkdata gebruiken, regels naleven |

## 10. Organisatie

**[keuze]**

- Wie neemt welk onderdeel (web, API, scanner)?
- Hoe vaak overleggen we (bijvoorbeeld wekelijks)?
- Branching-strategie en codereview (bijvoorbeeld feature branches + pull requests)
- Hoe verdelen we issues en taken (GitHub Projects)?

| Onderdeel | Eigenaar |
|---|---|
| Web (Vue) | _[naam]_ |
| API + database | _[naam]_ |
| App + scanner | _[naam]_ |

## 11. Besluiten die we nodig hebben

- [ ] Appframework (Vue + Capacitor, React Native of Flutter)
- [ ] API-taal (Go of NestJS)
- [ ] Licentie
- [ ] Taakverdeling en werkafspraken

## 12. Eerstvolgende stappen

1. Propositie bespreken en de besluiten hierboven nemen
2. Repo aanmaken met licentie en README
3. Fase 1 starten: monorepo, Docker Compose en API-skelet