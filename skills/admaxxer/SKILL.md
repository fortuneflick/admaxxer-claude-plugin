---
name: admaxxer
description: Admaxxer is ad account analytics and campaign control for AI agents. Read spend, ROAS, MER, attribution and traffic, then pause a campaign or change a daily budget with a preview the person approves. Use when asked about ad spend, ROAS, MER, attribution or traffic, when a campaign needs pausing or a budget change, or when building a new campaign from an agent.
---
# Running ads and analytics through Admaxxer

You are reading and changing a real advertising account. Money moves. Three
rules make that safe, and the server enforces all three, so you cannot talk
your way past them.

1. **A token is read-only unless the person minted it otherwise.** Of the
   28 tools, 24 read and 4 write; 5 need a token minted with
   the "Read + manage" scope at https://admaxxer.com/integrations/mcp.
   Asked to change something on a read-only token, you get
   `manage_scope_required` back. That is the person's decision, not an error
   to route around: say what the token cannot do and stop.
2. **Every write is two steps.** The first call returns a preview with a
   `confirmToken`. Show the preview, in full, and wait for the person to say
   yes. Then call again with the same arguments plus that token. Never send
   both steps without a human answer in between, and never treat an earlier
   "go ahead" as consent for a later change.
3. **Anything created lands paused.** There is no activate-on-create option.
   Say so when you finish building something, so nobody waits for a delivery
   that will not start until they set it live in the dashboard.

## Start by finding out what exists

Call `admaxxer_list_connections` first. It returns the connection IDs every
other tool takes, and their health. A connection whose token expired explains
an empty report better than a guess does.

For workspace-level numbers, `admaxxer_get_summary_kpis` is the fastest
orientation: revenue, ad spend, MER, blended ROAS, AOV, orders, sessions and
conversion rate over a window, in the workspace's display currency.

## Read before you recommend

Campaign, ad set and ad rosters come back whole for an account in one call.
Filter the response in memory on `campaignId` or `adSetId` to drill in.
**Do not loop tool calls to walk a hierarchy** — an ad account that gets burst
with requests can be rate limited or flagged, and that is expensive to undo.
When you need a narrower window, change the date range rather than calling
again per entity.

Attribution answers "which ad actually drove that sale": use
`admaxxer_get_attribution_breakdown` and name the model and lookback window
you used, because the same week reads differently under last-click than under
time-decay. When a platform's own number disagrees with the attributed one,
say both and say which is which instead of picking the flattering one.

## Changing a campaign

- `admaxxer_update_campaign` — Pauses/resumes a campaign or changes its daily budget — campaign-level only, never deletion. Requires a token minted with the 'Read + manage' scope, and EVERY change is two-step: your AI shows you a preview (e.g. "Daily budget $50 → $75") and only executes after you approve it in the conversation.
- `admaxxer_update_adset` — Pauses/resumes an ad set (Meta) or ad group (Google), or changes its daily budget when the platform keeps the budget on the ad set. Two-step preview + confirmToken. Requires the 'Read + manage' scope.
- `admaxxer_update_ad` — Pauses or resumes one ad. Status only — ads do not carry a daily budget. Two-step preview + confirmToken. Requires the 'Read + manage' scope.
- `admaxxer_create_launch` — Creates campaigns, ad sets, and ads — a full Meta funnel, or a complete Google Search campaign (budget, keywords, and ad built together). Two-step: your AI shows you the exact plan and only builds after you approve it in the conversation. Everything it creates lands PAUSED — there is no activate option over MCP, so nothing your AI builds can start spending; you set it live from the dashboard. Requires the 'Read + manage' scope.

All write tools refuse a token without the manage scope. None of them
delete anything. Campaign, ad-set and ad pause/resume (and budget where the
platform keeps it) are two-step. Creation lands paused. Budgets are in major
currency units (dollars, not cents) — a factor of a hundred here is a real
hundred.

## When you cannot answer

Say which tool returned nothing and why: no connection, an expired token, a
window with no data, a scope the token does not hold. A confident number
assembled from a failed read is worse than no number.

## Every tool

- `admaxxer_whoami` — Returns who this token is — workspace, scopes, whether writes are allowed, plan, and connected platforms. The first call on a new conversation.
- `admaxxer_list_connections` — Lists every Meta + Google ad-platform connection in the workspace, with status (healthy / token expired / disabled), platform, and account label. Used by the AI to discover which connections to query for insights.
- `admaxxer_list_campaigns` — Lists active or paused campaigns for a given connection (Meta or Google). Returns campaign ID, name, objective, status, daily budget, and lifetime spend.
- `admaxxer_list_adsets` — Lists ad sets (Meta) / ad groups (Google) for a connection with the same to-the-cent metrics as campaigns — spend, ROAS, conversions, CTR, CPC, daily budget. Returns the whole account in one call; your AI filters by campaignId to drill into a single campaign without another request.
- `admaxxer_list_ads` — Lists individual ads (Meta) / ad-group ads (Google) for a connection with the same metrics, plus creative thumbnail and preview links where the platform provides them. Returns the whole account in one call; your AI filters by adSetId or campaignId to drill in without another request.
- `admaxxer_get_campaign_insights` — Pulls metrics for one campaign over a date range — impressions, clicks, spend, conversions, ROAS, CPA. Optional breakdowns by age, gender, device, region.
- `admaxxer_get_account_insights` — Account-level rollup of insights across all campaigns for a connection. Optional breakdowns. Best for high-level ROAS / spend trends.
- `admaxxer_query_metrics` — Runs a whitelisted analytics-warehouse query (visitors, revenue, MER, LTV, MMM, attribution, forecast). The workspace_id is injected server-side — the AI cannot see another workspace's data.
- `admaxxer_get_workspace_context` — Returns the workspace's plan tier, currency, time zone, connected platforms, and last-sync timestamps. Useful for the AI to ground its answers.
- `admaxxer_get_event_setup` — Returns copy-paste event-tracking code (pixel install + identify() + funnel events) pre-filled with your website id, so your AI can wire analytics into your app for you.
- `admaxxer_update_campaign` *(write, confirm-gated)* — Pauses/resumes a campaign or changes its daily budget — campaign-level only, never deletion. Requires a token minted with the 'Read + manage' scope, and EVERY change is two-step: your AI shows you a preview (e.g. "Daily budget $50 → $75") and only executes after you approve it in the conversation.
- `admaxxer_update_adset` *(write, confirm-gated)* — Pauses/resumes an ad set (Meta) or ad group (Google), or changes its daily budget when the platform keeps the budget on the ad set. Two-step preview + confirmToken. Requires the 'Read + manage' scope.
- `admaxxer_update_ad` *(write, confirm-gated)* — Pauses or resumes one ad. Status only — ads do not carry a daily budget. Two-step preview + confirmToken. Requires the 'Read + manage' scope.
- `admaxxer_get_creation_options` — Read-only helper for creation: the Facebook Pages, pixels, and saved creatives available on a connection, plus whether it can create at all. Changes nothing — but it lives behind the same 'Read + manage' scope as the create tool, since that's all it's for.
- `admaxxer_create_launch` *(write, confirm-gated)* — Creates campaigns, ad sets, and ads — a full Meta funnel, or a complete Google Search campaign (budget, keywords, and ad built together). Two-step: your AI shows you the exact plan and only builds after you approve it in the conversation. Everything it creates lands PAUSED — there is no activate option over MCP, so nothing your AI builds can start spending; you set it live from the dashboard. Requires the 'Read + manage' scope.
- `admaxxer_get_summary_kpis` — Returns the dashboard's headline numbers for a date window — revenue, ad spend, blended MER/ROAS, orders, sessions, visitors — straight from the same analytics store the dashboard reads, so your AI quotes the numbers you see.
- `admaxxer_get_attribution_breakdown` — Channel-level revenue, spend, and ROAS under your chosen attribution model (last click, first click, linear, time decay, position based, and more) — the same rows as the Sources & Attribution drill-down.
- `admaxxer_get_web_analytics` — Sessions, unique visitors, pageviews, and bounce rate for a date window — the corrected, dashboard-matching web analytics read.
- `admaxxer_get_ai_search_visibility` — Reports whether ChatGPT, Perplexity, Gemini and other assistants mention the brand for each tracked prompt, with the 30-day mention rate, position and sentiment. Runs on the workspace's own connected key; empty until prompts have been run.
- `admaxxer_get_ai_citations` — Lists the URLs and domains AI assistants cited when they answered the workspace's tracked prompts over the last 30 days, so you can see which pages they treat as sources.
- `admaxxer_list_shopify_orders` — Lists recent Shopify orders for a connected store (id, total, currency, status, line items). Customer email is omitted. One cached page; pass next_cursor for more.
- `admaxxer_list_shopify_products` — Lists products in a connected Shopify store (title, status, type, vendor, variant prices). One cached page.
- `admaxxer_list_klaviyo_flows` — Lists Klaviyo flows on a connected account (name, status, archived). One cached page. Does not start or stop a flow.
- `admaxxer_list_klaviyo_lists` — Lists Klaviyo lists and segments on a connected account. One cached page. Does not subscribe anyone.
- `admaxxer_list_alerts` — Lists alert rules (goal hit, threshold, anomaly, trend) and the last firings. Does not create or send an alert.
- `admaxxer_get_audience_demographics` — Platform-reported age and gender of people the ads reached — the same Audience demographics card, not site-visitor demographics.
- `admaxxer_get_source_audience` — Countries, devices, and browsers of sessions behind one (source, medium) row from the attribution table.
- `admaxxer_export_report` — Returns the dashboard's daily, weekly, or monthly report as JSON. Does not send email.

Full descriptions and inputs: `references/tools.md`. Setup for each client:
https://admaxxer.com/documentation/connect-any-ai.
