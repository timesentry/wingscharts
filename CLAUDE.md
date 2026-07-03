# CLAUDE.md

This file provides guidance to Claude Code when working with wingscharts.

## Project Overview

wingscharts is TimeSentry's fork of the **Highcharts export server** (`highcharts/node-export-server`, v5.x) — a Node.js service that renders Highcharts configs to static images (PNG/JPEG/SVG/PDF) via a Puppeteer worker pool. wingsoftime consumes it as a **containerized HTTP sidecar** (port 7801) for chart exports in reports; it is not consumed as an npm package.

**Branch model: the default branch is `master`, not `main`.** CI runs unit tests on PRs to `master`; pushing to `stable` triggers the build-and-push workflow that force-commits `dist/`.

## Tech Stack

- Node.js >= 18 (Docker image: Node 22-slim + system Chromium, `PUPPETEER_SKIP_DOWNLOAD=true`)
- Express, Puppeteer (tarn worker pool, default 4–8 workers), Highcharts 12.x, jsdom + DOMPurify
- Rollup dual build (ESM + CJS) → `dist/`

## Commands

```bash
npm install            # runs install.js hook
npm start              # HTTP server mode (port 7801)
npm run start:dev      # nodemon watch mode, verbose logging
npm run build          # rollup bundle → dist/
npm run lint           # eslint --fix + prettier
npm run unit:test      # jest unit tests
npm run cli-tests / http-tests / node-tests   # integration suites
```

Config precedence: defaults (`lib/schemas/config.js`) < `.env` < custom JSON < CLI args. See `.env.sample`.

## How TimeSentry uses it

- Local dev: the `highcharts-export` service in `wingsoftime/docker-compose.yml` (profiles `local`/`127`/`tunnel`).
- Deployed: sidecar container on the `ts-api-*` / `ts-worker-*` Container Apps (see `wingsoftime/deploy/templates/`).
- Keep the fork's delta small: upstream is `highcharts/node-export-server`; our changes are Docker/deploy-oriented (system Chromium, CDN script cache prefetch at image build). Prefer rebasing on upstream over diverging.
- Requires a valid Highcharts license for use.

## TimeSentry domain gotchas (project-wide)

This service is domain-agnostic (it renders whatever chart config it's given), but if you touch anything that *labels or aggregates* TimeSentry data upstream of it: (1) every actor (person, firm, AI agent) is a `Company`, not a `User`; (2) time entries cross the company graph by **copying** (chains linked by `upstream_time_entry_id`, one `timekeeper_id` per chain) and must be chain-deduped when counted. Full write-up: monorepo root `CLAUDE.md` ("Domain model").
