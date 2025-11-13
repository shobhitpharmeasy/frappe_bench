# Personal Frappe Site Bench

This repository captures my personal site setup built on top of a Frappe bench.  
It packages the apps, configuration, and developer tooling I use to run a single bench that powers my site and a handful of companion apps (Helpdesk, Telephony, and Bench Manager).

## Stack Snapshot

- Frappe `15.88.0`
- Helpdesk `1.17.3`
- Telephony app (develop branch)
- Bench Manager app (develop branch)
- Python `3.12.12`, Node.js `18.20.8`, Yarn `1.x`
- MariaDB and Redis services (local installs via Homebrew)

> Versions are tracked in `sites/apps.json`. Update that file when you bump app versions or switch branches so the README stays accurate.

## Prerequisites

- macOS (tested on Sequoia 15.x / Darwin 25.1.0)
- Homebrew-installed services:
  - `brew install mariadb redis`
  - start them with `brew services start mariadb redis`
- `pipx install frappe-bench` (or install Bench globally through the virtualenv if you prefer)
- Optional but recommended: [`mise`](https://mise.jdx.dev/) to pin Python/Node versions locally (`mise install python@3.12.12 node@18.20.8`)

## First-Time Setup

```bash
git clone git@github.com:<your-handle>/frappe_bench.git
cd frappe_bench

# Re-create the virtual environment (the committed env/ folder is for reference only)
python3 -m venv env
source env/bin/activate
pip install -U pip

# Pull dependencies for every installed app
bench setup requirements --dev

# Install JS dependencies for the front-end apps
bench setup requirements --node

# Build assets once so the site loads correctly
bench build
```

If you are provisioning a fresh database, create a site and restore your content:

```bash
bench new-site my-site.local
# or
bench --site my-site.local restore /path/to/backup.sql.gz
```

Copy `sites/common_site_config.json` and any `site_config.json` files you need (scrub secrets if you are sharing the repo). Bench will prompt you for DB passwords when creating new sites—never commit real credentials.

## Daily Development Workflow

- **Start everything**: `bench start` (uses the bundled `Procfile` for web, socketio, workers, scheduler, and file watcher).
- **Single app focus**: `bench --site helpdesk.test serve --port 8000`
- **Watch assets**: `bench watch`
- **Run Python tests**: `bench run-tests --app helpdesk`
- **Run JS tests**: `cd apps/helpdesk/desk && yarn test`
- **Update apps**: `bench update --reset` (pulls latest from tracked branches)

Logs stream into the `logs/` directory (`web`, `scheduler`, `worker`, etc.). Tail them with `tail -f logs/worker.log`.

## Repository Layout Highlights

- `apps/` – all Frappe apps installed in this bench (`frappe`, `helpdesk`, `telephony`, `bench_manager`)
- `sites/apps.json` – authoritative list of apps + pinned versions
- `sites/common_site_config.json` – bench-level settings (ports, redis locations, default site)
- `sites/<site-name>/` – site-specific assets, public files, and private data
- `Procfile` – process definitions for using `foreman`, `honcho`, or the `bench start` command
- `logs/` – worker, scheduler, and web server logs (Git-ignored)

## Deployment Notes

- Production builds should run `bench build --production` and serve assets via nginx.
- Use `bench setup production <user>` to wire supervisor + nginx if you plan to deploy directly off this bench.
- Keep Redis and MariaDB credentials outside the repo; configure them via environment variables or separate config management.
- Snapshot the database regularly (`bench backup --with-files`) and verify restores on a staging site.

## Troubleshooting Cheatsheet

- **PIP package mismatch**: run `bench setup requirements --dev` after switching branches.
- **Node build errors**: ensure `node -v` reports `18.20.8` (reinstall via `mise` or `nvm` if needed).
- **Redis refusing connections**: confirm `redis-server` is running and matches the ports in `common_site_config.json`.
- **MariaDB auth issues**: reset the site DB password with `bench --site <site> set-config db_password <new-pass>`.

## Housekeeping

- Secrets in `sites/*/site_config.json` are for local development; clear them before pushing to a public repo.
- Use `bench update --patch` after pulling new migrations.
- Pin new app versions in `sites/apps.json` to keep the stack documented.
- Commit changes from within the repo root so relative paths resolve correctly.

---

If you add new apps or upgrade the stack, update this README so future you knows the exact steps to get everything running again.

