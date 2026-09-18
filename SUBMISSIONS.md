# Catalog submissions — owner checklist

> **INTERNAL. NOT CUSTOMER-FACING.** A working checklist for the repository
> owner. It is safe to keep in the public repository (no secrets), but nothing
> here is product copy.

Order: **1. push this repo → 2. xAI → 3. Anthropic → 4. Cursor → 5. claude.ai
connectors directory → 6. ChatGPT plugins (last) → 7. MCP registry (needs
DNS).**

This repository is the single canonical source every catalog points at. The
platform repository is private; the generated half of this repository
(`SKILL.md`, `skills/**`) is written from the MCP tool registry by
`npm run gen:agent-skill -- --out ../admaxxer-claude-plugin` and pinned by the
`check-agent-skill-parity` canary in the platform's `npm run postbuild`.
**Never hand-edit `SKILL.md` or anything under `skills/`** — regenerate and
push. Manifests, README and this file are hand-maintained.

## 0. Before anything

- [x] Public repository created: https://github.com/fortuneflick/admaxxer-claude-plugin (MIT).
- [x] `claude plugin validate .` passes locally.
- [ ] Record the 40-char SHA after each push: `git ls-remote https://github.com/fortuneflick/admaxxer-claude-plugin.git HEAD`

## Live check — MCP OAuth (2026-09-18)

Both the claude.ai connectors directory and the ChatGPT review require working
OAuth discovery. Verified against production on 2026-09-18:

| Check | Result |
|---|---|
| `POST https://admaxxer.com/mcp` with no credentials | **401** |
| `WWW-Authenticate` on that 401 | `Bearer realm="admaxxer-mcp", resource_metadata="https://admaxxer.com/.well-known/oauth-protected-resource"` |
| `GET https://admaxxer.com/.well-known/oauth-protected-resource` | **200** — resource `https://admaxxer.com/mcp`, authorization server `https://admaxxer.com`, scopes `ads:read`, `analytics:read`, `ads:manage`, `bearer_methods_supported: ["header"]`, docs `https://admaxxer.com/documentation/developer` |

Re-run before any submission:

```bash
curl -si -X POST https://admaxxer.com/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | grep -i www-authenticate
curl -s https://admaxxer.com/.well-known/oauth-protected-resource
```

**Not verified:** a full end-to-end OAuth authorization-code flow (dynamic
client registration, consent screen, token exchange). Discovery resolves; the
day-to-day path customers use today is the bearer token minted at
https://admaxxer.com/integrations/mcp. Walk the whole flow in a client before
telling a reviewer that OAuth sign-in is the supported path.

## 1. xAI plugin marketplace (Grok Build)

1. Fork https://github.com/xai-org/plugin-marketplace (fork
   `fortuneflick/plugin-marketplace` exists), branch `add-admaxxer` from
   upstream `main`.
2. Append the entry to `.grok-plugin/marketplace.json`, with `source.sha` set
   to the 40-char lowercase SHA of this repository's `main` (a tag or branch is
   rejected).
3. `python3 scripts/generate-plugin-index.py`, then
   `python3 scripts/validate-catalog.py` and
   `python3 scripts/generate-plugin-index.py --check` — all three must pass.
4. PR title `Add admaxxer`. Keywords and domains are brand-scoped on purpose;
   xAI rejects generic terms such as `analytics` or `ads`.
5. After any change to this repository, open a follow-up PR bumping `sha`.
   **Never a parallel entry.**

## 2. Anthropic plugin directory (Claude Code / Cowork)

- Portal: https://platform.claude.com/plugins/submit (Console; works on an
  individual account — the claude.ai-side path needs a Team/Enterprise org).
- Repository URL: `https://github.com/fortuneflick/admaxxer-claude-plugin`
- Plugin name `admaxxer` · marketplace name `admaxxer` · manifest
  `.claude-plugin/plugin.json` · category `productivity` · license MIT
- Homepage: `https://admaxxer.com/documentation/connect-any-ai`
- Description: use the `description` in `.claude-plugin/plugin.json` verbatim.
- Note for the reviewer: the plugin is skill-only by design; the server is
  added separately with `claude mcp add --transport http admaxxer
  https://admaxxer.com/mcp`, so installing it never registers a second server
  next to an existing connector. No credential is written to a config file and
  none is in this repository.
- Pushes to this repository are picked up automatically. **Never open a second
  submission.**

## 3. Cursor marketplace

- Portal: https://cursor.com/marketplace/publish (clicking Submit accepts the
  Cursor Publisher Terms — owner only).
- Repository URL `https://github.com/fortuneflick/admaxxer-claude-plugin`,
  manifest `.cursor-plugin/plugin.json`, marketplace file
  `.cursor-plugin/marketplace.json`, logo
  `https://raw.githubusercontent.com/fortuneflick/admaxxer-claude-plugin/main/assets/logo.svg`,
  org name `Admaxxer`, handle `admaxxer`, contact `hello@admaxxer.com`,
  website `https://admaxxer.com`.
- The Cursor plugin is skill-only; the one-click server install is the deeplink
  in README.md.

## 4. claude.ai connectors directory (Team/Enterprise gated)

Packet to have ready:

| Field | Value |
|---|---|
| Name | Admaxxer |
| Slug | `admaxxer` |
| Tagline (≤55 chars) | `Ad ops and attribution your agent can read` (42) |
| Description | The README's opening two paragraphs |
| Categories | Productivity, Analytics |
| MCP server URL | `https://admaxxer.com/mcp` |
| Auth | Bearer token minted in the dashboard; OAuth discovery metadata is served (verified above) |
| Docs URL | `https://admaxxer.com/documentation/connect-any-ai` |
| Privacy URL | `https://admaxxer.com/privacy` |
| Support | `hello@admaxxer.com` |
| Icon | `assets/icon-192.png` |
| Example prompts | "What did I spend on Meta last week and what was the ROAS?" · "Which channel actually drove last week's revenue?" · "Show me the worst ad sets by ROAS in my prospecting campaign" · "Pause the Summer Sale campaign (I'll confirm)" · "How many sessions did the site get this week?" |
| Test account | PLACEHOLDER — a demo workspace with a connected ad account, 30 days of data, a read-only token and a manage token, signed in without MFA |

## 5. ChatGPT plugins (last)

- Portal: https://platform.openai.com/plugins (OpenAI org login with Apps
  Management access and a verified publisher identity; there is no public
  status check).
- Requirements: `/.well-known/openai-apps-challenge` served from admaxxer.com
  with the token the portal issues; honest tool annotations; a fully featured
  demo account **without MFA**; exactly 5 positive and 3 negative test cases;
  privacy, terms and support URLs; tested in Developer Mode on desktop and
  mobile.
- **Owner step before submitting:** the challenge route does not exist yet. Ask
  for the token in the portal, then serve it next to the other `/.well-known`
  documents — a code change in the platform repository, not a DNS record.
- Policy: OpenAI's guidelines ban in-plugin upselling. The tools never mention
  plans or credits; say so in the review notes, along with the read-only
  default, the two-step confirm and the forced-paused creation.

## 6. MCP registry (registry.modelcontextprotocol.io)

`server.json` declares the namespace **`com.admaxxer/mcp-server`**. A `com.*`
namespace is proved by DNS, not by the repository, so this needs a TXT record —
an owner step, and the only DNS in this checklist.

```bash
brew install mcp-publisher
mcp-publisher login dns --domain admaxxer.com --private-key <ed25519 hex>
mcp-publisher publish          # from a directory holding this server.json
```

That login prints the TXT record to publish on `admaxxer.com`
(`_mcp-registry.admaxxer.com`, value `v=MCPv1; k=ed25519; p=<public key>`).
Keep the private key offline; it is what republishing needs.

**Alternative with no DNS:** rename the server to
`io.github.fortuneflick/admaxxer-mcp-server` and prove it with
`mcp-publisher login github`, an interactive device-code flow.

There is no npm launcher: the server is hosted only, so `server.json` carries
`remotes` and no `packages`.
