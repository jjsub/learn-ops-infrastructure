# Tech Stack (AI)

## 1. Run Questions

  🚀 How to Run the App

  The Golden Rule

  You only ever need to be in one directory to control everything:
  cd ~/workspace/lms/learn-ops-infrastructure
  All make commands run from here. That single folder orchestrates all other repos.

  ---
  📋 Run Commands Table

  ┌─────────────────────┬────────────────────────────────────────────────────┬────────────────────────┬───────────────────────────────────────┬────────────────────────────────┐
  │    Relevant File    │                      Location                      │     How to Run It      │             What It's For             │         How It's Used          │
  ├─────────────────────┼────────────────────────────────────────────────────┼────────────────────────┼───────────────────────────────────────┼────────────────────────────────┤
  │                     │                                                    │                        │ First-time only. Interactive wizard   │ Run once on a new machine.     │
  │ setup.sh            │ learn-ops-infrastructure/scripts/setup.sh          │ make setup             │ that clones repos, collects secrets,  │ It's idempotent — safe to      │
  │                     │                                                    │                        │ writes all .env files, and starts the │ re-run, skips steps already    │
  │                     │                                                    │                        │  stack                                │ done                           │
  ├─────────────────────┼────────────────────────────────────────────────────┼────────────────────────┼───────────────────────────────────────┼────────────────────────────────┤
  │                     │                                                    │                        │ The command hub — thin wrappers       │ Always your first stop. Every  │
  │ Makefile            │ learn-ops-infrastructure/Makefile                  │ make <target>          │ around docker compose so you never    │ day-to-day action lives here   │
  │                     │                                                    │                        │ have to remember long flags           │                                │
  ├─────────────────────┼────────────────────────────────────────────────────┼────────────────────────┼───────────────────────────────────────┼────────────────────────────────┤
  │                     │                                                    │                        │ Defines and wires together all 6      │ make up builds images if       │
  │ docker-compose.yml  │ learn-ops-infrastructure/docker-compose.yml        │ make up → runs docker  │ containers (db, api, client,          │ needed and starts everything   │
  │                     │                                                    │ compose up --build -d  │ prometheus, grafana,                  │ in the background (-d =        │
  │                     │                                                    │                        │ postgres_exporter)                    │ detached)                      │
  ├─────────────────────┼────────────────────────────────────────────────────┼────────────────────────┼───────────────────────────────────────┼────────────────────────────────┤
  │ Dockerfile (API)    │ learn-ops-api/Dockerfile                           │ Called automatically   │ Builds the Django API container from  │ Installs Python deps via       │
  │                     │                                                    │ by make up             │ Python 3.11                           │ pipenv, exposes port 8000      │
  ├─────────────────────┼────────────────────────────────────────────────────┼────────────────────────┼───────────────────────────────────────┼────────────────────────────────┤
  │                     │                                                    │ Runs automatically     │ Bootstraps the database before Django │ Waits for Postgres → runs      │
  │ entrypoint.sh       │ learn-ops-api/entrypoint.sh                        │ when the API container │  starts                               │ migrate → loads seed data →    │
  │                     │                                                    │  starts                │                                       │ starts the server              │
  ├─────────────────────┼────────────────────────────────────────────────────┼────────────────────────┼───────────────────────────────────────┼────────────────────────────────┤
  │ Dockerfile (Client) │ learn-ops-client/Dockerfile                        │ Called automatically   │ Builds the React frontend container   │ Installs npm deps, starts dev  │
  │                     │                                                    │ by make up             │ from Node 22                          │ server on port 3000            │
  ├─────────────────────┼────────────────────────────────────────────────────┼────────────────────────┼───────────────────────────────────────┼────────────────────────────────┤
  │ docker-compose.yml  │                                                    │ docker compose up -d   │ Starts the Valkey message broker      │ Must be running before         │
  │ (Valkey)            │ learn-ops-infrastructure/valkey/docker-compose.yml │ from the valkey/       │ (Redis-compatible) on port 6379       │ Monarch. The API publishes     │
  │                     │                                                    │ folder                 │                                       │ messages here; Monarch listens │
  ├─────────────────────┼────────────────────────────────────────────────────┼────────────────────────┼───────────────────────────────────────┼────────────────────────────────┤
  │                     │                                                    │                        │                                       │ Connects to the same           │
  │ docker-compose.yml  │ service-monarch/docker-compose.yml                 │ docker compose up from │ Starts the GitHub issue migrator      │ learningplatform Docker        │
  │ (Monarch)           │                                                    │  service-monarch/      │ microservice                          │ network; listens for migration │
  │                     │                                                    │                        │                                       │  jobs on Valkey                │
  ├─────────────────────┼────────────────────────────────────────────────────┼────────────────────────┼───────────────────────────────────────┼────────────────────────────────┤
  │                     │                                                    │ docker compose exec    │                                       │ Use for one-off tasks like     │
  │ manage.py           │ learn-ops-api/manage.py                            │ api python manage.py   │ Django's built-in management CLI      │ createsuperuser, shell,        │
  │                     │                                                    │ <command>              │                                       │ showmigrations                 │
  └─────────────────────┴────────────────────────────────────────────────────┴────────────────────────┴───────────────────────────────────────┴────────────────────────────────┘

  ---
  🗓️  Day 1 Step-by-Step

  # 1. Go to the control room
  cd ~/workspace/lms/learn-ops-infrastructure

  # 2. First time only — runs the full setup wizard
  make setup

  # 3. Every day after that — start everything
  make up

  # 4. Confirm all containers are healthy
  make ps

  # 5. Watch live logs (Ctrl+C to exit, containers keep running)
  make logs

  Once make up finishes, open your browser:

  ┌───────────────────────┬─────────────────────────────┐
  │          URL          │       What you'll see       │
  ├───────────────────────┼─────────────────────────────┤
  │ http://localhost:3000 │ React frontend (the app)    │
  ├───────────────────────┼─────────────────────────────┤
  │ http://localhost:8000 │ Django API (JSON responses) │
  ├───────────────────────┼─────────────────────────────┤
  │ http://localhost:9090 │ Prometheus metrics          │
  ├───────────────────────┼─────────────────────────────┤
  │ http://localhost:3001 │ Grafana dashboards          │
  └───────────────────────┴─────────────────────────────┘

  ---
  🛑 Stopping & Resetting

  ┌──────────────┬───────────────────────────────────────────────────────────┬────────────────────────────────────────────────────────────────────┐
  │   Command    │                       What It Does                        │                           When to Use It                           │
  ├──────────────┼───────────────────────────────────────────────────────────┼────────────────────────────────────────────────────────────────────┤
  │ make down    │ Stops containers, keeps database data                     │ End of your workday                                                │
  ├──────────────┼───────────────────────────────────────────────────────────┼────────────────────────────────────────────────────────────────────┤
  │ make restart │ down then up --build                                      │ After pulling new code or changing a Dockerfile                    │
  ├──────────────┼───────────────────────────────────────────────────────────┼────────────────────────────────────────────────────────────────────┤
  │ make reset   │ Stops containers and wipes all volumes (database deleted) │ Only when you want a completely clean slate — you'll lose all data │
  └──────────────┴───────────────────────────────────────────────────────────┴────────────────────────────────────────────────────────────────────┘

  ---
  🔍 The One Thing That Trips Everyone Up

  entrypoint.sh runs inside the API container every time it starts. It:
  1. Waits until Postgres is ready (polls with pg_isready)
  2. Runs python manage.py migrate (safe to repeat)
  3. Seeds the database with fixture data — but only if the DB is empty
  4. If DEBUG=True, starts the server through debugpy so VS Code can attach a debugger

  So if the app seems stuck on startup — it's almost always waiting on the database. make logs will show you exactly where it's stuck.

### 1a. Config Files

⚙️  Config & Environment Files

  ▎ The pattern everywhere: secrets live in .env files → code reads them with os.getenv("VAR_NAME") → .env.template shows the shape, never the real values. Never commit .env to git.

  ---
  learn-ops-api/.env (from .env.template)

  Location: learn-ops-api/.env

  ┌────────────────────────────────────────────────────────────────────────┬──────────────────────────────┬──────────────────────────────────────────────────────────────────────┐
  │                            Config Variable                             │        What It's For         │                            How It's Used                             │
  ├────────────────────────────────────────────────────────────────────────┼──────────────────────────────┼──────────────────────────────────────────────────────────────────────┤
  │ LEARN_OPS_DJANGO_SECRET_KEY                                            │ Django's master signing key  │ settings.py → SECRET_KEY = os.getenv(...) — required or Django won't │
  │                                                                        │                              │  start                                                               │
  ├────────────────────────────────────────────────────────────────────────┼──────────────────────────────┼──────────────────────────────────────────────────────────────────────┤
  │ LEARN_OPS_CLIENT_ID / LEARN_OPS_SECRET_KEY                             │ GitHub OAuth app credentials │ django-allauth reads these to redirect users through the GitHub      │
  │                                                                        │                              │ login flow                                                           │
  ├────────────────────────────────────────────────────────────────────────┼──────────────────────────────┼──────────────────────────────────────────────────────────────────────┤
  │ LEARN_OPS_HOST / LEARN_OPS_DB / LEARN_OPS_USER / LEARN_OPS_PASSWORD /  │ PostgreSQL connection        │ settings.py DATABASES dict — Django ORM uses all five to open a DB   │
  │ LEARN_OPS_PORT                                                         │ details                      │ connection                                                           │
  ├────────────────────────────────────────────────────────────────────────┼──────────────────────────────┼──────────────────────────────────────────────────────────────────────┤
  │ VALKEY_HOST / VALKEY_PORT / VALKEY_DB                                  │ Valkey (message broker)      │ API publishes migration jobs here; Monarch listens on the same       │
  │                                                                        │ connection                   │ address                                                              │
  ├────────────────────────────────────────────────────────────────────────┼──────────────────────────────┼──────────────────────────────────────────────────────────────────────┤
  │ DEBUG / DEVELOPMENT_MODE                                               │ Toggle dev vs prod behaviour │ DEBUG=True enables the debug toolbar, verbose errors, and the        │
  │                                                                        │                              │ debugpy remote debugger                                              │
  ├────────────────────────────────────────────────────────────────────────┼──────────────────────────────┼──────────────────────────────────────────────────────────────────────┤
  │ SLACK_BOT_TOKEN / GITHUB_TOKEN                                         │ External service credentials │ Views call GitHub API and post Slack notifications using these       │
  └────────────────────────────────────────────────────────────────────────┴──────────────────────────────┴──────────────────────────────────────────────────────────────────────┘

  ---
  learn-ops-client/.env

  Location: learn-ops-client/.env

  ┌─────────────────────┬──────────────────────────────┬───────────────────────────────────────────────────────────────────────────────────────────────────┐
  │   Config Variable   │        What It's For         │                                           How It's Used                                           │
  ├─────────────────────┼──────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────┤
  │ REACT_APP_API_URI   │ Base URL of the Django API   │ Every fetch() call in the React services is prefixed with this — change it to switch environments │
  ├─────────────────────┼──────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────┤
  │ REACT_APP_ENV       │ Active environment name      │ Used to toggle dev-only UI behaviour or logging verbosity                                         │
  ├─────────────────────┼──────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────┤
  │ CHOKIDAR_USEPOLLING │ Fix hot-reload inside Docker │ Docker can't receive native filesystem events from the host, so polling is used instead           │
  ├─────────────────────┼──────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────┤
  │ GENERATE_SOURCEMAP  │ Controls build source maps   │ Set to false to keep production builds smaller and avoid leaking source code                      │
  └─────────────────────┴──────────────────────────────┴───────────────────────────────────────────────────────────────────────────────────────────────────┘

  ---
  learn-ops-infrastructure/.env

  Location: learn-ops-infrastructure/.env

  ┌──────────────────────────────────┬─────────────────────────────────────────────────┬─────────────────────────────────────────────────────────────────────────────────────────┐
  │         Config Variable          │                  What It's For                  │                                      How It's Used                                      │
  ├──────────────────────────────────┼─────────────────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────┤
  │ POSTGRES_DB                      │ Database name Postgres creates on first boot    │ Passed directly into the postgres:16 Docker container as an init env var                │
  ├──────────────────────────────────┼─────────────────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────┤
  │ POSTGRES_USER /                  │ DB credentials                                  │ Must match LEARN_OPS_USER / LEARN_OPS_PASSWORD in the API's .env or the API can't       │
  │ POSTGRES_PASSWORD                │                                                 │ connect                                                                                 │
  ├──────────────────────────────────┼─────────────────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────┤
  │ DATA_SOURCE_NAME                 │ Full connection string for the Postgres         │ Lets Prometheus scrape Postgres health metrics via postgres_exporter on port 9187       │
  │                                  │ exporter                                        │                                                                                         │
  └──────────────────────────────────┴─────────────────────────────────────────────────┴─────────────────────────────────────────────────────────────────────────────────────────┘

  ---
  service-monarch/.env (from .env.template)

  Location: service-monarch/.env

  ┌─────────────────────────────────┬──────────────────────────────────┬──────────────────────────────────────────────────────────────────────────────────────────────┐
  │         Config Variable         │          What It's For           │                                        How It's Used                                         │
  ├─────────────────────────────────┼──────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────┤
  │ GH_PAT                          │ GitHub Personal Access Token     │ Monarch authenticates all GitHub API calls with this when creating issues on student repos   │
  ├─────────────────────────────────┼──────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────┤
  │ VALKEY_HOST / VALKEY_PORT       │ Same broker the API publishes to │ Monarch subscribes to channel_migrate_issue_tickets — must point to the same Valkey instance │
  ├─────────────────────────────────┼──────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────┤
  │ SLACK_WEBHOOK_URL / SLACK_TOKEN │ Slack integration                │ Monarch posts success or failure notifications to the cohort's Slack channel after migration │
  └─────────────────────────────────┴──────────────────────────────────┴──────────────────────────────────────────────────────────────────────────────────────────────┘

  ---
  learn-ops-infrastructure/prometheus.yml

  Location: learn-ops-infrastructure/prometheus.yml

  ┌──────────────────────────────────┬────────────────────────────────────────┬────────────────────────────────────────────────────────────────────────────────┐
  │            Config Key            │             What It's For              │                                 How It's Used                                  │
  ├──────────────────────────────────┼────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
  │ scrape_interval                  │ How often Prometheus polls for metrics │ Every N seconds it hits all targets below — 15s is the default                 │
  ├──────────────────────────────────┼────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
  │ targets → api:8000               │ Django API metrics endpoint            │ Prometheus scrapes /metrics/metrics — exposed by the django-prometheus package │
  ├──────────────────────────────────┼────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
  │ targets → postgres_exporter:9187 │ Postgres metrics endpoint              │ Lets Grafana display DB-level stats like query counts and active connections   │
  └──────────────────────────────────┴────────────────────────────────────────┴────────────────────────────────────────────────────────────────────────────────┘

  ---
  learn-ops-api/pytest.ini

  Location: learn-ops-api/pytest.ini

  ┌────────────────────────┬───────────────────────────────────────────┬─────────────────────────────────────────────────────────────────────────────────────────────────┐
  │       Config Key       │               What It's For               │                                          How It's Used                                          │
  ├────────────────────────┼───────────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────┤
  │ DJANGO_SETTINGS_MODULE │ Tells pytest which Django config to load  │ Without this, Django has no settings and all tests crash immediately                            │
  ├────────────────────────┼───────────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────┤
  │ --reuse-db             │ Re-uses the test DB between runs          │ Skips the expensive teardown/rebuild cycle — tests run much faster after the first run          │
  ├────────────────────────┼───────────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────┤
  │ --nomigrations         │ Skips migration files during test setup   │ Creates tables directly from models — faster, and avoids failures from migration history issues │
  ├────────────────────────┼───────────────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────────────────┤
  │ testpaths              │ Restricts where pytest searches for tests │ Prevents accidental test discovery in the wrong directories                                     │
  └────────────────────────┴───────────────────────────────────────────┴─────────────────────────────────────────────────────────────────────────────────────────────────┘

  ---
  ⚠️  The One Thing That Breaks Everything

  These variables across two different files describe the same database — one creates it, one connects to it. They must match exactly:

  learn-ops-infrastructure/.env  →  POSTGRES_DB / POSTGRES_USER / POSTGRES_PASSWORD
  learn-ops-api/.env             →  LEARN_OPS_DB / LEARN_OPS_USER / LEARN_OPS_PASSWORD

### 1b. How to Start It


  ---
  Before Anything: Prerequisites

  You need these installed on your machine:
  - Docker Desktop (running — check the whale icon in your menu bar)
  - Git
  - Make

  ---
  First Time Only — Full Setup Wizard

  cd ~/workspace/lms/learn-ops-infrastructure
  make setup

  This runs scripts/setup.sh which does everything for you in order:
  1. Checks Docker, Git, Make are installed
  2. Clones learn-ops-api, learn-ops-client, and service-monarch repos
  3. Asks for your GitHub PAT, Slack token, and OAuth credentials
  4. Writes all .env files from the templates automatically
  5. Offers to start the stack when done

  ▎ ✅ Run this once. It's safe to re-run — it skips steps already done.

  ---
  Every Day After That — Start the Stack

  Step 1 — Start Valkey first (the message broker everything depends on):
  cd ~/workspace/lms/learn-ops-infrastructure/valkey
  docker compose up -d

  Step 2 — Start the main stack:
  cd ~/workspace/lms/learn-ops-infrastructure
  make up

  Step 3 — Check everything came up healthy:
  make ps

  You should see all containers with status Up or healthy. If anything shows Exit or Restarting, run make logs to see why.

  Step 4 — Open the app:

  ┌───────────────────────┬─────────────────────────────┐
  │          URL          │        What's there         │
  ├───────────────────────┼─────────────────────────────┤
  │ http://localhost:3000 │ ✅ React frontend — the app │
  ├───────────────────────┼─────────────────────────────┤
  │ http://localhost:8000 │ ✅ Django API — raw JSON    │
  ├───────────────────────┼─────────────────────────────┤
  │ http://localhost:3001 │ 📊 Grafana dashboards       │
  ├───────────────────────┼─────────────────────────────┤
  │ http://localhost:9090 │ 📈 Prometheus metrics       │
  └───────────────────────┴─────────────────────────────┘

  ---
  Optional — Start Monarch (GitHub Issue Migrator)

  Only needed if you're working on the issue migration feature:
  cd ~/workspace/lms/service-monarch
  docker compose up -d

  ---
  What Happens Automatically on Startup

  When you run make up, the API container runs entrypoint.sh before Django starts. You don't need to do this manually — it handles:

  1. Waits for Postgres to be ready       (polls every 1 second)
  2. Runs database migrations             (python manage.py migrate)
  3. Seeds the database with starter data (only if DB is empty)
  4. Starts Django dev server on :8000

  ▎ If startup seems stuck, it's almost always waiting on Postgres. Run make logs to watch it in real time.

  ---
  Everyday Commands Cheatsheet

  # All commands run from here:
  cd ~/workspace/lms/learn-ops-infrastructure

  make up          # Start everything (builds images if needed)
  make down        # Stop everything (data is kept)
  make restart     # Stop + rebuild + start (use after pulling new code)
  make logs        # Stream live logs from all containers (Ctrl+C is safe)
  make ps          # See status of all containers
  make reset       # ⚠️   WIPES DATABASE — full clean slate

  ---
  The Most Common First-Day Problems

  ┌──────────────────────────┬─────────────────────────────────┬─────────────────────────────────────────────────────────────────────────────────────┐
  │         Symptom          │          Likely Cause           │                                         Fix                                         │
  ├──────────────────────────┼─────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
  │ API stuck on startup     │ Postgres not ready yet          │ Wait — entrypoint.sh retries automatically. Watch with make logs                    │
  ├──────────────────────────┼─────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
  │ network not found error  │ Valkey container not running    │ Start Valkey first: cd valkey && docker compose up -d                               │
  ├──────────────────────────┼─────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
  │ Login with GitHub fails  │ OAuth credentials wrong in .env │ Check LEARN_OPS_CLIENT_ID and LEARN_OPS_SECRET_KEY in learn-ops-api/.env            │
  ├──────────────────────────┼─────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
  │ Frontend can't reach API │ Wrong API URL in client .env    │ Check REACT_APP_API_URI=http://localhost:8000 in learn-ops-client/.env              │
  ├──────────────────────────┼─────────────────────────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
  │ DB credentials mismatch  │ .env files out of sync          │ POSTGRES_USER/PASSWORD in infra .env must match LEARN_OPS_USER/PASSWORD in api .env │
  └──────────────────────────┴─────────────────────────────────┴─────────────────────────────────────────────────────────────────────────────────────┘



### 1c. Where to Access It

Based on everything I've already read across the codebase — docker-compose.yml, settings.py, urls.py, and Monarch's config — here's the complete access table:

  ---
  🌐 Where to Access It

  ┌──────────────────────┬──────┬───────────────────────────────────────┬─────────────────────────────────────────────────────────┐
  │       Service        │ Port │                  URL                  │                     What You'll See                     │
  ├──────────────────────┼──────┼───────────────────────────────────────┼─────────────────────────────────────────────────────────┤
  │ React Frontend       │ 3000 │ http://localhost:3000                 │ The app — login with GitHub here                        │
  ├──────────────────────┼──────┼───────────────────────────────────────┼─────────────────────────────────────────────────────────┤
  │ Django API           │ 8000 │ http://localhost:8000                 │ Raw JSON — all REST endpoints                           │
  ├──────────────────────┼──────┼───────────────────────────────────────┼─────────────────────────────────────────────────────────┤
  │ Django Admin         │ 8000 │ http://localhost:8000/admin           │ Admin UI to manage DB records directly                  │
  ├──────────────────────┼──────┼───────────────────────────────────────┼─────────────────────────────────────────────────────────┤
  │ Django Log Viewer    │ 8000 │ http://localhost:8000/logs            │ In-app log browser (stored in DB)                       │
  ├──────────────────────┼──────┼───────────────────────────────────────┼─────────────────────────────────────────────────────────┤
  │ API Metrics          │ 8000 │ http://localhost:8000/metrics/metrics │ Raw Prometheus metrics exposed by Django                │
  ├──────────────────────┼──────┼───────────────────────────────────────┼─────────────────────────────────────────────────────────┤
  │ Debug Toolbar        │ 8000 │ http://localhost:8000/__debug__/      │ SQL queries, headers, cache info (only when DEBUG=True) │
  ├──────────────────────┼──────┼───────────────────────────────────────┼─────────────────────────────────────────────────────────┤
  │ Grafana              │ 3001 │ http://localhost:3001                 │ Visual dashboards for API and DB metrics                │
  ├──────────────────────┼──────┼───────────────────────────────────────┼─────────────────────────────────────────────────────────┤
  │ Prometheus           │ 9090 │ http://localhost:9090                 │ Raw metrics scraper UI — query and explore metrics      │
  ├──────────────────────┼──────┼───────────────────────────────────────┼─────────────────────────────────────────────────────────┤
  │ Postgres Exporter    │ 9187 │ http://localhost:9187/metrics         │ Raw DB metrics scraped by Prometheus                    │
  ├──────────────────────┼──────┼───────────────────────────────────────┼─────────────────────────────────────────────────────────┤
  │ Valkey (broker)      │ 6379 │ No browser UI — CLI only              │ Message broker between API and Monarch                  │
  ├──────────────────────┼──────┼───────────────────────────────────────┼─────────────────────────────────────────────────────────┤
  │ Monarch Metrics      │ 8080 │ http://localhost:8080                 │ Prometheus metrics for the Monarch microservice         │
  ├──────────────────────┼──────┼───────────────────────────────────────┼─────────────────────────────────────────────────────────┤
  │ Monarch Log Viewer   │ 8081 │ http://localhost:8081                 │ Web UI to browse Monarch logs                           │
  ├──────────────────────┼──────┼───────────────────────────────────────┼─────────────────────────────────────────────────────────┤
  │ Monarch Health Check │ 8081 │ http://localhost:8081/health          │ JSON health status of the Monarch service               │
  └──────────────────────┴──────┴───────────────────────────────────────┴─────────────────────────────────────────────────────────┘

  ---
  🗺️  Quick Mental Map

  You (browser)
    │
    ├── :3000  → React App        (what users interact with)
    │               ↕ API calls
    ├── :8000  → Django API       (the brain — handles all logic)
    │               ↕ reads/writes
    │           PostgreSQL :5432  (data — no browser UI)
    │               ↕ metrics
    ├── :9187  → Postgres Exporter
    │               ↕ scraped by
    ├── :9090  → Prometheus       (collects all metrics)
    │               ↕ visualized by
    ├── :3001  → Grafana          (dashboards)
    │
    └── Monarch (separate stack)
          ├── :8080  Metrics
          └── :8081  Logs + Health

  ---

  ▎ Tip: Start with http://localhost:3000. If GitHub login works, the whole stack is healthy — the frontend, API, database, and OAuth are all talking to each other correctly.

### 1d. Service Dependencies

I have everything I need from the codebase already. Here's the full picture:

  ---
  🗂️  Services — Big Picture Guide

  The Table

  ┌───────────────────┬───────────────────────────────────┬───────────────────────────────────────────────────────────────────────────────────────────┐
  │      Service      │            Depends On             │                                            Why                                            │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ React Frontend    │ Django API                        │ Has no logic of its own — every piece of data it shows comes from an API call to Django   │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ Django API        │ PostgreSQL, Valkey, GitHub, Slack │ The brain — handles all business logic, auth, and data. Nothing works without it          │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ PostgreSQL        │ Nothing                           │ Stores all permanent data — users, cohorts, students, courses, assessments                │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ Valkey            │ Nothing                           │ The message bus — API drops jobs here, Monarch picks them up. Must be running before both │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ Monarch           │ Valkey, GitHub API, Slack         │ Listens for migration jobs from Valkey. Useless without the broker and GitHub access      │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ Prometheus        │ Django API, Postgres Exporter     │ Scrapes metrics from both — blind without them running                                    │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ Postgres Exporter │ PostgreSQL                        │ Translates raw Postgres stats into a format Prometheus can read                           │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ Grafana           │ Prometheus                        │ Purely a visualizer — just reads what Prometheus already collected                        │
  └───────────────────┴───────────────────────────────────┴───────────────────────────────────────────────────────────────────────────────────────────┘

  ---
  What Each Service Does + What It Needs + What It Produces

  ---
  🖥️  React Frontend

  Role: The face of the app — what instructors and students look at in the browser.

  What it does: Renders dashboards, student lists, cohort views. Talks to Django for every piece of data. Has 3 different UIs depending on role (instructor / staff / student).

  ┌───────────────────┬───────────────────────────┐
  │      Expects      │           Value           │
  ├───────────────────┼───────────────────────────┤
  │ REACT_APP_API_URI │ URL of the Django API     │
  ├───────────────────┼───────────────────────────┤
  │ REACT_APP_ENV     │ development or production │
  └───────────────────┴───────────────────────────┘

  Output: A web app on http://localhost:3000

  ---
  🧠 Django API

  Role: The brain. Every feature in the system passes through here.

  What it does: Handles GitHub OAuth login, exposes ~30 REST endpoints, reads/writes to Postgres, publishes jobs to Valkey, calls GitHub and Slack APIs directly.

  ┌────────────────────────────────────────────┬──────────────────────────────┐
  │                  Expects                   │            Value             │
  ├────────────────────────────────────────────┼──────────────────────────────┤
  │ LEARN_OPS_DJANGO_SECRET_KEY                │ Django signing key           │
  ├────────────────────────────────────────────┼──────────────────────────────┤
  │ LEARN_OPS_CLIENT_ID / LEARN_OPS_SECRET_KEY │ GitHub OAuth credentials     │
  ├────────────────────────────────────────────┼──────────────────────────────┤
  │ LEARN_OPS_HOST/DB/USER/PASSWORD/PORT       │ PostgreSQL connection        │
  ├────────────────────────────────────────────┼──────────────────────────────┤
  │ VALKEY_HOST / VALKEY_PORT                  │ Message broker address       │
  ├────────────────────────────────────────────┼──────────────────────────────┤
  │ SLACK_BOT_TOKEN / GITHUB_TOKEN             │ External service credentials │
  └────────────────────────────────────────────┴──────────────────────────────┘

  Output: JSON REST API on http://localhost:8000

  ---
  🗄️  PostgreSQL

  Role: The source of truth. All data lives here permanently.

  What it does: Stores every record — users, cohorts, students, courses, projects, assessments, notes. Django talks to it exclusively through the ORM.

  ┌───────────────────┬─────────────────────────┐
  │      Expects      │          Value          │
  ├───────────────────┼─────────────────────────┤
  │ POSTGRES_DB       │ Database name to create │
  ├───────────────────┼─────────────────────────┤
  │ POSTGRES_USER     │ Login user              │
  ├───────────────────┼─────────────────────────┤
  │ POSTGRES_PASSWORD │ Login password          │
  └───────────────────┴─────────────────────────┘

  Output: A running database on port 5432 — no browser UI

  ---
  📨 Valkey

  Role: The post box between the API and Monarch.

  What it does: API publishes a message to channel channel_migrate_issue_tickets after creating team repos. Monarch listens on that channel and picks up the job. Neither service
  needs to know the other exists directly.

  ┌──────────────────┬───────────────────────────────────┐
  │     Expects      │               Value               │
  ├──────────────────┼───────────────────────────────────┤
  │ No config needed │ Runs with defaults out of the box │
  └──────────────────┴───────────────────────────────────┘

  Output: A message broker on port 6379 — no browser UI

  ---
  🦋 Monarch

  Role: The GitHub issue migrator — a focused, single-purpose microservice.

  What it does: Waits for a Valkey message → fetches issues from a template GitHub repo → copies them into every student team's repo → notifies Slack when done (or if it fails).

  ┌─────────────────────────────────┬─────────────────────────────────────────┐
  │             Expects             │                  Value                  │
  ├─────────────────────────────────┼─────────────────────────────────────────┤
  │ GH_PAT                          │ GitHub Personal Access Token            │
  ├─────────────────────────────────┼─────────────────────────────────────────┤
  │ VALKEY_HOST / VALKEY_PORT       │ Must match the same Valkey the API uses │
  ├─────────────────────────────────┼─────────────────────────────────────────┤
  │ SLACK_WEBHOOK_URL / SLACK_TOKEN │ To post success/failure notifications   │
  └─────────────────────────────────┴─────────────────────────────────────────┘

  Output: Issues migrated on GitHub + Slack notification. Metrics on :8080, logs on :8081

  ---
  📈 Prometheus

  Role: The metric collector — silently gathers numbers from every service.

  What it does: Every 15 seconds it polls the Django API and Postgres Exporter for stats (request counts, query times, error rates, DB connections) and stores them as a time series.

  ┌───────────────────────────┬───────────────────────────┐
  │          Expects          │           Value           │
  ├───────────────────────────┼───────────────────────────┤
  │ prometheus.yml            │ List of targets to scrape │
  ├───────────────────────────┼───────────────────────────┤
  │ Django API running        │ To hit /metrics/metrics   │
  ├───────────────────────────┼───────────────────────────┤
  │ Postgres Exporter running │ To hit :9187/metrics      │
  └───────────────────────────┴───────────────────────────┘

  Output: A queryable metrics store on http://localhost:9090

  ---
  🔌 Postgres Exporter

  Role: A translator — converts internal Postgres stats into Prometheus-readable format.

  What it does: Connects to Postgres and exposes database-level metrics (active connections, query durations, table sizes) on an HTTP endpoint for Prometheus to scrape.

  ┌──────────────────┬─────────────────────────────────┐
  │     Expects      │              Value              │
  ├──────────────────┼─────────────────────────────────┤
  │ DATA_SOURCE_NAME │ Full Postgres connection string │
  └──────────────────┴─────────────────────────────────┘

  Output: Raw metrics endpoint on http://localhost:9187/metrics

  ---
  📊 Grafana

  Role: The dashboard — turns raw numbers into readable charts.

  What it does: Connects to Prometheus as a data source and renders visual dashboards. No data of its own — purely a viewer.

  ┌────────────────────┬──────────────────────────┐
  │      Expects       │          Value           │
  ├────────────────────┼──────────────────────────┤
  │ Prometheus running │ To query for metric data │
  └────────────────────┴──────────────────────────┘

  Output: Visual dashboards on http://localhost:3001

  ---
  🔗 Startup Order That Matters

  PostgreSQL  ──┐
                ├──▶  Django API  ──▶  React Frontend
  Valkey      ──┘         │
                │          └──▶  Prometheus  ──▶  Grafana
                │
                └──▶  Monarch
                           │
                Postgres Exporter ──▶  Prometheus

  ▎ Rule of thumb: Valkey and Postgres first, then everything else. make up handles this automatically via depends_on in docker-compose.yml.

### 1e. Main Entry Points

  🗂️  Services — Big Picture Guide

  The Table

  ┌───────────────────┬───────────────────────────────────┬───────────────────────────────────────────────────────────────────────────────────────────┐
  │      Service      │            Depends On             │                                            Why                                            │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ React Frontend    │ Django API                        │ Has no logic of its own — every piece of data it shows comes from an API call to Django   │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ Django API        │ PostgreSQL, Valkey, GitHub, Slack │ The brain — handles all business logic, auth, and data. Nothing works without it          │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ PostgreSQL        │ Nothing                           │ Stores all permanent data — users, cohorts, students, courses, assessments                │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ Valkey            │ Nothing                           │ The message bus — API drops jobs here, Monarch picks them up. Must be running before both │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ Monarch           │ Valkey, GitHub API, Slack         │ Listens for migration jobs from Valkey. Useless without the broker and GitHub access      │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ Prometheus        │ Django API, Postgres Exporter     │ Scrapes metrics from both — blind without them running                                    │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ Postgres Exporter │ PostgreSQL                        │ Translates raw Postgres stats into a format Prometheus can read                           │
  ├───────────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────┤
  │ Grafana           │ Prometheus                        │ Purely a visualizer — just reads what Prometheus already collected                        │
  └───────────────────┴───────────────────────────────────┴───────────────────────────────────────────────────────────────────────────────────────────┘

  ---
  What Each Service Does + What It Needs + What It Produces

  ---
  🖥️  React Frontend

  Role: The face of the app — what instructors and students look at in the browser.

  What it does: Renders dashboards, student lists, cohort views. Talks to Django for every piece of data. Has 3 different UIs depending on role (instructor / staff / student).

  ┌───────────────────┬───────────────────────────┐
  │      Expects      │           Value           │
  ├───────────────────┼───────────────────────────┤
  │ REACT_APP_API_URI │ URL of the Django API     │
  ├───────────────────┼───────────────────────────┤
  │ REACT_APP_ENV     │ development or production │
  └───────────────────┴───────────────────────────┘

  Output: A web app on http://localhost:3000

  ---
  🧠 Django API

  Role: The brain. Every feature in the system passes through here.

  What it does: Handles GitHub OAuth login, exposes ~30 REST endpoints, reads/writes to Postgres, publishes jobs to Valkey, calls GitHub and Slack APIs directly.

  ┌────────────────────────────────────────────┬──────────────────────────────┐
  │                  Expects                   │            Value             │
  ├────────────────────────────────────────────┼──────────────────────────────┤
  │ LEARN_OPS_DJANGO_SECRET_KEY                │ Django signing key           │
  ├────────────────────────────────────────────┼──────────────────────────────┤
  │ LEARN_OPS_CLIENT_ID / LEARN_OPS_SECRET_KEY │ GitHub OAuth credentials     │
  ├────────────────────────────────────────────┼──────────────────────────────┤
  │ LEARN_OPS_HOST/DB/USER/PASSWORD/PORT       │ PostgreSQL connection        │
  ├────────────────────────────────────────────┼──────────────────────────────┤
  │ VALKEY_HOST / VALKEY_PORT                  │ Message broker address       │
  ├────────────────────────────────────────────┼──────────────────────────────┤
  │ SLACK_BOT_TOKEN / GITHUB_TOKEN             │ External service credentials │
  └────────────────────────────────────────────┴──────────────────────────────┘

  Output: JSON REST API on http://localhost:8000

  ---
  🗄️  PostgreSQL

  Role: The source of truth. All data lives here permanently.

  What it does: Stores every record — users, cohorts, students, courses, projects, assessments, notes. Django talks to it exclusively through the ORM.

  ┌───────────────────┬─────────────────────────┐
  │      Expects      │          Value          │
  ├───────────────────┼─────────────────────────┤
  │ POSTGRES_DB       │ Database name to create │
  ├───────────────────┼─────────────────────────┤
  │ POSTGRES_USER     │ Login user              │
  ├───────────────────┼─────────────────────────┤
  │ POSTGRES_PASSWORD │ Login password          │
  └───────────────────┴─────────────────────────┘

  Output: A running database on port 5432 — no browser UI

  ---
  📨 Valkey

  Role: The post box between the API and Monarch.

  What it does: API publishes a message to channel channel_migrate_issue_tickets after creating team repos. Monarch listens on that channel and picks up the job. Neither service
  needs to know the other exists directly.

  ┌──────────────────┬───────────────────────────────────┐
  │     Expects      │               Value               │
  ├──────────────────┼───────────────────────────────────┤
  │ No config needed │ Runs with defaults out of the box │
  └──────────────────┴───────────────────────────────────┘

  Output: A message broker on port 6379 — no browser UI

  ---
  🦋 Monarch

  Role: The GitHub issue migrator — a focused, single-purpose microservice.

  What it does: Waits for a Valkey message → fetches issues from a template GitHub repo → copies them into every student team's repo → notifies Slack when done (or if it fails).

  ┌─────────────────────────────────┬─────────────────────────────────────────┐
  │             Expects             │                  Value                  │
  ├─────────────────────────────────┼─────────────────────────────────────────┤
  │ GH_PAT                          │ GitHub Personal Access Token            │
  ├─────────────────────────────────┼─────────────────────────────────────────┤
  │ VALKEY_HOST / VALKEY_PORT       │ Must match the same Valkey the API uses │
  ├─────────────────────────────────┼─────────────────────────────────────────┤
  │ SLACK_WEBHOOK_URL / SLACK_TOKEN │ To post success/failure notifications   │
  └─────────────────────────────────┴─────────────────────────────────────────┘

  Output: Issues migrated on GitHub + Slack notification. Metrics on :8080, logs on :8081

  ---
  📈 Prometheus

  Role: The metric collector — silently gathers numbers from every service.

  What it does: Every 15 seconds it polls the Django API and Postgres Exporter for stats (request counts, query times, error rates, DB connections) and stores them as a time series.

  ┌───────────────────────────┬───────────────────────────┐
  │          Expects          │           Value           │
  ├───────────────────────────┼───────────────────────────┤
  │ prometheus.yml            │ List of targets to scrape │
  ├───────────────────────────┼───────────────────────────┤
  │ Django API running        │ To hit /metrics/metrics   │
  ├───────────────────────────┼───────────────────────────┤
  │ Postgres Exporter running │ To hit :9187/metrics      │
  └───────────────────────────┴───────────────────────────┘

  Output: A queryable metrics store on http://localhost:9090

  ---
  🔌 Postgres Exporter

  Role: A translator — converts internal Postgres stats into Prometheus-readable format.

  What it does: Connects to Postgres and exposes database-level metrics (active connections, query durations, table sizes) on an HTTP endpoint for Prometheus to scrape.

  ┌──────────────────┬─────────────────────────────────┐
  │     Expects      │              Value              │
  ├──────────────────┼─────────────────────────────────┤
  │ DATA_SOURCE_NAME │ Full Postgres connection string │
  └──────────────────┴─────────────────────────────────┘

  Output: Raw metrics endpoint on http://localhost:9187/metrics

  ---
  📊 Grafana

  Role: The dashboard — turns raw numbers into readable charts.

  What it does: Connects to Prometheus as a data source and renders visual dashboards. No data of its own — purely a viewer.

  ┌────────────────────┬──────────────────────────┐
  │      Expects       │          Value           │
  ├────────────────────┼──────────────────────────┤
  │ Prometheus running │ To query for metric data │
  └────────────────────┴──────────────────────────┘

  Output: Visual dashboards on http://localhost:3001

  ---
  🔗 Startup Order That Matters

  PostgreSQL  ──┐
                ├──▶  Django API  ──▶  React Frontend
  Valkey      ──┘         │
                │          └──▶  Prometheus  ──▶  Grafana
                │
                └──▶  Monarch
                           │
                Postgres Exporter ──▶  Prometheus

  ▎ Rule of thumb: Valkey and Postgres first, then everything else. make up handles this automatically via depends_on in docker-compose.yml.

✻ Sautéed for 36s

❯ provide explanation on what are the Main Entry Points | Service | Startup File | Routes / URL Config File |

  Read 2 files (ctrl+o to expand)

⏺ Here's the full entry points guide:

  ---
  🚪 Main Entry Points

  The Table

  ┌────────────────┬─────────────────────────────────────────────────┬────────────────────────────────────────────────────────────────────────────────────┐
  │    Service     │                  Startup File                   │                              Routes / URL Config File                              │
  ├────────────────┼─────────────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
  │ React Frontend │ src/index.js                                    │ src/components/LearnOps.js → ApplicationViews.js / StudentViews.js / StaffViews.js │
  ├────────────────┼─────────────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
  │ Django API     │ LearningPlatform/wsgi.py (prod) manage.py (dev) │ LearningPlatform/urls.py                                                           │
  ├────────────────┼─────────────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
  │ Monarch        │ service/main.py                                 │ No HTTP routes — listens on a Valkey channel                                       │
  ├────────────────┼─────────────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
  │ PostgreSQL     │ Docker image handles it                         │ No code — table structure comes from Django migrations                             │
  ├────────────────┼─────────────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
  │ Valkey         │ Docker image handles it                         │ No routes — pub/sub channels defined in code                                       │
  ├────────────────┼─────────────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
  │ Prometheus     │ Docker image handles it                         │ learn-ops-infrastructure/prometheus.yml                                            │
  ├────────────────┼─────────────────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
  │ Grafana        │ Docker image handles it                         │ No code — configured via browser UI                                                │
  └────────────────┴─────────────────────────────────────────────────┴────────────────────────────────────────────────────────────────────────────────────┘

  ---
  Service by Service

  ---
  ⚛️  React Frontend

  Startup file: learn-ops-client/src/index.js
  This is where React mounts the app into the HTML page. One line does it:
  ReactDOM.render(<Router><LearnOps /></Router>, document.getElementById("root"))

  Route logic: src/components/LearnOps.js
  This is the gatekeeper — checks if you're logged in, then picks which set of routes to load based on your role:

  ┌────────────┬─────────────────────┬──────────────────────────────────────────────────────────────────┐
  │    Role    │     Routes File     │                          What it shows                           │
  ├────────────┼─────────────────────┼──────────────────────────────────────────────────────────────────┤
  │ Instructor │ ApplicationViews.js │ Dashboard, cohorts, courses, books, projects, assessments, teams │
  ├────────────┼─────────────────────┼──────────────────────────────────────────────────────────────────┤
  │ Staff      │ StaffViews.js       │ Foundations exercise view only                                   │
  ├────────────┼─────────────────────┼──────────────────────────────────────────────────────────────────┤
  │ Student    │ StudentViews.js     │ Student dashboard, calendar, goals, assessment, proposals        │
  └────────────┴─────────────────────┴──────────────────────────────────────────────────────────────────┘

  ▎ Key idea: There is no single routing file. The router is split by role — each user type gets an entirely different set of pages.

  ---
  🧠 Django API

  Startup file (dev): learn-ops-api/manage.py
  Django's built-in CLI — used to start the dev server, run migrations, open a shell.

  Startup file (prod/Docker): learn-ops-api/LearningPlatform/wsgi.py
  The real entry point when running inside Docker via Gunicorn. entrypoint.sh calls this after migrations run.

  URL config: learn-ops-api/LearningPlatform/urls.py
  All API endpoints live here. Two styles:

  Router (auto-generates CRUD for ~30 resources):
    /students       /cohorts       /courses
    /books          /projects      /assessments  ... and more

  Manual paths (special cases):
    /auth/github            ← GitHub OAuth entry
    /auth/github/callback   ← GitHub redirects back here
    /admin                  ← Django admin UI
    /logs/                  ← In-app log viewer
    /metrics/               ← Prometheus metrics
    /__debug__/             ← Debug toolbar (DEBUG=True only)

  ---
  🦋 Monarch

  Startup file: service-monarch/service/main.py
  Three lines that start everything:
  migrator = TicketMigrator()
  asyncio.run(migrator.run())  # ← runs forever, listening on Valkey

  No HTTP routes — Monarch doesn't serve web pages. Instead of a URL config, it has a channel subscription:
  Listens on Valkey channel: channel_migrate_issue_tickets
  Receives: { source_repo, all_target_repositories, notification_channel }
  Does:      Migrate issues → notify Slack

  The real logic lives in service/core/monarch.py inside the TicketMigrator class.

  ---
  🗄️  PostgreSQL / 📨 Valkey / 📊 Prometheus / 📈 Grafana

  These have no startup code you write — they're off-the-shelf Docker images. Their "entry points" are their Docker image defaults, configured via environment variables or config
  files (prometheus.yml for Prometheus, .env for the rest).

  ▎ The only code that touches Postgres directly is Django's ORM via the migration files in learn-ops-api/LearningAPI/migrations/.

## 2. Services

  ---
  🧩 Services — Full Reference

  ---
  1. ⚛️  React Frontend

  Type: Client-side Web App

  Specific job: Renders the UI. Has zero business logic — it's purely a display layer that asks the Django API for data and shows it.

  How it's reached: Open a browser → http://localhost:3000

  How it's instantiated:
  Docker builds it from learn-ops-client/Dockerfile using Node 22. The container runs the React dev server which serves the app to the browser.

  Command that starts it:
  # Via Docker (normal way)
  make up

  # What Docker actually runs inside the container
  npm start

  ---
  2. 🧠 Django API

  Type: REST API / Web Server

  Specific job: The brain of the entire system. Handles authentication, all business logic, database reads/writes, GitHub and Slack API calls, and publishing jobs to Valkey.

  How it's reached:
  - API calls → http://localhost:8000
  - Admin UI → http://localhost:8000/admin
  - Logs → http://localhost:8000/logs
  - Metrics → http://localhost:8000/metrics/metrics

  How it's instantiated:
  Docker builds it from learn-ops-api/Dockerfile using Python 3.11. Before Django starts, entrypoint.sh runs automatically — it waits for Postgres, runs migrations, seeds the
  database, then hands off to the server.

  Command that starts it:
  # Via Docker (normal way)
  make up

  # What entrypoint.sh runs inside the container
  python manage.py runserver 0.0.0.0:8000

  # In debug mode (DEBUG=True) it wraps with debugpy first
  python -m debugpy --listen 0.0.0.0:5678 manage.py runserver 0.0.0.0:8000

  ---
  3. 🗄️  PostgreSQL

  Type: Relational Database

  Specific job: Stores all permanent data — every user, cohort, student, course, project, assessment, and note. Nothing is persisted anywhere else.

  How it's reached:
  - Django connects internally on database:5432 (Docker network name)
  - From your machine: localhost:5432 (with a DB client like TablePlus or psql)
  - No browser UI

  How it's instantiated:
  Pulled as a pre-built postgres:16 Docker image. On first boot it reads POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD from the infra .env and creates the database automatically.

  Command that starts it:
  # Via Docker (normal way — starts as part of the stack)
  make up

  ---
  4. 📨 Valkey

  Type: Message Broker (Redis-compatible)

  Specific job: Acts as the middleman between the Django API and Monarch. API drops a migration job here; Monarch picks it up. They never talk to each other directly.

  How it's reached:
  - Internally on port 6379 (Docker network)
  - No browser UI — CLI only
  - Channel it uses: channel_migrate_issue_tickets

  How it's instantiated:
  Pulled as a pre-built valkey/valkey Docker image. Started from its own separate compose file inside the infra repo.

  Command that starts it:
  # Must be started separately, before the main stack
  cd ~/workspace/lms/learn-ops-infrastructure/valkey
  docker compose up -d

  ---
  5. 🦋 Monarch

  Type: Async Microservice (Python)

  Specific job: A single-purpose service that migrates GitHub issue tickets from a template repo into every student team's repository. Notifies Slack when done or when it fails.

  How it's reached:
  - It isn't called directly — it listens on Valkey channel channel_migrate_issue_tickets
  - Metrics → http://localhost:8080
  - Log viewer → http://localhost:8081
  - Health check → http://localhost:8081/health

  How it's instantiated:
  Docker builds it from service-monarch/Dockerfile using Python 3.11-slim. Entry point is service/main.py which creates a TicketMigrator and runs it in an async loop — stays running
   forever, waiting for messages.

  Command that starts it:
  # Via Docker (from its own repo)
  cd ~/workspace/lms/service-monarch
  docker compose up -d

  # What Docker runs inside the container
  python service/main.py

  ---
  6. 📈 Prometheus

  Type: Metrics Collector / Time-Series Database

  Specific job: Every 15 seconds it polls the Django API and Postgres Exporter, collecting performance numbers — request counts, error rates, response times, DB connections.

  How it's reached: http://localhost:9090

  How it's instantiated:
  Pulled as a pre-built prom/prometheus Docker image. Configured entirely by learn-ops-infrastructure/prometheus.yml.

  Command that starts it:
  make up

  ---
  7. 🔌 Postgres Exporter

  Type: Metrics Adapter

  Specific job: Translates internal Postgres stats into a format Prometheus understands. Without it, Prometheus can't see anything about the database.

  How it's reached: http://localhost:9187/metrics (Prometheus hits this, not you)

  Command that starts it:
  make up

  ---
  8. 📊 Grafana

  Type: Visualization Dashboard

  Specific job: Reads from Prometheus and draws charts. Has no data of its own — purely a viewer.

  How it's reached: http://localhost:3001

  Command that starts it:
  make up

  ---
  🗂️  Summary Table

  ┌───────────────────┬────────────────────┬──────────────────────────────────┬────────────────────────────┬──────────────────────────┐
  │      Service      │        Type        │            Started By            │       Start Command        │        Reached At        │
  ├───────────────────┼────────────────────┼──────────────────────────────────┼────────────────────────────┼──────────────────────────┤
  │ React Frontend    │ Web App            │ make up                          │ npm start                  │ :3000                    │
  ├───────────────────┼────────────────────┼──────────────────────────────────┼────────────────────────────┼──────────────────────────┤
  │ Django API        │ REST API           │ make up                          │ python manage.py runserver │ :8000                    │
  ├───────────────────┼────────────────────┼──────────────────────────────────┼────────────────────────────┼──────────────────────────┤
  │ PostgreSQL        │ Database           │ make up                          │ Docker image default       │ :5432 (no UI)            │
  ├───────────────────┼────────────────────┼──────────────────────────────────┼────────────────────────────┼──────────────────────────┤
  │ Valkey            │ Message Broker     │ manual — /valkey folder          │ Docker image default       │ :6379 (no UI)            │
  ├───────────────────┼────────────────────┼──────────────────────────────────┼────────────────────────────┼──────────────────────────┤
  │ Monarch           │ Async Microservice │ manual — /service-monarch folder │ python service/main.py     │ :8080 metrics :8081 logs │
  ├───────────────────┼────────────────────┼──────────────────────────────────┼────────────────────────────┼──────────────────────────┤
  │ Prometheus        │ Metrics Collector  │ make up                          │ Docker image default       │ :9090                    │
  ├───────────────────┼────────────────────┼──────────────────────────────────┼────────────────────────────┼──────────────────────────┤
  │ Postgres Exporter │ Metrics Adapter    │ make up                          │ Docker image default       │ :9187 (no UI)            │
  ├───────────────────┼────────────────────┼──────────────────────────────────┼────────────────────────────┼──────────────────────────┤
  │ Grafana           │ Dashboard          │ make up                          │ Docker image default       │ :3001                    │
  └───────────────────┴────────────────────┴──────────────────────────────────┴────────────────────────────┴──────────────────────────┘

  ---
  🛠️  Tech Stack Table

  ┌────────────────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────┬───────────────────────────────────────────────┐
  │  Service Name  │                                                 Tech Stack                                                 │                    Purpose                    │
  ├────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────┤
  │ React Frontend │ React 16.13.1 · React Router 5.2.0 · Radix UI 1.x · Chart.js 4.4.1 · Node 22.13.0 · Tailwind CSS 3.3       │ Renders the UI — 3 role-based views           │
  │                │                                                                                                            │ (instructor, staff, student)                  │
  ├────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────┤
  │ Django API     │ Python 3.11.11 · Django · Django REST Framework · dj-rest-auth 4.0.1 · django-allauth 0.54.0 · structlog   │ Core backend — auth, business logic,          │
  │                │ 23.1.0 · gunicorn · psycopg2 · django-prometheus                                                           │ database, external API calls                  │
  ├────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────┤
  │ PostgreSQL     │ postgres:16 Docker image                                                                                   │ Relational database — all permanent data      │
  │                │                                                                                                            │ storage                                       │
  ├────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────┤
  │ Valkey         │ valkey/valkey:latest Docker image (Redis-compatible)                                                       │ Pub/sub message broker — decouples the API    │
  │                │                                                                                                            │ from Monarch                                  │
  ├────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────┤
  │ Monarch        │ Python 3.11-slim · asyncio · Pydantic 2.10.4 · Flask 3.0.3 · structlog 24.4.0 · tenacity 9.0.0 ·           │ Async microservice — migrates GitHub issues   │
  │                │ prometheus-client 0.21.1 · valkey 6.0.2                                                                    │ to student repos                              │
  ├────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────┤
  │ Prometheus     │ prom/prometheus:latest Docker image                                                                        │ Scrapes and stores metrics as time-series     │
  │                │                                                                                                            │ data                                          │
  ├────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────┤
  │ Postgres       │ postgres-exporter:latest Docker image                                                                      │ Bridges Postgres internal stats to Prometheus │
  │ Exporter       │                                                                                                            │  format                                       │
  ├────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────┤
  │ Grafana        │ grafana/grafana:latest Docker image                                                                        │ Visualizes Prometheus metrics as dashboards   │
  │                │                                                                                                            │ and charts                                    │
  └────────────────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────┴───────────────────────────────────────────────┘

  ---

  ▎ Rule: Run make up from learn-ops-infrastructure/ for most things. Start Valkey first, and Monarch separately only when you need it.


## 3. System Overview

⏺ 🗺️  System Overview — Learning Platform (LMS)

  ---
  What Is This App?

  A Learning Management System built for Nashville Software School. Instructors use it to track student progress, manage cohorts (classes), build courses, and automate GitHub
  workflows. Students use it to see their own progress and learning goals.

  ---
  The Map

                          👨‍💻 USER (Browser)
                                │
                                ▼
                      ┌─────────────────┐
                      │  React Frontend  │  :3000
                      │   (3 role UIs)   │
                      └────────┬────────┘
                               │ HTTP / JSON
                               ▼
                      ┌─────────────────┐
                ┌────▶│   Django API     │◀────┐
                │     │                 │     │
                │     └──┬──────────┬───┘     │
                │        │          │         │
                │        ▼          ▼         │
                │   ┌─────────┐  ┌────────┐   │
                │   │Postgres │  │ Valkey │   │
                │   │  :5432  │  │  :6379 │   │
                │   └─────────┘  └───┬────┘   │
                │                    │        │
                │                    ▼        │
                │           ┌─────────────┐   │
                │           │   Monarch   │   │
                │           │  :8080/8081 │   │
                │           └──────┬──────┘   │
                │                  │          │
                ▼                  ▼          ▼
           ┌─────────┐       ┌──────────┐  ┌───────┐
           │ GitHub  │       │  GitHub  │  │ Slack │
           │  OAuth  │       │   API    │  │  API  │
           └─────────┘       └──────────┘  └───────┘

           📈 Observability Stack (runs alongside)
           Postgres Exporter :9187
                 │
                 ▼
           Prometheus :9090 ──▶ Grafana :3001
           (scrapes API + DB)     (dashboards)

  ---
  How a Request Flows — Login to Dashboard

  1. User hits localhost:3000
  2. React checks: am I logged in? → No → redirect to /login
  3. User clicks "Login with GitHub"
  4. Django redirects to GitHub OAuth page
  5. GitHub authenticates user → sends them back to /auth/github
  6. Django creates a token, stores it in the browser
  7. React now has a token → loads the dashboard
  8. Every page = React calls Django API → Django queries Postgres → returns JSON → React renders it

  ---
  🗝️  Key Takeaways

  ┌─────┬────────────────────────────────────────────────────────────────────────────────────────────┬───────────────────────────────────────────────────────────────────────────┐
  │  #  │                                          Takeaway                                          │                              Why It Matters                               │
  ├─────┼────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
  │ 1   │ One command starts most things — make up from learn-ops-infrastructure/                    │ You never need to start services one by one                               │
  ├─────┼────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
  │ 2   │ Start Valkey first, separately — it lives in its own /valkey folder                        │ Without it, the API crashes when it tries to publish messages             │
  ├─────┼────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
  │ 3   │ Secrets live only in .env files — never hardcoded, never committed to git                  │ If .env is missing or wrong, nothing connects                             │
  ├─────┼────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
  │ 4   │ Three repos, one app — API, Client, and Monarch are separate git repos                     │ Each has its own .env, Dockerfile, and can be deployed independently      │
  ├─────┼────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
  │ 5   │ entrypoint.sh bootstraps the DB automatically — runs migrations and seeds data on every    │ You never run migrate manually in development                             │
  │     │ start                                                                                      │                                                                           │
  ├─────┼────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
  │ 6   │ Login is GitHub only — no username/password anywhere                                       │ LEARN_OPS_CLIENT_ID and LEARN_OPS_SECRET_KEY in the API .env must be      │
  │     │                                                                                            │ correct                                                                   │
  ├─────┼────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
  │ 7   │ Monarch is optional day-to-day — only needed for GitHub issue migration feature            │ Start it only when working on that specific workflow                      │
  ├─────┼────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
  │ 8   │ The React app has 3 completely different UIs — instructor, staff, student                  │ Role is set on the user's profile in the database                         │
  ├─────┼────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
  │ 9   │ Postgres and Valkey have no browser UI — interact via CLI or through the app               │ Use make logs or TablePlus/psql to inspect them                           │
  ├─────┼────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
  │ 10  │ Prometheus + Grafana are read-only observers — they change nothing in the app              │ Safe to ignore on day 1; useful when debugging performance later          │
  └─────┴────────────────────────────────────────────────────────────────────────────────────────────┴───────────────────────────────────────────────────────────────────────────┘
