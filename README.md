<p align="center"><img src="assets/logo.svg" alt="Admaxxer" width="220"></p>

# Admaxxer for AI agents

Admaxxer is ad account analytics and campaign control for AI agents: one workspace's Meta, Google, TikTok, Shopify and Klaviyo accounts, plus the attribution and web analytics the dashboard reads. The hosted MCP server at https://admaxxer.com/mcp exposes 17 tools (15 read, 2 write). A token is read-only unless minted with the Read + manage scope. Every write is two steps: a preview and confirmToken, then a second call after the person agrees. Anything created lands paused; there is no activate option. No tool deletes anything or changes account-level settings. Auth is a paste bearer from https://admaxxer.com/integrations/mcp or OAuth 2.1 with dynamic client registration. Starter prompts: "What did I spend on Meta last week and what was the ROAS?" "Which channel actually drove last week's revenue?" "Pause the Summer Sale campaign (I'll confirm)."

Three rules the server enforces, so no amount of prompting moves them:

- **Read-only by default.** A token reaches the write tools only when you mint
  it with the "Read + manage" scope. A read-only token gets
  `manage_scope_required` back.
- **Every write is two steps.** The first call returns a preview and a confirm
  token and changes nothing. The agent shows you the preview, you say yes, and
  only then does the second call execute.
- **Anything an agent creates lands paused.** There is no activate-on-create
  option over MCP, so nothing an agent builds can spend until you set it live
  in the dashboard.

Pause, resume, and daily budget at campaign, ad-set and ad level, plus
building new campaigns that land paused. No deletion. No account-level actions.

This repository ships the skill (`SKILL.md`, `skills/admaxxer/`) and the plugin
manifests for Claude Code, Cursor and Grok Build. The MCP server itself is
hosted at `https://admaxxer.com/mcp`.

## Get a token

Mint one at **https://admaxxer.com/integrations/mcp** (Settings → AI Providers
→ MCP). It is scoped to one workspace, you choose read-only or "Read + manage",
and you can revoke it there. Tokens start `mcp_ADM_`. Every command below
carries the same endpoint and a placeholder: replace
`mcp_ADM_YOUR_TOKEN_HERE`.

## Install

**Claude Code — the plugin brings the skill:**

```
/plugin marketplace add fortuneflick/admaxxer-claude-plugin
/plugin install admaxxer@admaxxer
```

### Cursor

```text
cursor://anysphere.cursor-deeplink/mcp/install?name=admaxxer&config=eyJ1cmwiOiJodHRwczovL2FkbWF4eGVyLmNvbS9tY3AiLCJoZWFkZXJzIjp7IkF1dGhvcml6YXRpb24iOiJCZWFyZXIgbWNwX0FETV9ZT1VSX1RPS0VOX0hFUkUifX0=
```

Open this link and Cursor offers to add the server. Replace the placeholder
token in Settings → MCP afterwards, or paste the same object into
`.cursor/mcp.json`.

### Claude Code

```bash
claude mcp add --transport http admaxxer https://admaxxer.com/mcp \
  --header "Authorization: Bearer mcp_ADM_YOUR_TOKEN_HERE"
```

Then `/mcp` inside Claude Code should list `admaxxer` as connected with 17
tools.

### Codex CLI

```bash
codex mcp add admaxxer --url https://admaxxer.com/mcp \
  --bearer-token-env-var ADMAXXER_MCP_TOKEN
```

The token stays in your environment; Codex writes only the variable name to
`~/.codex/config.toml`.

### Gemini CLI

```bash
gemini mcp add --transport http \
  --header "Authorization: Bearer mcp_ADM_YOUR_TOKEN_HERE" \
  admaxxer https://admaxxer.com/mcp
```

Check it with `gemini mcp list`.

### VS Code

```bash
code --add-mcp '{"name":"admaxxer","type":"http","url":"https://admaxxer.com/mcp","headers":{"Authorization":"Bearer mcp_ADM_YOUR_TOKEN_HERE"}}'
```

Or write the same server into `.vscode/mcp.json` under `servers`.

### Windsurf

```json
{
  "mcpServers": {
    "admaxxer": {
      "serverUrl": "https://admaxxer.com/mcp",
      "headers": {
        "Authorization": "Bearer mcp_ADM_YOUR_TOKEN_HERE"
      }
    }
  }
}
```

Goes in `~/.codeium/windsurf/mcp_config.json`, then refresh the MCP panel.

### OpenCode

```json
{
  "mcp": {
    "admaxxer": {
      "type": "remote",
      "url": "https://admaxxer.com/mcp",
      "enabled": true,
      "headers": {
        "Authorization": "Bearer {env:ADMAXXER_MCP_TOKEN}"
      }
    }
  }
}
```

Goes in `opencode.json`; the token is read from your environment rather than
written to the file.

### Clients that only speak HTTP+SSE

Use `https://admaxxer.com/mcp/sse` with the same `Authorization` header. It is
the older transport and carries the same tools.

### Skill only, any agent

```bash
npx skills add fortuneflick/admaxxer-claude-plugin
```

**Grok Build:** run `/marketplace` and pick Admaxxer, or add this repository as
a marketplace source. The Grok manifest (`.grok-plugin/plugin.json`) is the
only one that bundles the hosted server through its `mcpServers` field, so Grok
gets the tools and the skill in one install. The Claude Code and Cursor plugins
are skill-only on purpose: installing one never registers a second Admaxxer
server beside a connector you already have.

## What the agent can do

- List every connected Meta, Google, TikTok, Klaviyo and Shopify account and
  its health.
- Read campaigns, ad sets and ads with spend, ROAS, conversions, CTR, CPC and
  daily budget. A whole account comes back in one call, so drilling into a
  campaign costs no extra platform requests.
- Read the workspace's headline numbers for any window: revenue, ad spend,
  blended MER and ROAS, AOV, orders, sessions, visitors, conversion rate.
- Read channel-level revenue attribution under the model and lookback window
  you pick, plus web analytics, and which AI assistants cite your pages.
- Pause or resume a campaign, or change its daily budget, after you approve
  the preview.
- Build a campaign, ad set and ad, after you approve the plan. It lands paused.

Every tool, its inputs and the scope it needs are listed in
`skills/admaxxer/references/tools.md`, generated from the server's own registry
so it cannot drift. Client-by-client setup lives at
https://admaxxer.com/documentation/connect-any-ai.

## What the agent cannot do

- **It cannot widen its own token.** The scope is set when you mint the token
  and the agent has no tool that changes it.
- **It cannot skip your confirmation.** The confirm token comes from the
  server, only after a preview, and expires.
- **It cannot delete anything**, touch account-level settings, or set a created
  campaign live.
- **It cannot see another workspace.** The workspace is resolved from the
  token, never from a tool argument.

## Security

- **No executable code in the install path.** The skill is Markdown and the
  manifests are JSON. Nothing here runs, downloads a binary, or installs
  anything beyond copying those files into your agent.
- **One runtime endpoint:** `https://admaxxer.com/mcp` over HTTPS (plus the
  legacy SSE form at `/mcp/sse`). Every command above names that address; the
  tools run there.
- **Credentials:** a bearer token you mint and revoke at
  https://admaxxer.com/integrations/mcp. This repository contains no tokens and
  never asks for one in chat. The manifests read `ADMAXXER_MCP_TOKEN` from your
  environment rather than writing it to a config file.
- **Audit trail.** Every tool call that touches a connected ad account leaves a
  row you can read back in the dashboard.
- **No telemetry.** Nothing here phones home.

## Links

- Connect any AI agent, with the full tool catalog: https://admaxxer.com/documentation/connect-any-ai
- Mint a token: https://admaxxer.com/integrations/mcp
- Developer documentation: https://admaxxer.com/documentation/developer
- Pricing: https://admaxxer.com/pricing
- Support: hello@admaxxer.com

## License

MIT — see [LICENSE](LICENSE).
