# Apify for Cursor

Official Apify plugin for Cursor — adds the Apify MCP server, one `apify` routing agent, a routing rule that funnels Apify requests through that agent, and five bundled skills for the main Apify workflows: using existing Actors, building and deploying custom Actors, actorizing existing projects, generating Actor output schemas, and integrating Apify into existing applications.

> **Apify** is a platform of thousands of serverless cloud programs called **Actors** for web scraping, browser automation, and data extraction. Learn more at [apify.com](https://apify.com).

## What you get

| Component | Path | Purpose |
|---|---|---|
| Agent (entry point) | `apify/agents/apify.md` | Routes every Apify request to the right skill or transport path. **This is the one you should invoke.** |
| Routing rule | `apify/rules/apify-routing.mdc` | Tells Cursor to hand Apify-related requests to the `apify` agent first, instead of guessing between the per-task skills. |
| MCP server | `apify/mcp.json` → `apify` (`https://mcp.apify.com`) | Lets the agent search the Apify Store, fetch Actor details, run Actors, and read the Apify docs. |
| Skill | `apify/skills/apify-actor-development/` | Create, debug, and deploy a brand new Apify Actor from scratch. |
| Skill | `apify/skills/apify-actorization/` | Convert an existing JS/TS, Python, or CLI project into an Apify Actor. |
| Skill | `apify/skills/apify-generate-output-schema/` | Generate `dataset_schema.json` / `output_schema.json` / `key_value_store_schema.json` for an existing Actor. |
| Skill | `apify/skills/apify-sdk-integration/` | Add Apify Actor execution to an existing application using the `apify-client` package. |
| Skill | `apify/skills/apify-ultimate-scraper/` | Pick the right Actor from ~100 pre-built scrapers across 15+ platforms and run them end-to-end via the Apify CLI. |
| Asset | `assets/logo.svg` | Plugin branding asset used by the Cursor package. |

## Installation

### From the Cursor Marketplace

1. Open the **Marketplace** panel in Cursor.
2. Search for **Apify** (the plugin id is `apify`).
3. Click **Install**.
4. Reload the window when prompted.

### Manual / local install (for testing)

```bash
git clone https://github.com/apify/apify-cursor-plugin ~/.cursor/plugins/local/apify
# Then in Cursor: View → Command Palette → "Developer: Reload Window"
```

## First-run setup

The plugin uses **two different setup paths** depending on what you're doing. The `apify` agent will guide you through whichever one is needed, but here is the high-level map.

### Path 1 — Using existing Actors (MCP)

Uses **OAuth**. The first time the agent calls a tool that needs auth (e.g. `run-actor`), Cursor opens `console.apify.com` in your browser and asks you to sign in. Read-only tools (`search-actors`, `fetch-actor-details`, `search-apify-docs`, `fetch-apify-docs`) work without auth.

### Path 2 — Local Actor workflows and SDK integration

This path covers the bundled skills that do not rely on MCP:

- `apify-actor-development`, `apify-actorization`, and most `apify-ultimate-scraper` workflows use the **Apify CLI**.
- `apify-sdk-integration` uses an **`APIFY_TOKEN`** with the `apify-client` package or the REST API.
- `apify-generate-output-schema` works against the Actor files in your workspace and may use `apify run` later only as a validation step.

Install the CLI once if you're doing local Actor development or CLI-driven scraping:

```bash
npm install -g apify-cli
apify login          # interactive OAuth, or…
export APIFY_TOKEN="apify_api_xxxxxxxxxxxx"
```

Generate a token at [console.apify.com/settings/integrations](https://console.apify.com/settings/integrations). Don't have an account? [Sign up free](https://console.apify.com/sign-up) — no credit card required.

In practice:

- `apify-actor-development` and `apify-actorization` rely on the CLI for commands such as `apify create`, `apify init`, `apify run`, and `apify push`.
- `apify-sdk-integration` only needs `APIFY_TOKEN` and the `apify-client` npm package.
- `apify-ultimate-scraper` drives CLI commands such as `apify actors search`, `apify actors call`, and `apify datasets get-items`.
- `apify-generate-output-schema` analyzes the local Actor codebase first, then can optionally be followed by local validation.

### Working in remote sessions, devcontainers, or SSH (no browser)

The MCP OAuth flow needs a browser. If you're running Cursor over SSH, in a devcontainer, or in any environment where Cursor cannot open a browser, you have two options:

1. **Authenticate locally first.** Connect the Apify MCP server once on your laptop with a normal Cursor session so the OAuth refresh token is stored in your Cursor profile, then reconnect remotely.
2. **Skip MCP and use the non-MCP workflows.** The bundled skills can still work without MCP, but the transport depends on the task: CLI for Actor development, actorization, and scraper runs; `apify-client` plus `APIFY_TOKEN` for app integrations; and local Actor files for schema generation. Tell the agent to "use the Apify CLI" when you want the CLI path explicitly.

## How to use it

Almost every interaction should start with `@apify`. The agent reads the request, decides which route applies, and dispatches to the right skill or MCP tool.

```
@apify find me 5 well-rated coffee shops in Seattle and export to CSV
@apify build me an Actor that scrapes a sitemap and stores titles
@apify add Apify to this Next.js app so I can run a scraper from /api/scrape
@apify generate output schemas for the Actor in this folder
```

### About the slash menu

Cursor currently lists every skill in the slash menu, so you may also see `/apify-actor-development`, `/apify-ultimate-scraper`, etc. Please **prefer `@apify`** — the routing rule will redirect you back through the agent anyway, and the agent owns critical guardrails (such as the `apify` vs `apify-client` package name trap that silently breaks projects when picked wrong).

## Components reference

### MCP server

The `apify` MCP server is configured in `apify/mcp.json` (`https://mcp.apify.com`) and exposes:

- `search-actors` — search the Apify Store by keyword (no auth)
- `fetch-actor-details` — Actor specs, input schema, pricing (no auth)
- `run-actor` — execute an Actor and return results (OAuth)
- `get-dataset-items` — retrieve dataset rows from a previous run (OAuth)
- `search-apify-docs` / `fetch-apify-docs` — Apify documentation lookup

You can disable or re-enable the server in **Cursor Settings → MCP**.

### Skill references

Some skills ship with extra reference docs and workflow playbooks that the agent loads on demand:

- `apify/skills/apify-actor-development/references/` — `actor-json.md`, `input-schema.md`, `dataset-schema.md`, `key-value-store-schema.md`, `output-schema.md`, `actor-readme.md`, `logging.md`, `standby-mode.md`.
- `apify/skills/apify-actorization/references/` — `js-ts-actorization.md`, `python-actorization.md`, `cli-actorization.md`, `schemas-and-output.md`.
- `apify/skills/apify-ultimate-scraper/references/` — `actor-index.md`, `gotchas.md`, and a `workflows/` folder covering lead generation, competitive intel, influencer vetting, brand monitoring, review analysis, content & SEO, social analytics, trend research, recruitment, real estate, e-commerce price monitoring, contact enrichment, RAG, and company research.

The `apify-ultimate-scraper` skill drives the **Apify CLI** directly (`apify actors search`, `apify actors call`, `apify datasets get-items`) — there are no bundled helper scripts to install. Make sure `apify-cli` v1.4.0+ is on your `PATH`.

## Troubleshooting

**OAuth browser never opens / hangs.** See the "Working in remote sessions" section above.

**`apify: command not found` from the actor-development, actorization, or ultimate-scraper skills.** Install the Apify CLI: `npm install -g apify-cli` (requires Node.js 20.6+). Then run `apify login` or export `APIFY_TOKEN`.

**Authentication errors from CLI commands.** Run `apify login` to refresh OAuth, or set `APIFY_TOKEN` in your shell or project `.env`. Generate a token at [console.apify.com/settings/integrations](https://console.apify.com/settings/integrations).

**The wrong skill keeps getting picked.** That's exactly the problem the `apify` routing rule and agent are designed to prevent — make sure you're starting with `@apify`, not `/apify-<task>`.

**`apify` vs `apify-client`** — these are two different npm packages. The `apify` package is the SDK for **building** Actors (used inside an Actor's code, on the Apify platform). The `apify-client` package is the API client for **calling** Actors from your own application. The agent picks the right one for you; if you're installing manually, double-check.

**MCP server appears disconnected after Cursor restart.** Open **Cursor Settings → MCP** and toggle the `apify` server off and on. If that doesn't help, re-trigger OAuth by running any Actor command.

## Resources

- Apify Console — [console.apify.com](https://console.apify.com)
- Apify Store — [apify.com/store](https://apify.com/store)
- Docs (LLM-friendly) — [docs.apify.com/llms.txt](https://docs.apify.com/llms.txt)
- Docs (full) — [docs.apify.com/llms-full.txt](https://docs.apify.com/llms-full.txt)
- Source repo — [github.com/apify/apify-cursor-plugin](https://github.com/apify/apify-cursor-plugin)
- Issues / feedback — open an issue on the source repo, or email [support@apify.com](mailto:support@apify.com)

## License

Apache-2.0. See [LICENSE](./LICENSE).
