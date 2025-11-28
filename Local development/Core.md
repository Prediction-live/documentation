# Requirements
- Containerization engine (ARM use [OrbStack](https://orbstack.dev/))
- [Supabase CLI](https://supabase.com/docs/guides/local-development/cli/getting-started) (install via brew: `brew install supabase/tap/supabase`)

# Setup
1. Go to the project's root
2. Run `supabase init`
3. Run `supabase login`, connect (needs a token from the Users panel of the project)
4. Run `supabase link --project-link <projectRef>` (find the project ref under Settings > General)
5. Run `supabase db pull`, this create a migration file in `supabase/migrations/`, representing curent production state
6. Run `supabase start`
7. Supabase runs at `http://127.0.0.1:54323` / `localhost:54323`
# Workflow
1. Start dev with `supabase start`
2. Edit schema with the GUI at `localhost:54323` or via SQL
3. Save local changes, generate a migration file with `supabase db diff -f <migrationName>`
4. Commit the new file in `supabase/migrations` to Git
5. Push to Github
6. *If Branching enabled on Supabase: automatically deploys the migration to the Preview Branch*
# Useful commands
`(*run in the same folder you ran `supabase start`*)
1. `supabase status`: gives the local credentials
2. `supabase db diff -f <migrationFileName>`: generates the db migration. Automatically run by Supabase in the branch if branching is on. Else:
3. `supabase db push`: push a migration on the prod db

# Local Env
## Supabase

| **Variable**                           | **Value / Source**                                     |
| -------------------------------------- | ------------------------------------------------------ |
| `NEXT_PUBLIC_SUPABASE_URL`             | `http://127.0.0.1:54321`                               |
| `SUPABASE_URL`                         | `http://127.0.0.1:54321`                               |
| `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | Copy **anon key** from `supabase status`               |
| `SUPABASE_SECRET_KEY`                  | Copy **service_role key** from `supabase status`       |
| `SUPABASE_JWKS`                        | `http://127.0.0.1:54321/auth/v1/.well-known/jwks.json` |
## Postgres Database
| **Variable**               | **Value**                                                 |
| -------------------------- | --------------------------------------------------------- |
| `POSTGRES_URL`             | `postgresql://postgres:postgres@127.0.0.1:54322/postgres` |
| `POSTGRES_PRISMA_URL`      | `postgresql://postgres:postgres@127.0.0.1:54322/postgres` |
| `POSTGRES_URL_NON_POOLING` | `postgresql://postgres:postgres@127.0.0.1:54322/postgres` |
| `POSTGRES_USER`            | `postgres`                                                |
| `POSTGRES_PASSWORD`        | `postgres`                                                |
| `POSTGRES_HOST`            | `127.0.0.1`                                               |
| `POSTGRES_DATABASE`        | `postgres`                                                |
