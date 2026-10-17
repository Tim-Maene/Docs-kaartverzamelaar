# Propositie: Card Manager

> Status: **concept**, ter bespreking
> Opgesteld door: Tim Maene
> Datum: 17/10/2026

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
| 2. Data | Importer voor Scryfall-bulkdata (kaarten, sets, prijzen) |
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
| Zoeken en cache | PostgreSQL zelf, later eventueel Meilisearch of Redis | 5.5 |
| Scanner | Python-microservice (OpenCV, FastAPI) | 8 |
| Deployment | **[keuze]** VPS + Docker Compose | 6 |
| CI/CD | GitHub Actions | 6.4 |

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

- **Bron:** Scryfall, via de dagelijkse bulkdata, lokaal gecachet in onze eigen database.
- **Rate limits:** de zoek-endpoints zijn beperkt tot 2 requests per seconde, overige endpoints tot 10 per seconde. Bulkdata en bestanden op `*.scryfall.io` hebben geen rate limits.
- **Regels:** geen betaalmuur om Scryfall-data, geen herpublicatie of proxy van hun data, en niet impliceren dat data van een ander spel komt dan Magic. Lees de volledige voorwaarden na voor we iets publiek maken.
- **Kaartafbeeldingen** zijn eigendom van Wizards of the Coast. Kijk naar hun Fan Content Policy bij publieke release.
- **Geen Scryfall-data of kaartafbeeldingen in de repo.** Verwijs ernaar of laat de importer ze ophalen.
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

1. Propositie bespreken en de besluiten hierboven nemen
2. Eventueel een proof of concept per risicovolle keuze (bijvoorbeeld camera in Capacitor vs. React Native/Flutter)
3. Repo aanmaken met licentie en README
4. Fase 1 starten: monorepo, Docker Compose, API-skelet en CI