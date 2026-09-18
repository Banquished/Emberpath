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

## Start hele appen med Docker

Installer Docker med Compose, start Docker-motoren og plasser repoene som
søskenmapper som vist over. Stopp eventuell Vite-server med `Ctrl+C` først;
Vite og Compose-web bruker begge port 5173. Kjør fra `Emberpath`:

```powershell
docker compose up --build -d --wait
```

- Web: <http://localhost:5173/weight>
- API-dokumentasjon: <http://localhost:8000/docs>
- Helsesjekk: <http://localhost:8000/healthz>

Compose starter PostgreSQL, kjører Alembic-migreringer og starter deretter
backend og web. Web serveres av Nginx, som videresender `/api` til backend.
Web lytter på alle nettverksgrensesnitt på port 5173. API-et på port 8000 og
PostgreSQL er kun tilgjengelige direkte fra denne maskinen. Oppsettet er for
lokal utvikling uten autentisering. Hubrepoet kjører ingen egen webserver;
en eventuell dokumentasjonsserver kan ikke bruke port 8000 samtidig med API-et.

Standardverdiene virker uten `.env`. Kopier `.env.example` til `.env` hvis du
vil endre porter eller det lokale databasepassordet. Bruk et URL-sikkert
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
nettverket. Bruk kun et betrodd lokalt nettverk til denne appen uten innlogging.

## Tester og kodekontroll

Integrasjonstestene bruker en separat, midlertidig PostgreSQL-instans på
port 5433. De skal aldri kjøres mot databasen med egne vektlogger.

```powershell
docker compose --profile test up -d --wait postgres-test
cd ..\Emberpath-weight-service
uv run pytest
uv run ruff check .
uv run ruff format --check .
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

En vektlogg har `id`, `date` (YYYY-MM-DD) og `weight_kg` (tall i kg, maks to
desimaler). Én logg tillates per dato. Alle datoer er kalenderdatoer uten
tidssone; skjemaet foreslår dagens lokale dato.

| Metode | Endepunkt | Handling |
|---|---|---|
| POST | `/weight-logs` | Opprett logg |
| GET | `/weight-logs` | Hent historikk, nyeste dato først |
| GET | `/weight-logs/{id}` | Hent én logg |
| PATCH | `/weight-logs/{id}` | Endre dato eller vekt |
| DELETE | `/weight-logs/{id}` | Slett logg |

Duplikatdato gir 409, manglende logg gir 404 og ugyldig input gir 422.
`GET /healthz` sjekker at API-et kjører og er uavhengig av databasen.
