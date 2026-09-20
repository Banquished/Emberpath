# Weight service template migration

## Scope

Adopt the FastAPI-service operational foundation and asynchronous database
access without changing measurement storage or the existing frontend contract.
Authentication, user ownership and analytics remain separate future changes.

- Weight service baseline: `a1b414a791440b225625c838fc59d640cd18179e`
- Template baseline: `511019bc0bb984e39cf20f44f86121e0fddc8f48`

## Compatibility requirements

- Preserve `/weight-logs` and `/weight-logs/{log_id}`, their HTTP methods,
  status codes, request validation and JSON bodies, including legacy errors.
- Preserve numeric `weight_kg`, ISO measurement dates, UUID identifiers,
  newest-first ordering and the one-measurement-per-date constraint.
- Preserve the original `0001` migration and SQLAlchemy model/schema files.
- Add public metadata, readiness, request IDs and safe structured logging.
- Use template error responses on operational routes; existing weight routes
  retain their original error bodies and receive request IDs in headers.

## Verification progress

- [x] Existing test suite: 25 tests passed before changes.
- [x] API schema and measurement snapshot saved locally.
- [x] PostgreSQL custom-format backup created and restored successfully into
  the separate `emberpath_restore_test` database; revision remains `0001`.
- [x] Current backend image retained as
  `emberpath-weight-service:pre-template-migration` for local rollback.
- [x] Migrated suite: 96 tests passed with warnings treated as errors;
  Ruff lint/format and strict Pyright passed.
- [x] Existing database schema and API compatibility checks: no Alembic
  upgrade operations; CRUD OpenAPI paths, schemas and sampled errors match.
- [x] Isolated browser create/read/update verification against disposable
  PostgreSQL. Browser delete confirmation stalled in automation; deletion
  is covered by the passing PostgreSQL CRUD tests.
- [x] Compose service rebuilt and replaced on 2026-09-20; metadata, health,
  readiness and frontend proxy return HTTP 200. All 128 baseline measurements
  remain identical, with one additional measurement present (129 total).

Temporary verification containers and network were removed. The frontend and
template repositories were not changed. Changes remain uncommitted.

## Local backup

The backup and baseline snapshots are in `.local/migration/`, ignored by Git.
The archive is `emberpath-before-template.dump`; SHA-256:
`6DC0B4B0956E901204FA3671307D830D1B8DE4297821204EEBFF8FAB099EEA59`.
It contains personal data and is not part of the repository.

No schema upgrade is expected beyond the already-applied revision. If runtime
verification fails, use the retained backend image and previous Compose
configuration. Database restoration should not be necessary for a code-only
migration; preserve any measurements entered since the backup.
