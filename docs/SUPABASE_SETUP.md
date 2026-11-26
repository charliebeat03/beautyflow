# Supabase setup for BeautyFlow

BeautyFlow uses Supabase as a **managed PostgreSQL database**. The current application keeps its own Express/JWT authentication and connects to PostgreSQL from the backend with the `pg` package. It does not currently use Supabase Auth or the Supabase Data API from the browser.

## 1. Create the Supabase project

1. Open the Supabase dashboard and create a project in the Codexia Systems organization.
2. Choose a nearby region for the intended users and save the database password securely.
3. Wait until the project reports that it is healthy.
4. Open **Connect** in the project dashboard and copy the PostgreSQL connection string. Prefer the connection option appropriate to the backend host; use the Session Pooler if the host cannot reach the direct IPv6 endpoint.

## 2. Configure the backend

Set `DATABASE_URL` only in the backend runtime environment (never in a `VITE_*` variable and never in GitHub source files).

Example shape only; use the exact URI shown by Supabase:

```env
DATABASE_URL=postgresql://postgres:<DB_PASSWORD>@<HOST>:<PORT>/postgres
NODE_ENV=production
PORT=4000
CLIENT_ORIGIN=https://<frontend-domain>
JWT_SECRET=<long-random-secret>
JWT_REFRESH_SECRET=<different-long-random-secret>
REFRESH_TOKEN_HASH_SECRET=<another-long-random-secret>
```

If the database password contains URL-reserved characters, use the correctly URL-encoded password from a generated connection URI.

## 3. Schema and migrations

The versioned SQL files in the repository's `migrations/` directory are the source of truth for the current backend migration runner. The backend runs pending migrations at startup and records them in `schema_migrations`.

- Create the Supabase project/database first; do not manually create BeautyFlow tables.
- Keep the database password and `DATABASE_URL` server-side only.
- Do not run a second migration mechanism against the same database until migration histories have been deliberately reconciled.
- Back up production data before applying any future destructive migration.

## 4. GitHub Actions

The `BeautyFlow CI` workflow uses a temporary PostgreSQL 16 service inside GitHub Actions. It checks backend compilation, frontend TypeScript, unit/static tests, the frontend production build, and applies the SQL migrations twice to check repeatability.

CI does not need Supabase credentials and does not mutate the hosted Supabase project.

## 5. Current integration boundary

Supabase is currently the PostgreSQL provider. BeautyFlow's existing custom JWT authentication remains in place. Image uploads currently use the backend's local upload directory; moving user images to Supabase Storage with private/public bucket policies is a separate implementation task and must be completed before relying on ephemeral production hosts.

## 6. Secrets

Never commit `.env`, database passwords, Supabase secret/service-role keys, JWT secrets, or VAPID private keys. Put runtime secrets in the hosting provider's encrypted environment variables. GitHub Actions should receive only the minimum secrets needed by a specific workflow.
