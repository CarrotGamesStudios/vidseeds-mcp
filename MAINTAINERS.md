# Maintainers — VidSeeds.ai MCP connector package

This directory (`plugins/vidseeds-mcp/`) is the **source of truth** for the public marketplace repo [CarrotGamesStudios/vidseeds-mcp](https://github.com/CarrotGamesStudios/vidseeds-mcp). Mirror the whole tree to that repo on each connector release (see `docs/06-ai-integration/2026-05-24-mcp-connector-plugin-publishing.md` in the private monorepo).

## When to update this package

Any change that affects external MCP users:

| Area                     | Paths (private monorepo)                                                              |
| ------------------------ | ------------------------------------------------------------------------------------- |
| MCP tools                | `lib/mcp/tools/**`                                                                    |
| MCP auth / trial / quota | `lib/mcp/auth.ts`, `lib/mcp/entitlement.ts`, `lib/mcp/quota-tracking.ts`              |
| Local stdio MCP          | `scripts/mcp-local/**`                                                                |
| Connector version        | `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`, `server.json`, `README.md` |

## PR checklist

1. Update affected skills under [`skills/`](skills/) (workflows, tool names, quotas, errors).
2. Bump plugin `version` in `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json` when shipping a connector release.
3. Align **non-admin tool count** in `README.md` and plugin manifests with `bun run check:mcp-plugin-skills` (admin tools are omitted from public docs).
4. Keep root `server.json` aligned with the hosted endpoint, auth header, repository, and connector version; republish it with `mcp-publisher` when the public registry entry needs an update.
5. Run from monorepo root:
   ```bash
   bun run check:mcp-plugin-skills
   claude plugin validate --strict plugins/vidseeds-mcp/.claude-plugin/plugin.json
   claude plugin validate --strict plugins/vidseeds-mcp/.claude-plugin/marketplace.json
   bunx prettier --check "plugins/vidseeds-mcp/**/*.{json,md}"
   ```
6. On release: rsync this directory to the public repo and push (authorized maintainers only).

## Skills content boundary

Skills teach **what agents can achieve** and **efficient tool composition**. They must not expose enough detail to replicate VidSeeds internals.

### Include

- Ordered chains of stable `vidseeds_*` tool names
- User-visible errors: `AUTH_REQUIRED`, `SUBSCRIPTION_REQUIRED`, `MCP_QUOTA_EXCEEDED`, insufficient seeds
- Latency and polling guidance (e.g. thumbnail jobs ~70–100s, poll ~3s)
- Inputs agents need (`projectId`, `jobId`, `channelId` when multiple YouTube channels exist) — only what tool descriptions already state

### Exclude

- File paths (`lib/mcp/`, `lib/services/`, route handlers)
- AI provider or model routing, prompt templates, scoring algorithms
- Tariffication implementation, refund keys, internal job stores
- Copy-paste of private `docs/06-ai-integration/*` parity docs
- **All** `vidseeds_admin_*` tools and admin scopes

Per-tool seed costs and parameter detail live on the **hosted MCP server** (`tools/list` descriptions). Skills complement that with workflows, not a full tool catalog.

## Skill catalog

| Skill                  | Role                                         |
| ---------------------- | -------------------------------------------- |
| `vidseeds-setup`       | PAT/OAuth, trial, client install             |
| `vidseeds-efficiency`  | Quotas, seeds, async polling, anti-patterns  |
| `vidseeds-thumbnails`  | Thumbnail + Thumbnail Studio workflows       |
| `vidseeds-projects`    | Project create → metadata → thumbnail attach |
| `vidseeds-analytics`   | Research & intelligence tool map             |
| `vidseeds-local-video` | Client-first probe / frames / trim           |
| `vidseeds-publishing`  | Connections & publish flows                  |

See [`skills/README.md`](skills/README.md) for authoring format.
