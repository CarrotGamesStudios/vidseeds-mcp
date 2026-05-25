# VidSeeds.ai connector for Claude Code & Codex

Drive **VidSeeds.ai** — pre-upload video **SEO & metadata optimization** and
multi-platform publishing — directly from your AI coding client.

> **VidSeeds.ai** analyzes a creator's existing video and produces optimized titles,
> descriptions, tags, thumbnails, and chapters for YouTube, TikTok, Instagram, Facebook,
> LinkedIn, and X, then publishes them. It is **not** a video generator or editor.

This package is an [MCP](https://modelcontextprotocol.io) connector that exposes the
VidSeeds.ai workflow (**139 tools**, all prefixed `vidseeds.`) as a plugin for
**Claude Code** and **Codex**. The connector ships no credentials — it tells your client
how to call the hosted endpoint `https://vidseeds.ai/api/mcp` with a token you supply.

- **Endpoint:** `https://vidseeds.ai/api/mcp` (MCP Streamable HTTP)
- **Auth:** Personal Access Token only (`Authorization: Bearer vs_pat_…`)
- **Server:** `vidseeds` v1.6.0

---

## 1. Get a Personal Access Token (required)

1. Sign in at <https://vidseeds.ai> (free account).
2. Open **Settings → Developer Tokens**: <https://vidseeds.ai/settings/developer-tokens>.
3. Create a token and copy the secret (`vs_pat_…`) — it is shown **only once**
   (90-day default expiry).
4. Expose it to your client as the `VIDSEEDS_PAT` environment variable (below).

> 🔒 The token value never belongs in any file. This package references it **by name**
> (`VIDSEEDS_PAT`); your client injects the value from the environment at connect time.
> A leaked PAT grants full non-admin access to your account — treat it like a password.

> ℹ️ Creating a token is free. **Tool access** (auth vs entitlement are separate) may
> depend on your VidSeeds.ai plan or trial — the server enforces entitlements per call.

---

## 2. Install in Claude Code

### Option A — install the plugin (recommended)

```text
/plugin marketplace add CarrotGamesStudios/vidseeds-mcp
/plugin install vidseeds@vidseeds
```

Then make the token available in the environment Claude Code is launched from:

```bash
export VIDSEEDS_PAT="vs_pat_your_token_here"
claude
```

Run `/mcp` to confirm the `vidseeds` server is connected and listing tools. If you see an
auth error, `VIDSEEDS_PAT` was not set in the launching shell — export it and restart.

> The plugin's `.mcp.json` injects your token via `Bearer ${VIDSEEDS_PAT}` — the same
> shape Anthropic's official GitHub connector uses. Set `VIDSEEDS_PAT` before launching;
> if it is unset, Claude Code flags the `vidseeds` server as needing configuration.

### Option B — add the server directly (no plugin)

For a quick test without installing the plugin. Prefer Option A for daily use — it
keeps the token in the environment (`VIDSEEDS_PAT`) and reads it at runtime, instead
of writing the resolved token into `~/.claude.json`.

Reference the env var rather than pasting the raw token, so the secret never lands in
your shell history or terminal scrollback:

```bash
export VIDSEEDS_PAT="vs_pat_your_token_here"
claude mcp add --transport http vidseeds https://vidseeds.ai/api/mcp \
  --header "Authorization: Bearer $VIDSEEDS_PAT"
```

---

## 3. Install in Codex

Codex does not yet offer self-serve central plugin publishing, so the most reliable
path is to add the server to your Codex config.

### Option A — `~/.codex/config.toml` (recommended)

```toml
[mcp_servers.vidseeds]
url = "https://vidseeds.ai/api/mcp"
bearer_token_env_var = "VIDSEEDS_PAT"
```

Then export the token in the environment Codex runs in:

```bash
export VIDSEEDS_PAT="vs_pat_your_token_here"
```

### Option B — CLI

```bash
codex mcp add vidseeds --url https://vidseeds.ai/api/mcp --bearer-token-env-var VIDSEEDS_PAT
```

### Option C — repo/team marketplace

This repo also ships a Codex repo marketplace at `.agents/plugins/marketplace.json` and a
plugin manifest at `.codex-plugin/plugin.json` (MCP config in `codex.mcp.json`). Point a
Codex marketplace at this repo and install `vidseeds` via `codex` → `/plugins`. The same
`VIDSEEDS_PAT` environment variable applies.

---

## 4. What the connector can do

139 tools spanning the full VidSeeds.ai creator workflow:

| Area                     | Examples                                                                                                                    |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| Metadata optimization    | titles, descriptions, tags, chapters; optimization history                                                                  |
| Thumbnails               | AI generation, edit, restyle; Thumbnail Studio (trends, briefs, subjects, overlay text, style profiles); publish to YouTube |
| Projects                 | create from a YouTube video, snapshots, apply thumbnail, per-platform config                                                |
| Analytics & intelligence | channel/video analytics, channel intelligence, video autopsy, outlier detection, public-channel audit                       |
| Discovery & research     | keyword research, trending videos, breakout channels, video ideas                                                           |
| Auto-clips               | list/get saved shorts, edit metadata, caption style, reframe, download                                                      |
| Translation & captions   | metadata localization (85 languages), caption upload/get/delete                                                             |
| Local video media        | `probe`, frame/clip extraction, precision-trim & midroll analysis (commands run locally)                                    |
| Publishing               | platform connections, schedule/confirm publish, direct YouTube upload                                                       |
| Account                  | seed balance, subscription, referrals, support                                                                              |

> **Seeds:** some tools (thumbnail generation, AI intelligence, translation, …) spend
> seeds from your VidSeeds.ai balance. Read-only tools are free. Every tool's description
> states its cost; check `vidseeds.get_seed_balance` if unsure.

---

## Security

- This package contains **no credentials**. It declares how Claude Code / Codex should
  call `https://vidseeds.ai/api/mcp` with a user-supplied PAT read from `VIDSEEDS_PAT`.
- Never commit, paste, or share a `vs_pat_…` token. Rotate or revoke tokens at
  <https://vidseeds.ai/settings/developer-tokens>.
- Cookie/session auth is rejected by the server by design — a PAT is the only accepted
  credential, so a misconfigured client can't piggyback on a dashboard login.

## Versioning

The plugin version tracks the VidSeeds.ai MCP server version (currently **1.6.0**).
The wildcard PAT scope reaches new tools automatically as the server grows, so existing
tokens keep working without being recreated.

## Links

- Product: <https://vidseeds.ai>
- Developer tokens: <https://vidseeds.ai/settings/developer-tokens>
- MCP: <https://modelcontextprotocol.io>

## License

MIT — see [`LICENSE`](./LICENSE).
