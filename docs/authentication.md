# Authentication

Clerk handles registration, sign-in and sessions. The web app sends its session
token to the weight service with each request. The service validates the token
and restricts measurements to an internal Emberpath user UUID. External identities
are mapped by issuer and subject; email addresses are not ownership keys.

## Configuration

Add these public configuration values to the hub's ignored `.env`, preserving
existing database settings:

```dotenv
VITE_CLERK_PUBLISHABLE_KEY=pk_test_your_development_publishable_key
CLERK_ISSUER=https://your-instance.clerk.accounts.dev
CLERK_AUTHORIZED_PARTIES=http://localhost:5173,http://127.0.0.1:5173
```

Use the matching Clerk development instance for both the publishable key and
issuer. No Clerk secret key is needed by the browser or weight service. Never
prefix a secret key with `VITE_` or pass it as a Docker build argument.

For Vite development, put `VITE_CLERK_PUBLISHABLE_KEY` in the web repo's ignored
`.env.local`. For Compose, the hub passes that public value as a web build
argument; rebuild the web image after changing it.

```powershell
docker compose up --build -d --wait
```

Protected routes fail closed without authentication configuration. Metadata,
health and readiness remain public. Frontend visibility controls do not replace
backend access checks.

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
port. Add the chosen LAN origin explicitly; do not use a wildcard. The frontend
continues to proxy `/api`, so PostgreSQL and port 8000 can remain loopback-only.
Clerk browser authentication may require a secure context on non-localhost hosts;
use a trusted HTTPS development setup when plain LAN HTTP is unsupported. This
change does not configure a tunnel, certificates or public exposure.

## Provider boundaries

Each service owns authorization for its data. No additional authentication
microservice is required. Internal UUIDs keep measurement ownership independent
of Clerk; a later provider change requires explicit identity mapping and may
require users to sign in or enrol authentication factors again.
