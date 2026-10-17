# Propositie: Card Manager

> Status: **concept**, ter bespreking
> Opgesteld door: _[naam]_
> Datum: _[datum]_

Alles wat als **[keuze]** is gemarkeerd, moeten we samen beslissen. Onderaan staat een overzicht van de besluiten die we nodig hebben. Bij elke keuze staat een voorstel, maar dat is een startpunt voor de discussie.

---

## Inhoud

1. [Idee](#1-idee)
2. [Doelen](#2-doelen)
3. [Buiten scope](#3-buiten-scope-voorlopig)
4. [Functionaliteit per fase](#4-functionaliteit-per-fase)
5. [Technische keuzes](#5-technische-keuzes)
6. [Deployment](#6-deployment)
7. [Datamodel](#7-datamodel-grove-lijnen)
8. [De scanner](#8-de-scanner)
9. [Databron en juridische punten](#9-databron-en-juridische-punten)
10. [Risico's](#10-risicos)
11. [Organisatie](#11-organisatie)
12. [Besluiten](#12-besluiten-die-we-nodig-hebben)
13. [Eerstvolgende stappen](#13-eerstvolgende-stappen)

---

## 1. Idee

Een kaartbeheerder voor verzamelkaarten, te beginnen met **Magic: The Gathering**. Gebruikers beheren hun collectie via een website en een mobiele app, en kunnen later kaarten scannen met de camera. Het project is bedoeld om te leren, samen iets af te maken en onze programmeervaardigheden te tonen.

## 2. Doelen

- Een werkende collectiebeheerder: zoeken, toevoegen, hoeveelheid, staat en foil bijhouden
- Een kaartscanner die kaarten herkent in de database
- Een architectuur die later meer spellen aankan
- Een nette, goed gedocumenteerde repo die we kunnen laten zien aan werkgevers of opleidingen
- Een echte deployment die online draait, niet alleen op onze eigen laptops

## 3. Buiten scope (voorlopig)

- Andere spellen dan Magic
- Marktplaats- of handelsfuncties
- Betalingen
- Realtime multiplayer

## 4. Functionaliteit per fase

| Fase | Inhoud |
|---|---|
| 1. Fundament | Monorepo, Docker Compose, database, API-skelet met OpenAPI, CI |
| 2. Data | Importer voor [Scryfall](https://scryfall.com/docs/api)-bulkdata (kaarten, sets, prijzen) |
| 3. Web | Zoeken, kaartdetail, accounts, collectiebeheer |
| 4. Deployment | Eerste versie online, HTTPS, automatische deploys |
| 5. App | Mobiele app met camera-toegang |
| 6. Scanner | Eerst OCR op de kaartnaam, later image matching op de exacte printing |
| 7. Extra's | Deckbouwer, prijsgeschiedenis, CSV-import/export, tweede spel |

Elke fase levert iets werkends op, zodat we steeds een resultaat hebben om te tonen.

---

## 5. Technische keuzes

### 5.0 Overzicht

| Onderdeel | Voorstel | Zie |
|---|---|---|
| Web-frontend | **[keuze]** Vue 3 + TypeScript | 5.1 |
| Mobiele app | **[keuze]** Vue + Capacitor | 5.2 |
| API-server | **[keuze]** Go (alternatieven: Laravel, zie 5.3.1; .NET, zie 5.3.2) | 5.3 |
| Database | **[keuze]** PostgreSQL | 5.4 |
| Databron (Magic) | [Scryfall API](https://scryfall.com/docs/api), via bulkdata | 9 |
| Zoeken en cache | PostgreSQL zelf, later eventueel Meilisearch of Redis | 5.5 |
| Scanner | Python-microservice (OpenCV, FastAPI) | 8 |
| Deployment | **[keuze]** VPS + Docker Compose | 6 |
| CI/CD | GitHub Actions | 6.4 |
| Architectuur en codeerafspraken | **[keuze]** modulaire monoliet + aparte scanner, DDD-lite, code first met migraties | 5.6 |

### 5.1 Web-frontend

| Optie | Voordelen | Nadelen |
|---|---|---|
| **Vue 3** (Composition API, TS, Vite, Pinia, Vue Router) | Gemakkelijk te leren, duidelijke structuur, voldoet aan onze wens (Vue + Composition API + TS), goede tooling | Kleiner ecosysteem en minder vacatures dan React |
| **React** (TS, Vite, React Router of TanStack Router, Zustand/Redux) | Grootste ecosysteem, veel vacatures, hergebruik in React Native mogelijk | Meer keuzes en losse bibliotheken te combineren, minder opinionated |
| **Svelte / SvelteKit** | Weinig boilerplate, snel, aangenaam om in te werken | Kleinere community, minder voorbeelden |
| **Angular** | Alles ingebouwd, sterk getypeerd, streng | Zwaar en meer ceremonie voor een hobbyproject |

Ook relevant:

- **Meta-framework**: Nuxt (Vue) of Next.js (React) bieden server-side rendering en SEO. Voor een app achter een login is dat niet nodig, voor publieke kaartpagina's wel interessant. Een gewone SPA met Vite is eenvoudiger.
- **UI-bibliotheek**: Tailwind (eigen componenten), PrimeVue, Vuetify of Naive UI voor Vue; shadcn/ui of MUI voor React.
- **Data ophalen**: TanStack Query bestaat voor zowel Vue als React en vermijdt veel handmatige laadlogica.
- **Met een PHP-backend**: als we voor Laravel kiezen, kunnen we Vue als losse SPA houden (aanbevolen) of Inertia.js gebruiken. Zie 5.3.1.

**Voorstel:** Vue 3 + TypeScript + Vite + Pinia, als SPA. Kies React als we de app in React Native willen bouwen en codehergebruik van logica willen.

### 5.2 Mobiele app

| Optie | Voordelen | Nadelen |
|---|---|---|
| **Vue + Capacitor** (eventueel Ionic Vue) | Zelfde code als het web, één taal, echte iOS/Android-app, toegang tot de camera | Realtime beeldverwerking zwakker dan native |
| **Quasar** | Web, PWA en mobiel in één Vue-framework, veel componenten | Eigen conventies, sterkere koppeling aan het framework |
| **PWA** (installeerbare website) | Geen app store, het lichtst om te maken | Beperkte camera en prestaties, minder "echte app" |
| **React Native** (Expo) | Native UI, sterke camera- en realtime-ondersteuning | Ander framework dan Vue, weinig codehergebruik met het web |
| **Flutter** | Uitstekende prestaties, uniforme UI, nieuwe taal (Dart) | Geen hergebruik met Vue, extra taal erbij |
| **NativeScript-Vue** | Native UI met Vue | Kleinere community, onderhoud eerst controleren |
| **Native** (Swift + Kotlin) | Beste prestaties en toegang | Twee aparte apps, veel werk voor drie mensen |
| **.NET MAUI** | Eén C#-codebase voor iOS en Android, past bij een .NET-backend | Kleinere community, geen hergebruik met onze Vue-website, tooling soms wisselvallig |

**Voorstel:** Vue + Capacitor. Als de scanner later te traag blijkt, kan alleen het scannerscherm naar een native module worden verplaatst.

### 5.3 API-server

| Optie | Voordelen | Nadelen |
|---|---|---|
| **Go** (net/http, chi, Gin, Echo of Fiber) | Snel, eenvoudige taal, één binary, past goed bij containers, onderscheidend | Types dubbel bijhouden (op te lossen met OpenAPI-codegeneratie), minder "magie" |
| **TypeScript + NestJS** | Types delen met de frontend, nette architectuur (modules, DI), snel te starten | Overal dezelfde taal, minder onderscheidend |
| **TypeScript + Fastify / Hono / Express** | Lichter en vrijer dan NestJS, veel voorbeelden | Meer zelf te structureren |
| **C# / .NET (ASP.NET Core)** | Sterk getypeerd, uitstekende tooling (Visual Studio, Rider, VS Code), zeer goede prestaties, volledig pakket (ORM, auth, background services), draait op Linux in containers, goed gevraagd in het bedrijfsleven | Wat meer opzet om mee te starten, minder gebruikelijk in hobbyprojecten, geen gedeelde types met de frontend (op te lossen met OpenAPI-codegeneratie) |
| **Kotlin + Spring Boot (of Ktor)** | Veel gebruikt in het bedrijfsleven, volwassen ecosysteem | Veel opzet en boilerplate, hoger geheugengebruik |
| **Java + Spring Boot** | Zeer veel vacatures, enorme community | Meer ceremonie dan de andere opties |
| **Rust (Axum / Actix)** | Zeer indrukwekkend, uitstekende prestaties en veiligheid | Steile leercurve, waarschijnlijk te zwaar voor een hobbyproject |
| **Python (FastAPI)** | Snel te bouwen, automatische OpenAPI-docs, past bij de scanner | Zwakker getypeerd en trager dan Go of C#; we hebben Python al voor de scanner |
| **PHP (Laravel)** | Zeer productief, veel ingebouwd (auth, ORM, queues/jobs, scheduler, Artisan-commands), grote community en veel vacatures | Minder "onderscheidend" voor ons doel, geen gedeelde types, meer onderdelen om te deployen (PHP-FPM, queue-worker, scheduler), extra taal als niemand PHP kent |
| **Elixir (Phoenix)** | Uitstekend voor realtime, mooie taal | Klein ecosysteem, sterke omschakeling voor het team |

Ook relevant:

- **API-stijl**: REST met OpenAPI (voorstel), GraphQL (flexibel voor de frontend, meer opzet) of gRPC (voor communicatie met de scanner-microservice).
- **Types delen**: genereer clients en types uit de OpenAPI-specificatie (bijvoorbeeld `openapi-typescript` voor de frontend), zodat een niet-TypeScript backend geen probleem is.
- **ORM / querybibliotheek**:
  - Go: sqlc (SQL → getypeerde code), GORM, Ent
  - TypeScript: Prisma, Drizzle, TypeORM
  - C#: Entity Framework Core
  - Python: SQLAlchemy
- **Authenticatie**: eigen implementatie (sessies of JWT), of een dienst/bibliotheek zoals Keycloak, Authentik, Auth.js, Supabase Auth of Clerk. Voor leerwaarde is een eigen eenvoudige implementatie nuttig, maar let op beveiliging (wachtwoorden hashen met argon2 of bcrypt, nooit zelf versleutelen).

**Voorstel:** Go met chi of Echo en sqlc, of NestJS als we snel willen vooruitgaan en types willen delen. **Laravel** (zie 5.3.1) is een goed alternatief als we vooral snel resultaat willen en ingebouwde queues, jobs en commands waarderen. **.NET** (zie 5.3.2) is een goede keuze als we een streng getypeerde, professionele stack willen met uitstekende tooling.

### 5.3.1 PHP / Laravel als backend

PHP draait alleen op de server, **nooit in de browser of in een Capacitor-app**. Het kan dus niet de frontend vervangen, maar wel de backend. Er zijn twee manieren om het met Vue te combineren:

| Variant | Hoe het werkt | Voordelen | Nadelen |
|---|---|---|---|
| **A. Laravel als API + losse Vue-SPA** (aanbevolen) | Laravel levert alleen een REST-API, Vue 3 + TypeScript blijft een gewone SPA | Dezelfde API voor web én app (Capacitor), Vue + Composition API + TS blijven volledig behouden | Twee projecten, types genereren via OpenAPI |
| **B. Laravel + Inertia.js + Vue** (monoliet) | Laravel regelt routes en pagina's, Vue rendert de views | Minder losse API-code voor het web, snel te ontwikkelen, Vue (Composition API + TS) blijft mogelijk | Een app in Capacitor heeft alsnog een echte API nodig; een Inertia-site in een app voelt als een website in een omhulsel en werkt slecht met camera en offline |

Blade of Livewire (PHP rendert de HTML) past niet bij ons Vue-plan en is niet bruikbaar voor een app.

**Wat we winnen met Laravel**

- **Queues en jobs**: ideaal voor de Scryfall-import, prijsupdates en het doorsturen van scanfoto's naar de Python-service. Werkt met Redis of de database als queue, inclusief retries en afhandeling van mislukte jobs.
- **Scheduler en Artisan-commands**: terugkerende taken (bijvoorbeeld "importeer elke nacht de bulkdata") schrijven we als command en plannen we met één cronregel.
- **Veel kant-en-klaar**: authenticatie (Sanctum voor API-tokens), validatie, migraties, ORM (Eloquent), mail en tests.
- **Zeer productief**: snel bij een werkende fase 1 tot 3.
- PHP en Laravel zijn bekend en gevraagd op de arbeidsmarkt.

**Wat we verliezen**

- **Minder onderscheidend**: Laravel doet veel voor ons, dus we leren minder over wat eronder gebeurt dan in Go of een lichter framework.
- **Geen gedeelde types** met de frontend. Oplossing: een OpenAPI-specificatie genereren (bijvoorbeeld met Scramble of L5-Swagger) en daar types uit maken met `openapi-typescript`.
- **Meer deploymentonderdelen**: PHP-FPM (of Laravel Octane), een queue-worker en een scheduler naast de webserver. In Docker Compose goed te regelen, maar meer containers dan één Go-binary.
- **Extra taal** als niemand van ons PHP al kent.

**Jobs, queues en commands bestaan ook in andere stacks**

| Stack | Queue/jobs | Commands/scheduler |
|---|---|---|
| Laravel | Ingebouwd (Redis of database) | Artisan + scheduler |
| NestJS | BullMQ | `@nestjs/schedule`, CLI-commands |
| Go | Asynq, River | Cobra-commands, cron |
| Python | Celery, RQ | Management-scripts, cron |
| C# / .NET | Hangfire, `BackgroundService` + Channels | Hangfire, Quartz.NET, `BackgroundService` |

Het verschil is dat Laravel het kant-en-klaar en samenhangend aanbiedt, bij de andere stacks voegen we het zelf toe.

**Voorstel bij keuze voor PHP:** variant A (Laravel als API + Vue-SPA), met Sanctum, Redis-queues en een queue-worker en scheduler als aparte containers.

### 5.3.2 C# / .NET als backend

.NET is een volwaardige keuze voor de API en past goed bij onze wens om niet te makkelijk, maar ook niet te moeilijk te kiezen. Het is sterk getypeerd (net als TypeScript, wat de overstap makkelijk maakt) en heeft veel ingebouwd.

**Onderdelen van de stack**

| Onderdeel | Opties |
|---|---|
| Framework | **ASP.NET Core Web API** met controllers, of **minimal APIs** (minder boilerplate, goed voor een kleinere API) |
| ORM / data | **Entity Framework Core** (werkt met PostgreSQL via Npgsql, ook met SQL Server of SQLite), **Dapper** (lichter, schrijf zelf SQL) |
| OpenAPI | Ingebouwde OpenAPI-ondersteuning in recente versies, of **NSwag / Swashbuckle**; NSwag kan ook een **TypeScript-client** genereren voor Vue |
| Authenticatie | **ASP.NET Core Identity**, JWT-bearer of een externe provider (Keycloak, Authentik) |
| Achtergrondtaken | `BackgroundService` / `IHostedService` (ingebouwd), **Hangfire** (jobs met dashboard), **Quartz.NET** (planning) |
| Realtime | **SignalR**, mocht dat later nodig zijn |
| Tests | xUnit of NUnit, `WebApplicationFactory` voor API-tests, Testcontainers voor een echte database |
| Lokaal ontwikkelen | Docker Compose, of **Aspire** om meerdere services en een database samen op te starten |

**Combineren met onze frontend**

- Net als bij Laravel: .NET als API + losse Vue-SPA (aanbevolen), waarbij dezelfde API ook voor de Capacitor-app werkt.
- **Blazor** (C# in de browser of op de server) vervangt Vue, maar dat past niet bij onze keuze voor Vue + TypeScript en is niet geschikt voor onze Capacitor-aanpak. Alleen interessant als we expres alles in C# willen.
- **.NET MAUI** is de .NET-optie voor een mobiele app (zie 5.2), maar met Vue + Capacitor houden we één frontend-aanpak.

**Wat we winnen met .NET**

- **Streng en veilig getypeerd**, uitstekende refactoring en tooling.
- **Zeer goede prestaties** en een kleine, snelle container op Linux; niet alleen voor Windows.
- **Veel kant-en-klaar** (DI, configuratie, logging, auth, background services), zonder het "alles in één" van Laravel.
- Veel vraag op de arbeidsmarkt, goed om te kunnen tonen.
- Het geheel is breder en professioneler dan een lichter framework, en toch minder complex dan Rust.

**Wat we verliezen**

- **Meer opzet en conventies** dan Go of Fastify; de leercurve is hoger als niemand C# kent.
- **Geen gedeelde types** met de frontend; oplossing: OpenAPI + `openapi-typescript` of NSwag.
- **Extra taal** naast TypeScript en Python.
- **Meer geheugen** dan een Go-binary, al is dat voor ons verwaarloosbaar.

**Voorstel bij keuze voor .NET:** ASP.NET Core Web API met EF Core + PostgreSQL, OpenAPI met een gegenereerde TypeScript-client, Hangfire of een `BackgroundService` voor de Scryfall-import, en een losse Vue-SPA.

### 5.4 Database

| Optie | Voordelen | Nadelen |
|---|---|---|
| **PostgreSQL** | Robuust, relationeel, uitstekende JSON-ondersteuning, full-text search, `pgvector` voor beeldmatching, grote community | Iets meer opzet dan SQLite |
| **MySQL / MariaDB** | Veel gebruikt, eenvoudig, goed ondersteund door hosts | Minder krachtige functies en extensies dan PostgreSQL |
| **SQLite** (bijvoorbeeld met Litestream of Turso) | Geen server, extreem eenvoudig, snel voor lezen | Beperkte gelijktijdige schrijftoegang, minder geschikt voor groei en meerdere servers |
| **MongoDB** | Flexibel schema, handig voor wisselende kaartdata per spel | Relaties (collectie ↔ kaart ↔ printing) zijn onnatuurlijk, minder streng |
| **Microsoft SQL Server** | Sterk met C#, goede tooling | Zwaarder, licentie en hosting minder vanzelfsprekend |
| **CockroachDB / PlanetScale / Turso** | Gedistribueerd of serverless, schaalbaar | Overkill voor ons, vaak vendor lock-in |

Hoe verhouden die zich tot onze eisen?

- Onze data is sterk **relationeel**: gebruikers, collecties, printings, sets.
- We willen **zoeken** op naam, tekst en filters.
- Later willen we **vectorzoeken** voor de scanner.

**Voorstel:** PostgreSQL. Het dekt relationele data, zoeken en vectorzoeken in één systeem.

Hosting van de database:

| Optie | Opmerking |
|---|---|
| Zelf draaien in Docker op de VPS | Goedkoop en leerzaam, wij zorgen zelf voor back-ups |
| Managed (Neon, Supabase, Railway, Fly Postgres, cloudprovider) | Minder beheer, automatische back-ups, maar kosten en beperkingen; gratis tiers veranderen vaak |

### 5.5 Zoeken, cache en vectoropslag

| Doel | Opties | Voorstel |
|---|---|---|
| Kaarten zoeken (naam, tekst, filters) | PostgreSQL full-text + `pg_trgm`, **Meilisearch**, **Typesense**, Elasticsearch/OpenSearch | Beginnen met PostgreSQL, later Meilisearch als het te traag of te beperkt is |
| Cache en sessies | **Redis** / Valkey, in-memory cache in de app (bijvoorbeeld `IMemoryCache` in .NET), geen cache | Pas toevoegen als er een echt probleem is |
| Tijdreeksen (prijsgeschiedenis, statistieken) | **InfluxDB**, **TimescaleDB** (PostgreSQL-extensie), QuestDB, gewoon een PostgreSQL-tabel | Beginnen met een gewone PostgreSQL-tabel, later TimescaleDB of InfluxDB als de datahoeveelheid groot wordt |
| Vectorzoeken voor de scanner | **pgvector**, Qdrant, Milvus, Weaviate, FAISS (in-memory) | pgvector, eenvoudig omdat het in dezelfde database zit |
| Achtergrondtaken (data-import, prijsupdates) | Cron + script, een job queue (Asynq, BullMQ, Celery, Laravel queues), de scheduler van de host | Cron of een kleine queue; bij Laravel de ingebouwde queues en scheduler |
| Bestanden (scans, uploads) | Lokale schijf, S3-compatibele opslag (Cloudflare R2, Backblaze B2, MinIO) | Lokaal in het begin |

**Over InfluxDB**

InfluxDB is een **tijdreeksdatabase** en geen cache. Het is gemaakt om grote hoeveelheden gemeten waarden met een tijdstempel op te slaan en te bevragen, zoals metrics of sensordata. Als **cache** (snelle sleutel/waarde-opslag voor veelgevraagde data of sessies) is het niet geschikt; daarvoor zijn Redis/Valkey of een cache in het geheugen de gebruikelijke keuze.

Waar InfluxDB wél interessant voor is bij ons project:

- **Prijsgeschiedenis van kaarten**: elke dag de prijs van duizenden printings bijhouden en er grafieken van maken.
- **Waarde van een collectie in de tijd**: totale waarde per gebruiker per dag.
- **Metrics van de applicatie** (verzoeken, foutpercentages, scan-succes), bijvoorbeeld samen met Grafana.

Wat het kost: een extra dienst om te draaien, te back-uppen en te leren. **Voorstel:** prijsgeschiedenis eerst in een gewone PostgreSQL-tabel (met een index op kaart + datum). Pas als die tabel echt te groot of te traag wordt, kijken we naar TimescaleDB (blijft in PostgreSQL) of InfluxDB.

### 5.6 Architectuur en codeerafspraken

We moeten vooraf afspreken **hoe** we coderen, anders schrijft ieder zijn eigen stijl en wordt de code moeilijk samen te voegen. Dit zijn de keuzes die we moeten maken. Meer structuur is niet automatisch beter: voor onze app (veel CRUD, een import, een scanner) is te veel architectuur een risico.

#### 5.6.1 Codestructuur en architectuurstijl

| Optie | Wat het inhoudt | Voordelen | Nadelen |
|---|---|---|---|
| **Framework-standaard** (bijvoorbeeld de basis-mapstructuur van Laravel, MVC) | We volgen de conventies van het framework | Snel, iedereen kent het, weinig overleg nodig | Business-logica belandt makkelijk in controllers en models, wordt groot en moeilijk testbaar |
| **Gelaagd** (controller → service → repository) | Duidelijke lagen met vaste verantwoordelijkheden | Eenvoudig te begrijpen, goed te testen | Kan veel doorgeefcode opleveren, lagen worden snel "anemisch" |
| **Vertical slices / feature-mappen** | Alles voor één feature (endpoint, logica, data) staat samen | Minder springen tussen mappen, goed voor kleine teams | Minder strikt, gedeelde logica moet bewust worden geplaatst |
| **Clean / Onion / Hexagonal** | Domein in het midden, afhankelijkheden wijzen naar binnen, infrastructuur aan de rand | Domeinlogica onafhankelijk van framework en database, goed testbaar | Meer mappen, interfaces en opzet, overkill voor simpele CRUD |
| **DDD** (Domain-Driven Design) | Modelleren rond het domein: entities, value objects, aggregates, domeinevents; en bounded contexts per deelgebied | Sterke structuur, laat vakkennis zien, goed voor complexe regels | Veel concepten en ceremonie, weinig nut waar de logica simpel is, leercurve voor het team |
| **CQRS** (opdrachten en queries gescheiden) | Aparte modellen voor schrijven en lezen | Duidelijke intentie, schaalbaar | Extra complexiteit; gebruik hooguit de lichte vorm (aparte handlers) |

**Past DDD bij ons?**

Onze app is grotendeels CRUD, dus **volledige DDD is waarschijnlijk te zwaar**. Een lichte vorm ("DDD-lite") past wel goed:

- **Bounded contexts** zijn hier natuurlijk: *Catalog* (kaarten, sets, printings, import), *Collection* (collecties en entries), *Scanning* (scanaanvragen), *Pricing* (prijzen en geschiedenis), *Identity* (gebruikers). Dat sluit aan bij onze wens om later meer spellen toe te voegen: de Catalog-context krijgt dan een adapter per spel.
- **Waar echte regels zijn** (bijvoorbeeld "een collectie-entry heeft een hoeveelheid, staat en foil-status"), gebruiken we entities en value objects. Voor simpele lijsten en zoekacties niet.
- **Spelregel:** we voegen een patroon alleen toe als het een concreet probleem oplost, en we leggen de keuze vast (zie 5.6.6).

**Voorstel per stack**

| Stack | Voorgestelde structuur |
|---|---|
| **Laravel** | De standaard mapstructuur, aangevuld met een `app/Actions` (of `Services`) voor business-logica en groepering per feature. Geen volledige DDD; controllers dun houden |
| **.NET** | Modulaire monoliet met DDD-lite en Clean Architecture per module (zie voorbeeld hieronder), of vertical slices als dat te zwaar voelt |
| **Go** | Standaardindeling met `cmd/` en `internal/`, pakketten per feature, interfaces alleen waar nodig (hexagonal-light) |
| **NestJS** | Modules per bounded context (past van nature bij het framework), services en repositories per module |

Voorbeeld van een .NET-structuur (modulaire monoliet, DDD-lite):

```
src/
  CardManager.Api/                 # ASP.NET Core, endpoints, composition root
  CardManager.Modules.Catalog/     # kaarten, sets, printings, Scryfall-import
    Domain/  Application/  Infrastructure/
  CardManager.Modules.Collection/  # collecties, entries, staat, hoeveelheid
    Domain/  Application/  Infrastructure/
  CardManager.Modules.Scanning/    # scanaanvragen, koppeling met scanner-service
  CardManager.Modules.Pricing/     # prijzen en prijsgeschiedenis
  CardManager.Modules.Identity/    # gebruikers en authenticatie
  CardManager.SharedKernel/        # gedeelde basisklassen en afspraken
tests/
```

Regels voor zo'n structuur: een module praat alleen met een andere module via een publiek contract (interface of event), nooit rechtstreeks met haar database of interne klassen.

#### 5.6.2 .NET: minimal APIs of controllers

| Optie | Voordelen | Nadelen |
|---|---|---|
| **Minimal APIs** | Weinig boilerplate, snel op te zetten, endpoints te groeperen (route groups), past goed bij vertical slices | Minder "kant-en-klaar" dan controllers, structuur moeten we zelf afspreken, kan rommelig worden zonder discipline |
| **Controllers** (gewone Web API) | Bekend patroon, veel voorbeelden, filters en attributen, vaste structuur | Meer ceremonie, de klassen kunnen groot worden |
| **Bibliotheken bovenop minimal APIs** (bijvoorbeeld FastEndpoints, Carter) | Eén klasse per endpoint, duidelijke structuur | Extra afhankelijkheid, kleinere community dan de ingebouwde opties |

Beide benaderingen zijn volwaardig en worden door ASP.NET Core ondersteund; prestatie speelt hier geen beslissende rol. Check bij het kiezen de actuele aanbevelingen in de Microsoft-documentatie.

**Voorstel:** minimal APIs, met endpoints per feature gegroepeerd in route groups (past bij modules en vertical slices). Kies controllers als we vooral een bekende, strakke structuur willen.

#### 5.6.3 Database first of code first

| Aanpak | Wat het inhoudt | Wanneer |
|---|---|---|
| **Code first** | We beschrijven het model in code (entities of models), en migraties maken het schema | Nieuw project zonder bestaande database; EF Core, Eloquent, Prisma, Drizzle |
| **Database first** | Het schema bestaat al; we genereren de modelcode daaruit | Bestaande of externe database; EF Core scaffolding |
| **SQL-first (schema als bron)** | We schrijven zelf SQL-migraties en queries, tooling genereert getypeerde code | Wanneer we de controle over SQL willen; sqlc (Go), Dapper, Flyway/goose |

Onze situatie: we beginnen **vanaf nul** en hebben geen bestaande database. Database first is daarom niet logisch. Dat laat code first (met migraties) of SQL-first over.

**Voorstel:**

- **Laravel en .NET (EF Core):** code first met migraties.
- **Go:** SQL-first: SQL-migraties als bron van waarheid, getypeerde code via sqlc.
- **De kaartdata** (Scryfall) wordt apart geïmporteerd naar eigen tabellen. We bewaren het ruwe brondata-ID (bijvoorbeeld het Scryfall-id) en maken de import herhaalbaar (upsert), zodat we de database altijd opnieuw kunnen opbouwen.

#### 5.6.4 Migraties

**Ja, we werken met migraties.** Zonder migraties kan niemand veilig het schema wijzigen, en kunnen we niet betrouwbaar deployen.

| Stack | Tool |
|---|---|
| Laravel | Ingebouwde migraties (`php artisan migrate`) |
| .NET | EF Core migrations (eventueel migration bundles voor deploys) |
| Go | goose, golang-migrate of Atlas |
| NestJS | Prisma Migrate, Drizzle Kit of TypeORM migrations |
| Stack-onafhankelijk | Flyway, Liquibase, DbUp |

**Afspraken**

- Migraties staan **in Git** en worden **gereviewd** (ook de automatisch gegenereerde).
- Een migratie die al is toegepast (op een gedeelde of productieomgeving) **wijzigen we nooit meer**; we maken een nieuwe.
- **Vooruit corrigeren** in plaats van terugdraaien in productie; we vertrouwen op back-ups en testen die ook.
- Migraties **testen in CI** tegen een lege database.
- **Uitvoeren bij deploy** als aparte stap (niet door meerdere app-instanties tegelijk), zodat er geen race ontstaat.
- **Seed-data** (testgegevens, beginwaarden) houden we gescheiden van schema-migraties.
- **Naamgeving en volgorde:** duidelijke namen, met tijdstempel of volgnummer, één migratie per wijziging.
- **Samenwerking:** bij tegelijk werken aan het schema kunnen conflicten ontstaan (bij EF Core vooral in het model-snapshotbestand). Spreek af wie een migratie maakt, en trek eerst de laatste wijzigingen binnen.
- **Lokaal** mag je de database leegmaken en opnieuw opbouwen; productie nooit.

#### 5.6.5 Microservices of alles in één

| Optie | Voordelen | Nadelen |
|---|---|---|
| **Monoliet** | Eenvoudigste ontwikkeling, deploy en debugging, transacties eenvoudig | Kan door slechte grenzen rommelig worden |
| **Modulaire monoliet** | Eén deployment, maar duidelijke modules met vaste grenzen; later op te splitsen | Vraagt discipline om de grenzen te bewaken |
| **Monoliet + enkele services** (bijvoorbeeld aparte scanner) | Services alleen waar daar een goede reden voor is | Meer onderdelen dan een zuivere monoliet |
| **Volledige microservices** | Onafhankelijk schaalbaar en deploybaar, laat architectuurkennis zien | Veel complexiteit: netwerk, consistentie, observability, versiebeheer, meer deployment; zwaar voor drie mensen in hun vrije tijd |

Een aparte scanner past wel: die gebruikt andere taal en bibliotheken (Python, OpenCV, ML), heeft een andere belasting en kan los schalen. Voor de rest zijn er weinig redenen om te splitsen.

**Voorstel:** **modulaire monoliet** voor de API en **een aparte scanner-service** (Python). De modules krijgen duidelijke grenzen, zodat we ze later kunnen afsplitsen als daar een reden voor is. Communicatie met de scanner via REST of gRPC, bij voorkeur asynchroon (via een queue) voor langere scans.

#### 5.6.6 Overige codeerafspraken

| Onderwerp | Keuze om te maken | Voorstel |
|---|---|---|
| **Formatting en linting** | ESLint + Prettier (TS), `gofmt`/`golangci-lint` (Go), `dotnet format` + `.editorconfig` (.NET), Pint (Laravel) | Automatisch formatteren, afgedwongen in CI en met pre-commit hooks (lefthook of husky) |
| **Branching** | Trunk-based of feature branches + pull requests | Feature branches met pull requests, `main` altijd werkend |
| **Commits** | Vrij of Conventional Commits | Conventional Commits (`feat:`, `fix:`, `docs:`) |
| **Code review** | Wie reviewt, hoeveel goedkeuringen | Minstens één review van een ander voor merge |
| **Tests** | Wat testen we, hoeveel | Unit-tests voor logica, integratietests voor API en database, enkele end-to-end tests; CI faalt bij falende tests |
| **API-contract** | OpenAPI als bron van waarheid | OpenAPI-specificatie, daaruit gegenereerde clients en types voor de frontend |
| **Foutformaat** | Eigen formaat of standaard | Standaard foutformaat (RFC 9457 Problem Details) |
| **API-versionering** | Geen, in het pad (`/v1/`) of in headers | `/v1/` in het pad vanaf het begin |
| **Configuratie en secrets** | `.env`, omgevingsvariabelen | Config via omgevingsvariabelen, geen secrets in Git |
| **Logging** | Gestructureerde logs (JSON) | Gestructureerd loggen, met correlatie-id per verzoek |
| **Documentatie** | README, en vastleggen van keuzes | README per onderdeel + **ADR's** |

**ADR's (Architecture Decision Records):** voor elke belangrijke keuze (framework, database, architectuurstijl, microservices of niet) schrijven we een korte notitie in `docs/adr/` met: de context, de opties, de keuze en de gevolgen. Zo kunnen we later zien waarom we iets deden, en laat het ook zien dat de keuzes bewust zijn gemaakt.

---

## 6. Deployment

### 6.1 Opties voor het hosten van backend en database

| Optie | Voordelen | Nadelen |
|---|---|---|
| **VPS + Docker Compose** (Hetzner, DigitalOcean, Scaleway, OVH, Vultr) | Goedkoop, volledige controle, we leren veel (Linux, netwerk, back-ups), past bij een portfolio | We beheren updates, beveiliging en back-ups zelf |
| **PaaS** (Fly.io, Railway, Render, Koyeb) | Snel opgezet, deploy vanuit Git, weinig beheer | Kosten schalen mee, gratis tiers wijzigen, minder leerwaarde |
| **Cloud** (AWS, Azure, Google Cloud) | Zeer uitgebreid, vaak gevraagd op de arbeidsmarkt | Complex, kostenrisico, veel om te leren voor een eerste versie |
| **Containerdiensten** (AWS ECS/Fargate, Google Cloud Run, Azure Container Apps) | Containers draaien zonder servers beheren | Meer cloud-kennis nodig, kosten bij weinig verkeer wisselend |
| **Kubernetes** (k3s of een managed cluster) | Indrukwekkend, schaalbaar | Zwaar overkill voor dit project, tenzij we het expres willen leren |
| **Homelab** (Raspberry Pi of oude pc + Cloudflare Tunnel of Tailscale) | Bijna gratis, leerzaam | Betrouwbaarheid hangt af van thuisverbinding en stroom |

### 6.2 Opties voor het hosten van de frontend

| Optie | Opmerking |
|---|---|
| **Statische hosting** (Cloudflare Pages, Netlify, Vercel, GitHub Pages) | Eenvoudig voor een Vue-SPA, automatisch HTTPS en CDN, vaak gratis tier |
| Zelfde VPS achter een reverse proxy (Caddy, Nginx, Traefik) | Alles op één plek, minder diensten om te beheren |

### 6.3 Onderdelen van een goede deployment

- **Containers**: elk onderdeel (API, scanner, database, frontend) in een eigen Docker-image, samengebracht in Docker Compose. Bij Laravel komen daar een queue-worker en een scheduler bij.
- **Reverse proxy en HTTPS**: Caddy (eenvoudigst, automatische certificaten), Traefik of Nginx
- **Domeinnaam en DNS**: bijvoorbeeld via Cloudflare
- **Secrets**: geen wachtwoorden in de repo; gebruik `.env`-bestanden buiten Git, GitHub Secrets of een secretsmanager
- **Omgevingen**: minimaal lokaal en productie, bij voorkeur ook een staging-omgeving
- **Back-ups**: geplande database-dumps (`pg_dump`) naar externe opslag, en een keer testen of herstellen werkt
- **Monitoring en logs**: eenvoudige health checks, uptime-monitoring (Uptime Kuma), logs bekijken met `docker logs` of Loki/Grafana als uitbreiding
- **Migraties**: database-migraties automatisch uitvoeren bij deploy (bijvoorbeeld goose, Flyway, Prisma Migrate)

### 6.4 CI/CD

| Onderdeel | Opties | Voorstel |
|---|---|---|
| Pijplijn | **GitHub Actions**, GitLab CI | GitHub Actions |
| Container registry | **GitHub Container Registry (GHCR)**, Docker Hub | GHCR |
| Deploy | SSH + `docker compose pull && up -d`, Watchtower, Coolify, Dokploy, GitOps (Argo CD) | SSH of een zelf-gehoste tool als Coolify |
| Stappen | Lint → test → build → image publiceren → deployen | Idem, deployen alleen vanaf `main` |

### 6.5 De mobiele app verspreiden

| Optie | Opmerking |
|---|---|
| Testbuilds via TestFlight (iOS) en interne testtracks in Google Play (Android) | Voor vrienden en testers; vereist ontwikkelaarsaccounts |
| Publiceren in de App Store en Google Play | Beide vragen een betaald ontwikkelaarsaccount (bedrag actueel controleren) en een reviewproces |
| Rechtstreeks een Android-APK delen | Gratis en snel voor testers, niet bedoeld voor breed publiek |
| PWA | Geen store nodig, wel beperkingen |

### 6.6 Voorstel

- **Fase 1-4:** een kleine VPS (bijvoorbeeld bij Hetzner of een vergelijkbare aanbieder) met **Docker Compose** en **Caddy**, database in Docker met geplande back-ups. De frontend op Cloudflare Pages of dezelfde VPS.
- **Deploys:** GitHub Actions bouwt images naar GHCR, de VPS haalt ze op.
- **Later:** zo nodig overstappen op managed database of containerdienst als het gebruik groeit.

> De prijzen en gratis tiers van hostingdiensten veranderen regelmatig. Vergelijk de actuele voorwaarden voor we kiezen.

---

## 7. Datamodel (grove lijnen)

- **Spelonafhankelijke kern**: `users`, `collections`, `collection_entries` (hoeveelheid, staat, foil, taal) met een `game`-veld
- **Per spel een eigen kaarttabel of adapter** (`mtg_cards`, later bijvoorbeeld `pokemon_cards`), met een **importer per databron**
- Onderscheid tussen **kaart (oracle)** en **printing**: één Magic-kaart heeft vaak tientallen printings met verschillende sets en prijzen. Een collectie bestaat uit printings, niet uit kaartnamen.

## 8. De scanner

Drie aanpakken, oplopend in moeilijkheid:

1. **OCR op de kaartnaam** (Tesseract of Google ML Kit): makkelijk te starten, maar levert alleen de naam, niet de exacte printing, en is gevoelig voor licht en lettertypes.
2. **Image hashing** (OpenCV voor kaartdetectie en perspectiefcorrectie, dan pHash/dHash): goede middenweg, herkent de exacte printing.
3. **Embeddings + vectorzoeken** (bijvoorbeeld CLIP met pgvector): het robuustst en indrukwekkendst, maar het moeilijkst.

Waar draait de scanner?

| Optie | Voordelen | Nadelen |
|---|---|---|
| **Op de server** (Python-microservice) | Eén plek om te verbeteren, de app blijft licht | Foto's uploaden, vertraging, serverkosten |
| **Op het toestel** (ML Kit, TensorFlow Lite, OpenCV mobile) | Snel en werkt offline, live scannen mogelijk | Moeilijker te bouwen en per platform te onderhouden |
| **Hybride** | Snelle herkenning op het toestel, verfijnen op de server | Meeste werk |

**Voorstel:** eerst een app zonder scanner, dan OCR als snelle eerste versie, daarna image matching op de server als upgrade.

## 9. Databron en juridische punten

### 9.1 Waarom Scryfall

[Scryfall](https://scryfall.com/docs/api) biedt een REST-achtige API waarmee je de kaartdata van de gewone site in een makkelijk te verwerken formaat kunt ophalen. De API staat op `https://api.scryfall.com`.

Binnen het [Fan Content Policy van Wizards of the Coast](https://company.wizards.com/fancontentpolicy) stelt Scryfall zijn kaartdata en afbeeldingendatabase **gratis** beschikbaar, vooral om extra Magic-software te maken, onderzoek te doen of community-content te maken. Een kaartbeheerder voor Magic past daar goed in.

Ook voor ons nuttig:

- Bulkdata om de hele database lokaal te laden, zonder rate limits
- Kaarten, sets, prijzen, rulings en afbeeldingen op één plek
- Geen API-sleutel of betaald abonnement nodig

### 9.2 Technische eisen voor onze requests

- Alleen **HTTPS** (TLS 1.2 of hoger); gewone HTTP wordt niet geaccepteerd.
- Alle responses zijn **UTF-8**, met ook niet-ASCII-tekens in veel velden. Onze database en code moeten daar goed mee omgaan.
- Elke request naar `api.scryfall.com` moet een **`User-Agent`**- en een **`Accept`**-header bevatten.
  - De `User-Agent` moet **kloppen met ons gebruik** en de naam van onze applicatie bevatten, bijvoorbeeld `CardManager/1.0`. We mogen de HTTP-bibliotheek de header niet zelf laten kiezen.
  - Voor de `Accept`-header is een algemene voorkeur genoeg, zoals `application/json;q=0.9,*/*;q=0.8`.
  - Doen we requests vanuit JavaScript in de browser, dan laten we de `User-Agent` van de browser ongemoeid.
- **Rate limits** (zie de [documentatie over rate limits](https://scryfall.com/docs/api/rate-limits)): de zoek-endpoints zijn beperkt tot 2 requests per seconde, overige endpoints tot 10 per seconde. Bulkdata en bestanden op `*.scryfall.io` hebben geen rate limits.
- **Praktisch gevolg:** onze importer (en later de scanner) leest de **bulkdata**, en we doen geen live requests per gebruiker op de zoekendpoints.

### 9.3 Regels voor het gebruik van de data

Volgens de documentatie moeten we deze regels volgen:

| Regel | Wat het voor ons betekent |
|---|---|
| Geen Scryfall-logo's en de naam Scryfall niet gebruiken op een manier die suggereert dat zij ons of ons product steunen | Geen Scryfall-logo in de app; bronvermelding formuleren zonder "officieel" of "in samenwerking met" |
| Geen **betaalmuur** om de Scryfall-data: geen betalingen, enquêtes, abonnementen, beoordelingen of verplichte chatservers/kanalen als voorwaarde | Heeft de app accounts, dan moet iedereen de kaartdata anoniem of met een gratis account kunnen gebruiken. Betaalde extra's mogen alleen niet de toegang tot de Scryfall-data zelf blokkeren |
| Geen **nieuwe spellen** maken met Scryfall-data en niet laten lijken dat de data van een ander spel komt dan Magic | Scryfall alleen voor onze Magic-tabellen; voor andere spellen zijn aparte databronnen nodig (zie datamodel in sectie 7) |
| Niet simpelweg **herverpakken, herpubliceren of proxyen**; onze software moet extra waarde bieden | Onze API mag geen doorgeefluik zijn voor Scryfall-data. Collectiebeheer, scanner, prijsgeschiedenis en deckbouwer zijn onze toegevoegde waarde |

### 9.4 Regels voor kaartafbeeldingen

Wanneer we afbeeldingen van Scryfall tonen:

- Het **copyright en de naam van de artiest** op de kaart mogen niet worden afgedekt, bijgesneden of weggelaten.
- **Niet vervormen, scheeftrekken of uitrekken.**
- **Niet vervagen, verscherpen, ontkleuren of kleurverschuiven.**
- **Geen eigen watermerken, stempels of logo's** op kaartafbeeldingen.
- De afbeeldingen **niet zo plaatsen** dat het lijkt alsof iemand anders dan Wizards of the Coast de kaart maakte, of dat de kaart uit een ander spel komt.
- Bij gebruik van de **`art_crop`** moeten we de artiest en het copyright elders in dezelfde interface vermelden, of de volledige kaartafbeelding elders in die interface tonen.

Let op bij de scanner: deze regels gaan over het **tonen** van afbeeldingen. Als we afbeeldingen intern bewerken (bijvoorbeeld schalen of grijswaarden voor image matching), tonen we die bewerkte versies niet aan gebruikers. Twijfel je, controleer dan de documentatie of vraag het na.

Bij herhaaldelijk verkeerd of misleidend gebruik van data of afbeeldingen kan Scryfall onze toegang beperken of blokkeren.

### 9.5 Overige punten

- **Kaartafbeeldingen** blijven eigendom van Wizards of the Coast. Lees ook het [Fan Content Policy](https://company.wizards.com/fancontentpolicy) bij een publieke release.
- **Geen Scryfall-data of kaartafbeeldingen in de repo.** Verwijs ernaar of laat de importer ze ophalen.
- Lees de volledige [API-documentatie](https://scryfall.com/docs/api) opnieuw voor we iets publiek maken; regels kunnen veranderen.

### 9.6 Licentie

- **Licentie:** **[keuze]**

| Optie | Opmerking |
|---|---|
| **MIT** | Heel permissief, eenvoudig, populair in portfolio's |
| **Apache 2.0** | Zoals MIT, met een patentclausule |
| **GPL / AGPL** | Copyleft; AGPL geldt ook voor webdiensten |
| **CC BY 4.0** | Voor documentatie en onderzoek |

**Voorstel:** MIT voor code, CC BY 4.0 voor documentatie.

> Dit is algemene informatie en geen juridisch advies.

## 10. Risico's

| Risico | Aanpak |
|---|---|
| De scanner is moeilijker dan verwacht | Eerst een versie zonder scanner, dan OCR, dan pas image matching |
| Te veel technologieën voor drie mensen | Eén frontend-aanpak, proof of concept voor risicovolle keuzes |
| Motivatie in onze vrije tijd | Kleine mijlpalen, een werkend resultaat per fase |
| Wijzigingen of limieten bij Scryfall | Lokale kopie van de data, bulkdata gebruiken, regels naleven |
| Hostingkosten lopen op | Klein beginnen (VPS), kosten bijhouden, geen dure cloudservices zonder reden |
| Dataverlies of een beveiligingslek | Automatische back-ups, wachtwoorden correct hashen, updates bijhouden, geen secrets in Git |
| Een van ons haakt af | Documentatie bijhouden, geen kennis bij één persoon laten |
| Te veel architectuur voor een eenvoudige app (DDD, microservices) | DDD-lite en een modulaire monoliet, alleen extra patronen als ze een concreet probleem oplossen |
| Verschillende codeerstijlen of conflicterende migraties | Afspraken in 5.6, formatting automatisch afdwingen, vaste werkwijze voor migraties |

## 11. Organisatie

**[keuze]**

- Wie neemt welk onderdeel (web, API, scanner, deployment)?
- Hoe vaak overleggen we (bijvoorbeeld wekelijks)?
- Branching-strategie en codereview (bijvoorbeeld feature branches + pull requests)
- Hoe verdelen we issues en taken (GitHub Projects)?

| Onderdeel | Eigenaar |
|---|---|
| Web (frontend) | _[naam]_ |
| API + database | _[naam]_ |
| App + scanner | _[naam]_ |
| Deployment + CI/CD | _[naam]_ |

## 12. Besluiten die we nodig hebben

- [ ] Webframework (Vue, React of anders)
- [ ] Appframework (Vue + Capacitor, React Native, Flutter of anders)
- [ ] API-taal en framework (Go, NestJS, C# / .NET, Kotlin, Rust, Python, PHP/Laravel of anders)
- [ ] Als Laravel: losse Vue-SPA (variant A) of Inertia (variant B)
- [ ] Architectuurstijl per backend (framework-standaard, gelaagd, vertical slices, Clean, DDD of DDD-lite)
- [ ] Als .NET: minimal APIs of controllers
- [ ] Database first, code first of SQL-first, en werken met migraties (en de afspraken daarover)
- [ ] Monoliet, modulaire monoliet of microservices (en welke aparte services, bijvoorbeeld de scanner)
- [ ] Codeerafspraken: formatting, branching, commits, review, tests, foutformaat en ADR's
- [ ] Database (PostgreSQL of anders) en waar die draait
- [ ] Extra diensten: cache (Redis?), zoeken (Meilisearch?), tijdreeksen voor prijsgeschiedenis (PostgreSQL, TimescaleDB of InfluxDB?)
- [ ] Hosting (VPS, PaaS, cloud of homelab)
- [ ] Deploymentmethode en CI/CD (GitHub Actions + Docker Compose?)
- [ ] Licentie
- [ ] Taakverdeling en werkafspraken

### Als we niet kunnen kiezen: de standaardstack

| Onderdeel | Keuze |
|---|---|
| Web | Vue 3 + TypeScript + Vite + Pinia |
| App | Capacitor |
| API | Go + chi + sqlc, met OpenAPI |
| Database | PostgreSQL (+ pgvector later) |
| Scanner | Python + OpenCV + FastAPI |
| Hosting | Eén VPS + Docker Compose + Caddy |
| CI/CD | GitHub Actions → GHCR → SSH-deploy |
| Licentie | MIT (code), CC BY 4.0 (docs) |

## 13. Eerstvolgende stappen

1. Propositie bespreken, de besluiten hierboven nemen en ze vastleggen als korte ADR's (zie 5.6.6)
2. Eventueel een proof of concept per risicovolle keuze (bijvoorbeeld camera in Capacitor vs. React Native/Flutter)
3. Repo aanmaken met licentie en README
4. Fase 1 starten: monorepo, Docker Compose, API-skelet en CI