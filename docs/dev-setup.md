# Development Setup

## Prerequisites
- Docker + Docker Compose (recommended path)
- Ruby matching `.ruby-version` (`3.4.7`)
- Bundler
- PostgreSQL client tools (optional but useful)
- ImageMagick + pkg-config (required to compile `rmagick`)

## Docker-based setup (recommended)

### 1) Environment
Create a `.env` file at repository root with the required value:

```bash
cat > .env <<'EOF'
RAILS_MASTER_KEY=dummy
EOF
```

Required for boot:
- `RAILS_MASTER_KEY` (required by compose/env setup)

Default local Docker behavior:
- `docker-compose.yml` starts a local MinIO service.
- the `web` container defaults `AWS_BUCKET_URL` to the internal Docker service endpoint `minio:9000`.
- a dedicated `minio-setup` service uses the MinIO client to create the development bucket automatically.
- that same setup service also applies anonymous download access for local file previews.
- no AWS bucket is required for the default local setup.
- if your existing `.env` already defines `AWS_BUCKET_URL`, it overrides the MinIO default.

Optional overrides:
- `AWS_BUCKET_URL` supports AWS-hosted S3 URLs, for example: `https://<access_key>:<secret>@s3-eu-central-1.amazonaws.com/<bucket>`

`docker-compose.yml` already provides:
- `DATABASE_URL=postgresql://sqily:sqily@db:5432/sqily_development`
- app binding and volume mounts
- MinIO on `http://127.0.0.1:9000`
- MinIO console on `http://127.0.0.1:9001`
- a one-shot `minio-setup` service that prepares the bucket and public-read access via `minio/mc`

### 2) Start services
```bash
docker compose up -d db minio web
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
docker compose exec web bin/rails test
```

### Run CI-equivalent local checks
```bash
docker compose exec web bin/rails test
docker compose exec web bundle exec standardrb
docker compose exec web bin/brakeman
```
(Equivalent to `bin/ci`, but robust with Docker `exec` PATH differences.)

### Run a specific test file
```bash
docker compose exec web bin/rails test test/controllers/skills_controller_test.rb
```

### Rails console
```bash
docker compose exec web bin/rails console
```

### DB reset from scratch
```bash
docker compose exec web bin/rails db:drop db:create db:schema:load db:seed
```

### Rebuild cron schedule locally
```bash
docker compose exec web bundle exec rake sqily:update_crontab
```

## Scheduled jobs (development awareness)
- Defined in `config/schedule.rb`:
  - hourly: `sqily:hourly`
  - daily 08:00: `sqily:daily`
- Task definitions in `lib/tasks/sqily.rake`.

## Storage and email configuration notes

### File storage
- Default app code uses custom storage concerns (`AwsFileStorage` / `PublicFileStorage`).
- Default local Docker setup uses MinIO, not AWS.
- For S3-backed behavior, configure:
  - `AWS_BUCKET_URL`
  - optional `AWS_BUCKET_PREFIX`
- For local or CI S3-compatible services (for example MinIO), use:
  - a bucket URL whose path is the bucket name,
  - `region` query param when the host is not AWS-shaped,
  - `path_style=true` for MinIO/path-style addressing.
- In Docker development, the app container uses `minio:9000` while browser-facing file URLs use `127.0.0.1:9000`.
- Docker Compose injects that MinIO browser URL through a local-only `MINIO_PUBLIC_BUCKET_URL` env var, so AWS setups keep deriving their public URL from `AWS_BUCKET_URL`.
- Local Docker development also applies anonymous download access automatically so uploaded files can be previewed directly in the browser.

### Create AWS bucket and least-privilege IAM user (AWS CLI, eu-central-1)
The repository includes an automation script:
- [scripts/setup-aws-s3-dev.sh](/Users/julienreichel/git/sqily/scripts/setup-aws-s3-dev.sh)

This is now optional for local development. Use it only if you explicitly want to test against real AWS S3 instead of local MinIO.
When using real AWS via Docker Compose, `web` still depends on the local `minio` and `minio-setup` services. Those services continue to provision the local MinIO bucket, but the Rails app uses your explicit `AWS_BUCKET_URL` override instead of the local MinIO default.

Important compatibility note:
- browser/Trix direct uploads now use an AWS SigV4-compatible presigned POST
- this works with SigV4-only regions such as `eu-central-1`
- local MinIO remains the default development path because it avoids provisioning a real AWS bucket

What it does:
1. Creates (or reuses) an S3 bucket in `eu-central-1`.
2. Configures bucket ownership/public-access settings compatible with current app uploads (`acl: public-read`).
3. Applies a public-read bucket policy so direct object URLs work in the browser.
4. Applies permissive S3 CORS rules so browser-based Trix uploads can POST directly to the bucket.
5. Creates (or reuses) an IAM user.
6. Applies a least-privilege inline policy for that single bucket:
   - bucket metadata: `s3:GetBucketLocation`, `s3:ListBucket`
   - object operations: `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject`, `s3:PutObjectAcl`
7. Creates one URI-compatible access key and writes `AWS_BUCKET_URL` + `AWS_BUCKET_PREFIX=development` to `.env`.

Run:
```bash
chmod +x scripts/setup-aws-s3-dev.sh
./scripts/setup-aws-s3-dev.sh
```

Optional flags:
```bash
./scripts/setup-aws-s3-dev.sh --profile my-admin-profile --project sqily --env-file .env
```

After running it:
```bash
docker compose down
docker compose up -d db minio web
```

Notes:
- The script requires AWS credentials with IAM + S3 provisioning permissions, including bucket policy and bucket CORS updates.
- If the IAM user already has 2 active access keys, AWS blocks creation of a third key; remove one key and rerun.
- To clean old keys:
```bash
aws iam list-access-keys --user-name sqily-dev-s3-app
aws iam delete-access-key --user-name sqily-dev-s3-app --access-key-id <key-id>
```

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
- check that MinIO is running: `docker compose ps`
- if overriding storage config, ensure `AWS_BUCKET_URL` is valid in `.env`.
- recreate containers so env is applied:
```bash
docker compose down
docker compose up -d db minio web
```

### Tests fail with `Aws::S3::Errors::InvalidAccessKeyId`
Symptom:
- failing tests in storage callbacks (for example `AwsFileStorage#save_file`, `AwsAvatarStorage#save_avatar`).

Fix:
- configure a reachable S3-compatible endpoint via `AWS_BUCKET_URL`.
- default Docker development uses local MinIO, so first verify it is running and reachable on `127.0.0.1:9000`.
- for local development, easiest real-AWS path remains `./scripts/setup-aws-s3-dev.sh`.
- for CI, the repository now uses local MinIO instead of a real AWS bucket.
- local Docker and CI both provision MinIO through the same `scripts/setup_minio.sh` script.
- recreate containers:
```bash
docker compose down
docker compose up -d db minio web
```

### DB connection errors
Symptom:
- cannot connect to Postgres.

Fix:
- ensure `docker compose ps` shows `db` healthy.
- run Rails tasks with `docker compose exec web ...`.
- then rerun `docker compose exec web bin/rails db:prepare`.

### Assets or JS/CSS not reflecting changes
Fix:
- restart web container/server,
- clear temp and logs:
```bash
docker compose exec web bin/rails log:clear tmp:clear
```

### Permission/role behavior seems inconsistent
Fix:
- verify current user membership/moderator/admin flags in console,
- confirm community permalink context in URL,
- use seeded fixture users for deterministic role scenarios.

## Recommended first-day workflow
1. Start with Docker and seed data.
2. Use full Docker runtime (`docker compose up -d db minio web`).
3. Run `docker compose exec web bin/rails test` once to validate local baseline.
4. Walk one core user flow and one moderator flow manually.
5. Only then start implementation work on new features.

## GitHub CI setup

GitHub Actions workflow:
- [ci.yml](/Users/julienreichel/git/sqily/.github/workflows/ci.yml)

CI storage model:
1. GitHub Actions starts a local MinIO service.
2. The workflow points `AWS_BUCKET_URL` to that local S3-compatible endpoint.
3. The workflow reuses `scripts/setup_minio.sh` to create the `sqily-test` bucket and enable anonymous download access before `bin/rails db:prepare` and `bin/rails test`.
4. No GitHub secret is required for bucket access in CI.

Repository secrets not required for current CI:
1. `AWS_BUCKET_URL`
2. `AWS_BUCKET_PREFIX`
3. `RAILS_MASTER_KEY`

Development vs CI:
1. Local development now defaults to local MinIO via Docker Compose.
2. Local development may still use a real AWS bucket via `.env` and `scripts/setup-aws-s3-dev.sh` if explicitly desired.
3. CI uses local MinIO by default to exercise the real S3 storage code path without cloud provisioning.
