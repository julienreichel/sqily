# Development Setup

## Prerequisites
- Docker + Docker Compose (recommended path)
- Ruby matching `.ruby-version` (`3.4.1`)
- Bundler
- PostgreSQL client tools (optional but useful)

## Option A: Docker-based setup (recommended)

### 1) Environment
Set required environment values (at least):
- `RAILS_MASTER_KEY`

`docker-compose.yml` already provides:
- `DATABASE_URL=postgresql://sqily:sqily@db:5432/sqily_development`
- app binding and volume mounts

### 2) Start services
```bash
docker compose up --build
```

### 3) Initialize database (if needed)
In another terminal:
```bash
docker compose exec web bin/rails db:prepare
docker compose exec web bin/rails db:seed
```

### 4) Access app
- [http://localhost:3000](http://localhost:3000)

## Option B: Native setup

### 1) Install dependencies
```bash
bundle install
```

### 2) Prepare DB
```bash
bin/rails db:prepare
bin/rails db:seed
```

### 3) Start server
```bash
bin/rails server
```

## Seed data
- `db/seeds.rb` loads fixtures with `db:fixtures:load`.
- This gives a rich baseline dataset for manual testing and feature walkthroughs.

## Common commands

### Run test suite
```bash
rails test
```

### Run CI-equivalent local checks
```bash
bin/ci
```
(Executes tests + `standardrb` + `brakeman`.)

### Run a specific test file
```bash
rails test test/controllers/skills_controller_test.rb
```

### Rails console
```bash
bin/rails console
```

### DB reset from scratch
```bash
bin/rails db:drop db:create db:schema:load db:seed
```

### Rebuild cron schedule locally
```bash
bundle exec rake sqily:update_crontab
```

## Scheduled jobs (development awareness)
- Defined in `config/schedule.rb`:
  - hourly: `sqily:hourly`
  - daily 08:00: `sqily:daily`
- Task definitions in `lib/tasks/sqily.rake`.

## Storage and email configuration notes

### File storage
- Default app code uses custom storage concerns (`AwsFileStorage` / `PublicFileStorage`).
- For S3-backed behavior, configure:
  - `AWS_BUCKET_URL`
  - optional `AWS_BUCKET_PREFIX`

### Mail
- SMTP delivery in production-style runtime uses `SMTP_URL`.
- In local dev, configure letter-opener/mailcatcher equivalent if needed (not preconfigured in repo).

## Troubleshooting

### App cannot boot due to credentials
Symptom:
- boot errors about encrypted credentials or missing keys.

Fix:
- ensure `RAILS_MASTER_KEY` is set in environment.

### DB connection errors
Symptom:
- cannot connect to Postgres.

Fix:
- verify `db` service is healthy (`docker compose ps`),
- verify `DATABASE_URL` matches compose values,
- run `bin/rails db:prepare` again.

### Assets or JS/CSS not reflecting changes
Fix:
- restart web container/server,
- clear temp and logs:
```bash
bin/rails log:clear tmp:clear
```

### Permission/role behavior seems inconsistent
Fix:
- verify current user membership/moderator/admin flags in console,
- confirm community permalink context in URL,
- use seeded fixture users for deterministic role scenarios.

## Recommended first-day workflow
1. Start with Docker and seed data.
2. Run `rails test` once to validate local baseline.
3. Walk one core user flow and one moderator flow manually.
4. Only then start implementation work on new features.
