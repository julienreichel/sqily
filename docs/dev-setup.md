# Development Setup

## Prerequisites
- Docker + Docker Compose (recommended path)
- Ruby matching `.ruby-version` (`3.4.7`)
- Bundler
- PostgreSQL client tools (optional but useful)
- ImageMagick + pkg-config (required to compile `rmagick`)

## Docker-based setup (recommended)

### 1) Environment
Create a `.env` file at repository root with required values:

```bash
cat > .env <<'EOF'
RAILS_MASTER_KEY=dummy
AWS_BUCKET_URL=https://dev_key:dev_secret@s3-us-east-1.amazonaws.com/sqily-dev
EOF
```

Required for boot:
- `AWS_BUCKET_URL` (app parses it at boot in `AwsFileStorage`)
- `RAILS_MASTER_KEY` (required by compose/env setup)

`docker-compose.yml` already provides:
- `DATABASE_URL=postgresql://sqily:sqily@db:5432/sqily_development`
- app binding and volume mounts

### 2) Start services
```bash
docker compose up -d db web
```

### 3) Install gems in container (first time / after Gemfile changes)
```bash
docker compose exec web bundle install
```

### 4) Initialize database
```bash
docker compose exec web bin/rails db:prepare
docker compose exec web bin/rails db:seed
```

### 5) Run server
```bash
docker compose exec web bin/rails server -b 0.0.0.0
```

### 6) Access app
- [http://localhost:3000](http://localhost:3000)

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

## Native extension prerequisites

`rmagick` requires ImageMagick development libraries.

Install once on macOS:
```bash
brew install imagemagick pkg-config
xcode-select --install
```

If bundler still cannot find ImageMagick:
```bash
export PKG_CONFIG_PATH="$(brew --prefix imagemagick)/lib/pkgconfig"
bundle config set --local build.rmagick "--with-opt-dir=$(brew --prefix imagemagick)"
bundle install
```

## Troubleshooting

### App cannot boot due to credentials
Symptom:
- boot errors about encrypted credentials or missing keys.

Fix:
- ensure `RAILS_MASTER_KEY` is set in environment.

### Web container exits immediately on boot
Symptom:
- `docker compose ps` shows `web` not running.
- logs show `URI::InvalidURIError` in `app/models/concerns/aws_file_storage.rb`.

Fix:
- set `AWS_BUCKET_URL` in `.env`.
- recreate containers so env is applied:
```bash
docker compose down
docker compose up -d db web
```

### DB connection errors
Symptom:
- cannot connect to Postgres.

Fix:
- ensure `docker compose ps` shows `db` healthy.
- run Rails tasks with `docker compose exec web ...`.
- then rerun `bin/rails db:prepare`.

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
2. Use full Docker runtime (`docker compose up -d db web`).
3. Run `rails test` once to validate local baseline.
4. Walk one core user flow and one moderator flow manually.
5. Only then start implementation work on new features.
