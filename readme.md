# Emberpath

Emberpath samler vektlogging, trening og kosthold. Første funksjon er daglig
vektlogging med oppretting, historikk, redigering og sletting.

Dette repoet eier felles dokumentasjon og Docker Compose. Hvert app-repo eier
sin egen Dockerfile:

```text
Repos/
  Emberpath/                 # compose.yaml og dokumentasjon
  Emberpath-weight-service/  # FastAPI, PostgreSQL-modell og migreringer
  Emberpath-web/             # React og API-klient
```

Se [migreringsnotatet](docs/weight-service-template-migration.md) for
kontrollpunkter og lokal sikkerhetskopi ved overgang til FastAPI-malen.

## Start hele appen med Docker

Konfigurer først Clerk som beskrevet i [autentiseringsoppsettet](docs/authentication.md).
Installer Docker med Compose, start Docker-motoren og plasser repoene som
søskenmapper som vist over. Stopp eventuell Vite-server med `Ctrl+C` først;
Vite og Compose-web bruker begge port 5173. Kjør fra `Emberpath`:

```powershell
docker compose up --build -d --wait
```

- Web: <http://localhost:5173/weight>
- API-dokumentasjon: <http://localhost:8000/docs>
- Tjenestemetadata: <http://localhost:8000/>
- Helsesjekk: <http://localhost:8000/healthz>
- Databaseberedskap: <http://localhost:8000/readyz>

Compose starter PostgreSQL, kjører Alembic-migreringer og starter deretter
backend og web. Web serveres av Nginx, som videresender `/api` til backend.
Backend krever PostgreSQL (`DATABASE_REQUIRED=true`). Compose bruker `/readyz`
før web starter; `/healthz` og `/` fungerer uten databaseforbindelse.
Web lytter på alle nettverksgrensesnitt på port 5173. API-et på port 8000 og
PostgreSQL er kun tilgjengelige direkte fra denne maskinen. Oppsettet er for
lokal utvikling med Clerk-autentisering. Hubrepoet kjører ingen egen webserver;
en eventuell dokumentasjonsserver kan ikke bruke port 8000 samtidig med API-et.

Sett Clerk-verdiene i `.env`; behold eventuelle eksisterende databaseverdier.
Ved førstegangsoppsett kan `.env.example` kopieres til `.env`. Bruk et URL-sikkert
passord. PostgreSQL setter passordet ved første opprettelse av datavolumet;
endring av `.env` endrer ikke passordet i en eksisterende database.

```powershell
docker compose ps
docker compose logs --tail=100 weight-service
docker compose down
```

Vektloggene lagres i volumet `postgres-data` og bevares når containerne
stoppes eller bygges på nytt. `docker compose down -v` sletter også lagrede data.

## Lokal utvikling med automatisk omlasting

Kjør PostgreSQL og backend med Compose, og web med Vite. Fra dette repoet:

```powershell
docker compose stop web
docker compose up --build -d --wait weight-service
```

Dette starter også databasen og kjører migreringene. Start web i en annen terminal:

```powershell
cd ..\Emberpath-web
npm ci
npm run dev
```

Åpne <http://localhost:5173/weight>. Vite lytter på `0.0.0.0:5173` og
videresender `/api` til `127.0.0.1:8000`. Den avslutter hvis port 5173 er
opptatt, så stopp prosessen som bruker porten før oppstart.

For automatisk omlasting av backend også, stopp Compose-backend og start
Uvicorn lokalt:

```powershell
docker compose stop weight-service
cd ..\Emberpath-weight-service
uv sync --locked
Copy-Item .env.example .env
uv run alembic upgrade head
uv run uvicorn src.main:app --reload --port 8000
```

Kopier backendens `.env` kun ved førstegangsoppsett; behold eksisterende
konfigurasjon ved senere oppstart. Sett `DATABASE_URL` i denne filen før
migrering hvis databaseport eller passord er endret i hubens `.env`.
For eksempel brukes `postgresql+psycopg://emberpath:emberpath_local@127.0.0.1:55432/emberpath`
dersom `POSTGRES_PORT=55432`.

## Test fra Wi-Fi

Både Vite og Compose-web kan nås fra en telefon på samme lokale nettverk.
Finn PC-ens lokale IPv4-adresse med `ipconfig`, og åpne
`http://<LAN_IP>:5173/weight` på telefonen. `localhost` på telefonen peker på
telefonen selv. API-kall går gjennom webserverens `/api`-proxy; backend og
database trenger ikke eksponeres på nettverket.

PC-en må være på, nettverket må tillate trafikk mellom enhetene, og Windows-
brannmuren må tillate innkommende TCP-trafikk på port 5173 for det aktuelle
nettverket. Legg telefonens faktiske webadresse til `CLERK_AUTHORIZED_PARTIES`.
Clerk kan kreve HTTPS for innlogging fra andre verter enn localhost; se
[autentiseringsoppsettet](docs/authentication.md). Bruk et betrodd nettverk.

## Tester og kodekontroll

Integrasjonstestene bruker en separat, midlertidig PostgreSQL-instans på
port 5433. De skal aldri kjøres mot databasen med egne vektlogger.

```powershell
docker compose --profile test up -d --wait postgres-test
cd ..\Emberpath-weight-service
$env:TEST_DATABASE_URL = "postgresql+psycopg://emberpath:emberpath_test@127.0.0.1:5433/emberpath_test"
uv run pytest
uv run ruff check .
uv run ruff format --check .
uv run pyright
```

Testdatabasen har navnet `emberpath_test`, brukeren `emberpath` og det lokale
testpassordet `emberpath_test`. Backendens README beskriver `TEST_DATABASE_URL`.
Testdatabasen bruker minnelagring og mister innholdet når den stoppes.

Kjør frontendkontrollene fra `Emberpath-web`:

```powershell
npm test
npm run lint
npm run build
```

Stopp testdatabasen fra `Emberpath` med:

```powershell
docker compose --profile test stop postgres-test
```

## API

Se [import og eksport](docs/weight-data-transfer.md) for datafiler med komma, semikolon eller tabulator,
konvertering fra pounds til kilogram og forhåndsvisning før import.

En vektlogg har `id`, `date` (YYYY-MM-DD) og `weight_kg` (tall i kg, maks to
desimaler). Én logg tillates per bruker og dato. Alle datoer er kalenderdatoer uten
tidssone; skjemaet foreslår dagens lokale dato.

| Metode | Endepunkt | Handling |
|---|---|---|
| POST | `/weight-logs` | Opprett logg |
| GET | `/weight-logs` | Hent historikk, nyeste dato først |
| GET | `/weight-logs/summary` | Oppsummer valgt periode for innlogget bruker |
| GET | `/weight-logs/rolling-average` | Hent 7-, 14- eller 30-dagers glidende gjennomsnitt |
| GET | `/weight-logs/{id}` | Hent én logg |
| PATCH | `/weight-logs/{id}` | Endre dato eller vekt |
| DELETE | `/weight-logs/{id}` | Slett logg |
| GET | `/weight-goals/active` | Hent aktivt vektmål, eller `null` |
| PUT | `/weight-goals/active` | Opprett eller erstatt aktivt vektmål |
| PATCH | `/weight-goals/{id}` | Marker aktivt mål som fullført eller avbrutt |

Alle vektruter krever et gyldig Clerk-sessiontoken som `Authorization: Bearer`.
Duplikatdato gir 409, manglende eller andre brukeres logg gir 404 og ugyldig input gir 422.
`GET /healthz` sjekker at API-et kjører og er uavhengig av databasen.

Oppsummeringen tar valgfrie `start_date` og `end_date` i formatet YYYY-MM-DD,
inkludert begge grensene. Uten grenser brukes hele historikken. Responsen gir
antall registreringer, gjennomsnittsvekt, første og siste registrering, samt
endring i kg og prosent fra første til siste registrering. Dager uten målinger
teller ikke i gjennomsnittet. Tall avrundes til to desimaler; endring krever minst
to målinger. Tomme perioder gir antall 0 og `null` for de andre verdiene.
Kortene over grafen følger samme periodefilter som grafen og historikken, og
oppdateres når en registrering opprettes, endres eller slettes.

Grafens glidende gjennomsnitt kan settes til 7, 14 eller 30 dager med
«Rolling average». Standard er 7 dager. Backend tar parameteren
`window_days=7|14|30`; andre verdier gir 422. Vinduet inkluderer måledatoen og
de foregående `window_days - 1` kalenderdagene. Kun registrerte målinger telles;
delvise vinduer er tillatt og manglende dager telles ikke som null.
Valgfrie `start_date` og `end_date` begrenser returnerte punkter, men målinger
før perioden inngår fortsatt i beregningen. Responsen gir valgt `window_days`
og `points` med `date`, `mean_weight_kg` og `measurement_count`.
Alle beregninger gjelder innlogget bruker. Frontend beholder valgt vindu når
periodefilteret endres, og cacher vinduene separat.
## Personlige vektmål

Weight-service eier vektmålene. Hver bruker kan ha ett aktivt mål med
`target_weight_kg`, `start_date` og valgfri `target_date`. For nye mål med måldato
må datoen være etter startdatoen. Når et mål erstattes, beholdes det gamle med status `replaced`.
Et aktivt mål kan avsluttes eksplisitt med status `completed` eller `cancelled`;
en måling som passerer målvekten fullfører ikke målet automatisk.

Webappen viser det aktive målet med en stiplet, lavendelfarget linje i grafen.
Uten måldato er linjen flat. Med måldato og lagret startvekt viser den en planlagt
utvikling fra startvekten til målvekten. Grafen viser maksimalt én kalendermåned
fremover, med uendret helning og faktisk måldato. «Show goal» skjuler eller viser
mållinjen; målinger og oppsummeringer følger fortsatt det valgte periodefilteret.
Linjen viser dagens aktive plan, ikke en prognose eller en historisk målkurve.

`baseline_weight_kg` lagres på målet. Ved oppretting kan startvekten oppgis
manuelt; ellers brukes siste registrering på eller før startdatoen. Et datert
mål uten en slik registrering krever at startvekten oppgis. Senere målinger
endrer ikke den lagrede startvekten. Eksisterende mål uten startvekt beholder
flat linje til målet oppdateres med en startvekt.
Ved endring av et mål med samme startdato beholdes lagret startvekt dersom en
ny verdi ikke oppgis.

API-responsens `plan` gir antall dager, total endring og planlagt endring per
uke og fjortendagersperiode. Ukentlig endring beregnes som
`(målvekt - startvekt) * 7 / antall dager`, avrundet til to desimaler.
Negative tall betyr planlagt vektnedgang, positive tall planlagt oppgang.
Uten gyldig datoperiode og startvekt er `plan` lik `null`. Dette er beregninger
av brukerens valgte plan, ikke en anbefalt endringstakt.

Andre tjenester skal bruke API-et for å lese mål, ikke lese eller endre tabellen
direkte. Alle målruter krever samme autentisering og brukerisolasjon som vektlogger.
