# Authentication

Clerk handles registration, sign-in and sessions. The web app sends its session token to the weight or nutrition service for protected requests. Each service verifies the token and maps its own records to a service-owned user ID using the verified issuer and subject; email addresses are not ownership keys. Both services admit valid sessions from the configured Clerk instance and authorized frontend origins while isolating each user's records. Hiding a page is not an access control.

## Configuration

Add these public configuration values to the hub's ignored `.env`, preserving
existing database settings:

```dotenv
VITE_CLERK_PUBLISHABLE_KEY=pk_test_your_development_publishable_key
CLERK_ISSUER=https://your-instance.clerk.accounts.dev
CLERK_AUTHORIZED_PARTIES=http://localhost:5173,http://127.0.0.1:5173
```

Use the matching Clerk development instance for the publishable key and both
services' issuer. There is no Nutrition-specific owner-ID setting or plan-write
switch. No Clerk secret key is needed by the browser or either service. Never
prefix a secret key with `VITE_` or pass it as a Docker build argument.

For Vite development, put `VITE_CLERK_PUBLISHABLE_KEY` in the web repo's ignored
`.env.local`. For Compose, the hub passes that public value as a web build
argument; rebuild the web image after changing it.

```powershell
docker compose up --build -d --wait
```

Protected routes fail closed without authentication configuration. Metadata,
health and readiness remain public. Nutrition plan writes require a valid
session and available migrated storage; previews do not save a plan.
Removing the temporary write gate does not imply that a calculated or saved
plan is individually safe or that the PoC is approved for public release.

Local development and staging are environment labels, not account admission
controls. If a deployment should admit only selected people, restrict access
at a verified deployment or Clerk boundary. A valid second account can use
its own Nutrition plans but cannot access the first account's records.

For standalone Uvicorn runs, use each service's ignored `.env`, matching the
hub's Clerk issuer and authorized origins. Compose reads the hub's `.env`, not
the service files. Remove retired `CLERK_OWNER_SUBJECT` and
`PLAN_WRITES_ENABLED` keys from a Nutrition service `.env` before upgrading;
unsupported settings are rejected rather than ignored.

## Existing measurements

The ownership migration retains existing measurements as unclaimed rows. They
are hidden from all users until an operator explicitly assigns them to a verified
account using the weight service's administrative claim command. Registering the
first account never automatically claims the history. See the weight service
README for the command and verification procedure.

After signing in and opening the weight page once, confirm the exact Clerk user
ID in the Dashboard. From the hub, preview and then apply the assignment:

```powershell
docker compose exec weight-service python -m src.claim_legacy --subject user_your_verified_id
docker compose exec weight-service python -m src.claim_legacy --subject user_your_verified_id --apply
```

The account must already have an identity mapping created by an authenticated
request. The command only assigns unclaimed measurements, never transfers another
user's history, and rolls back if dates conflict with that account's own entries.
Reload the journal after assignment. Avoid adding measurements before claiming
the existing history to prevent those date conflicts.

## Local network testing

Authorized parties must contain exact frontend origins, including scheme and
port. Add the chosen LAN origin explicitly for both services; do not use a
wildcard. The frontend continues to proxy `/api`, so both PostgreSQL databases
and API ports can remain loopback-only. Their default host ports are 8000
for Weight and 8001 for Nutrition, configurable through `WEIGHT_API_PORT`
and `NUTRITION_API_PORT` in the hub `.env`.
Clerk browser authentication may require a secure context on non-localhost hosts;
use a trusted HTTPS development setup when plain LAN HTTP is unsupported. This
change does not configure a tunnel, certificates or public exposure.

The local Compose setup uses a development Host-header wildcard in both APIs.
Keep their direct ports bound to loopback and expose only the web proxy on the
LAN. Before changing that boundary, harden Host handling for both services
together without breaking the documented LAN workflow; production settings
already require an explicit allowed-host list.

## Provider boundaries

Each service owns authorization for its data. No additional authentication
microservice is required. Separate internal UUIDs keep measurement and nutrition
plan ownership independent of Clerk; a later provider change requires explicit
identity mapping and may require users to sign in or enrol authentication factors
again. A Nutrition identity or plan never authorizes access to another user's
weight data.
