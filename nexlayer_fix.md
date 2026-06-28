# Nexlayer deploy fix — Superset (PINNED)

Do NOT regenerate nexlayer.yaml or Dockerfile. The deploy config is hand-tuned.

Root cause of the prior 500: Apache Superset booted on its default SQLite DB with
no schema initialized (`superset db upgrade` / `superset init` never ran), so every
page hit `sqlite3.OperationalError: no such table: themes` and returned HTTP 500 at
`/` and `/login/`.

Pinned fix (keep all of this):
- `SUPERSET__SQLALCHEMY_DATABASE_URI` points Superset at the postgres metadata pod.
  This is the image's native env-override prefix; the plain `DATABASE_URL` var is
  IGNORED by the apache/superset entrypoint, which is why it kept using SQLite.
- Stable `SUPERSET_SECRET_KEY`.
- App-prefixed, shared-namespace-safe service names: `superset-postgres-service.pod`
  and a shared `${POSTGRES_PASSWORD}` referenced in BOTH the DB pod and the app URI.
- A string-form boot `command` runs the one-time bootstrap before serving:
  `superset db upgrade && superset fab create-admin ... && superset init && /usr/bin/run-server.sh`.

Use the prebuilt `apache/superset:latest` image — do not build Superset from source.
