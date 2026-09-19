# Admaxxer MCP tools

Generated from the server's tool registry by `npm run gen:agent-skill`. Do
not hand-edit: the next generation overwrites it, and the parity canary fails
the build if this file and the server disagree.

Endpoint `https://admaxxer.com/mcp` (Streamable HTTP), or `https://admaxxer.com/mcp/sse` for
clients that only speak the older HTTP+SSE form. Authorization is
`Bearer <token>`, with a token minted at
https://admaxxer.com/integrations/mcp — read-only unless the person
picks "Read + manage".

28 tools: 24 read, 4 write. 5 require the manage scope.

## `admaxxer_whoami`

**Scope:** read · **Kind:** read-only

Who this token is: workspace id/name, user id, token id, scopes, whether
writes are allowed, plan, timezone, and connected platforms. Use this first on
a new conversation. Do NOT use it to look up another workspace — the workspace
is always the token's. Typical timeout 30s.

**Inputs:** none

**Example prompt:** Who am I authenticated as, and can I change campaigns?

## `admaxxer_list_connections`

**Scope:** read · **Kind:** read-only

List this workspace's connected ad-platform accounts (Meta, Google, Shopify,
TikTok, Klaviyo). Returns connection IDs you can pass to the other admaxxer_*
tools. Call this FIRST to discover what data is available. Do NOT invent a
connection_id. Typical timeout 30s.

**Inputs:** none

**Example prompt:** What ad accounts do I have connected?

## `admaxxer_list_campaigns`

**Scope:** read · **Kind:** read-only

List ad campaigns for a connected account with the SAME normalized,
to-the-cent metrics the Admaxxer dashboard shows: spend, revenue, ROAS,
conversions, impressions, clicks, CTR, CPC, and daily budget — all in MAJOR
currency units (dollars, not cents) with lowercase statuses (active/paused).
Defaults to the last 7 days; pass date_from/date_to (YYYY-MM-DD) for a custom
window. Use admaxxer_list_connections first to get connection_id values. Do
NOT call this once per campaign — one call returns the roster; page with
cursor if total > 100. Typical timeout 30s.

**Inputs:** `connection_id` (required), `date_from`, `date_to`, `limit`, `cursor`

**Example prompt:** Show me all my Meta campaigns sorted by spend.

## `admaxxer_list_adsets`

**Scope:** read · **Kind:** read-only

List ad sets (Meta) / ad groups (Google) for a connected account with the SAME
normalized, to-the-cent metrics as admaxxer_list_campaigns: spend, revenue,
ROAS, conversions, impressions, clicks, CTR, CPC, and daily budget — all in
MAJOR currency units (dollars, not cents) with lowercase statuses
(active/paused). This is ACCOUNT-WIDE: it returns every ad set across every
campaign in ONE call. To drill into a single campaign, filter the returned
rows in memory by their campaignId field (the same id admaxxer_list_campaigns
returns) — do NOT call again per campaign. Each row also carries campaignName
for display. Daily budget is set only on Meta ad sets that carry their own
budget (non-CBO); it is null under campaign-level budget optimization and
always null for Google ad groups. Defaults to the last 7 days; pass
date_from/date_to (YYYY-MM-DD) for a custom window. Read-only. Returns
{connection, currency, writable, adsets, cache, fetchedAt}.

**Inputs:** `connection_id` (required), `date_from`, `date_to`, `limit`, `cursor`

**Example prompt:** Show me the ad sets under my best Meta campaign, by ROAS.

## `admaxxer_list_ads`

**Scope:** read · **Kind:** read-only

List individual ads (Meta) / ad-group ads (Google) for a connected account
with the SAME normalized, to-the-cent metrics as admaxxer_list_campaigns:
spend, revenue, ROAS, conversions, impressions, clicks, CTR, CPC — all in
MAJOR currency units (dollars, not cents) with lowercase statuses
(active/paused). This is ACCOUNT-WIDE: it returns every ad across every ad set
and campaign in ONE call. To drill into a single ad set or campaign, filter
the returned rows in memory by their adSetId or campaignId field (the same ids
admaxxer_list_adsets / admaxxer_list_campaigns return) — do NOT call again per
ad set. Each row carries campaignName + adSetName for display, plus
thumbnailUrl and previewUrl when the platform provides them (Meta creatives;
Google search/PMax ads have none, so those are null). Defaults to the last 7
days; pass date_from/date_to (YYYY-MM-DD) for a custom window. Read-only.
Returns {connection, currency, writable, ads, cache, fetchedAt}.

**Inputs:** `connection_id` (required), `date_from`, `date_to`, `limit`, `cursor`

**Example prompt:** Which ads in my retargeting ad set have the worst ROAS this week?

## `admaxxer_get_campaign_insights`

**Scope:** read · **Kind:** read-only

Fetch performance metrics (spend, impressions, clicks, CTR, CPA, ROAS,
conversions) for a single campaign over a date range. Defaults to the last 7
days if no dates are given. Do NOT use this to list every campaign — use
admaxxer_list_campaigns (cached, one call). Typical timeout 30s.

**Inputs:** `connection_id` (required), `campaign_id` (required), `date_from`, `date_to`

**Example prompt:** How did campaign 'BFCM Scale' perform last week?

## `admaxxer_get_account_insights`

**Scope:** read · **Kind:** read-only

Fetch account-level performance across all campaigns for a date range,
optionally broken down by age, gender, country, device, placement, etc. (Meta
breakdowns vocabulary). Do NOT request many breakdown combinations in a loop.
Typical timeout 30s.

**Inputs:** `connection_id` (required), `date_from`, `date_to`, `breakdowns`

**Example prompt:** What's my Meta account ROAS for the last 30 days?

## `admaxxer_query_metrics`

**Scope:** read · **Kind:** read-only

Query Admaxxer analytics via a whitelisted analytics pipe. Pick a pipe +
supply params; workspace_id is injected server-side. Available pipes:
blended_pnl (P&L over a range), revenue_breakdown (by
platform|source|product|collection|discount_code), ltv_snapshot (customer LTV
vs CAC), subscription_health (MRR/ARR/churn), dashboard_why_delta (top drivers
between two periods). Do NOT pass workspace_id — it is injected. Do NOT invent
a pipe name. Typical timeout 30s.

**Inputs:** `pipe` (required), `params` (required)

**Example prompt:** What's my blended MER for the last 14 days vs the prior 14 days?

## `admaxxer_get_workspace_context`

**Scope:** read · **Kind:** read-only

Return workspace metadata so the external AI can orient: workspace id/name,
plan key, active ad platforms, primary currency, and timezone. Prefer
admaxxer_whoami on a new conversation (it also returns scopes). Do NOT use
this to select another workspace. Typical timeout 30s.

**Inputs:** none

**Example prompt:** What plan am I on and when was my data last synced?

## `admaxxer_get_event_setup`

**Scope:** read · **Kind:** read-only

Return ready-to-paste analytics event-tracking code (pixel install +
identify() + SaaS funnel events like
signup/trial_started/subscription_started) PRE-FILLED with THIS workspace's
real pixel website id and domain. Use this to instrument a SaaS app: drop
block 1 into the site's <head> and add identify()/funnel calls to
signup/login/checkout code. READ-ONLY — returns code text only, changes
nothing. Note: SaaS revenue (MRR/ARR, incl. annual plans) is captured
automatically by the Stripe connection, NOT by these pixel events; identify()
is the attribution glue and only attributes events going forward (it cannot
back-fill pre-existing subscriptions). Full human guide:
https://admaxxer.com/documentation/saas-analytics.

**Inputs:** none

**Example prompt:** Fetch my Admaxxer event setup and add the pixel and identify call to this app.

## `admaxxer_update_campaign`

**Scope:** Read + manage · **Kind:** write (two-step confirm)

Pause/resume a campaign or set its daily budget (campaign-level only).
TWO-STEP: the first call WITHOUT confirmToken changes nothing — it returns a
preview {confirmRequired, confirmToken, summary, diff}; show the preview to
the user and ask for explicit approval, then call again with the SAME
arguments PLUS the confirmToken. Requires an MCP token minted with the 'Read +
manage' scope. Do NOT use this to change an ad set or ad — use
admaxxer_update_adset / admaxxer_update_ad. Do NOT use this to delete or to
raise spend past the preview the person approved. Typical timeout 30s.

**Inputs:** `connection_id` (required), `campaign_id` (required), `status`, `dailyBudget`, `confirmToken`

**Example prompt:** Pause the 'Summer Sale — Prospecting' campaign on Meta (I'll confirm).

## `admaxxer_update_adset`

**Scope:** Read + manage · **Kind:** write (two-step confirm)

Pause/resume an ad set (Meta) / ad group (Google), or set its daily budget
when the platform keeps the budget on the ad set (Meta non-CBO only).
TWO-STEP: first call without confirmToken returns a preview and changes
nothing; after the person agrees, call again with the same arguments plus
confirmToken. Requires the 'Read + manage' scope. Do NOT use this for a
campaign or an ad. Google ad-group budget returns budget_not_editable — change
the campaign instead. Typical timeout 30s.

**Inputs:** `connection_id` (required), `adset_id` (required), `status`, `dailyBudget`, `confirmToken`

**Example prompt:** Pause the retargeting ad set under Summer Sale (I'll confirm).

## `admaxxer_update_ad`

**Scope:** Read + manage · **Kind:** write (two-step confirm)

Pause or resume one ad. Status only — ads do not carry a daily budget.
TWO-STEP: first call without confirmToken returns a preview and changes
nothing; after the person agrees, call again with the same arguments plus
confirmToken. Requires the 'Read + manage' scope. Do NOT use this to edit
creative or to delete. Typical timeout 30s.

**Inputs:** `connection_id` (required), `ad_id` (required), `status` (required), `confirmToken`

**Example prompt:** Pause the worst-ROAS ad in that ad set (I'll confirm).

## `admaxxer_get_creation_options`

**Scope:** Read + manage · **Kind:** read-only

Fetch everything the AI needs to build a valid campaign launch for one Meta or
Google connection: for Meta — the Facebook pages the ad can run as, the ad
account's pixels (required for sales/leads objectives), and recent creatives
you can reuse; for Google — the customer account name. Also returns writable
(false ⇒ this connection's token is read-only and admaxxer_create_launch will
be refused). Warm-cached (~15 min) — re-calling costs zero platform calls.
Call this BEFORE admaxxer_create_launch so you fill pageId/pixelId/creativeId
from real values instead of guessing. Requires an MCP token minted with the
'Read + manage' scope. Do NOT use this as a substitute for list_campaigns.
Typical timeout 30s.

**Inputs:** `connection_id` (required)

**Example prompt:** What pages and pixels can I build a Meta ad with on this account?

## `admaxxer_create_launch`

**Scope:** Read + manage · **Kind:** write (two-step confirm)

Create a campaign / ad set (Meta) or ad group (Google) / ad. Google launches
ONE atomic all-or-nothing batch (pre-validated with a dry run before anything
is created — it either fully succeeds or creates nothing). Meta creates the
entities SEQUENTIALLY (campaign → ad set → image upload → creative → ad); on a
partial failure it returns `resume` so you can continue without re-creating
what already exists. TWO-STEP PROTOCOL (mandatory): the FIRST call WITHOUT
confirmToken creates NOTHING — it returns a plan {summary, steps, warnings,
confirmToken}. Show that summary to the human, get explicit approval, then
call AGAIN with IDENTICAL arguments PLUS the confirmToken from the preview.
(An expired/consumed token re-issues a fresh preview; a token that no longer
matches the arguments is rejected.) PAUSED RULE: Everything is created PAUSED.
Activation is not available over MCP — a human sets it live from the Ads
Manager, or via admaxxer_update_campaign's confirmed status change. Do NOT try
to activate here. Money is in MAJOR currency units (dollars, not cents).
Requires an MCP token minted with the 'Read + manage' scope. Fields mirror the
launch contract: platform 'meta'|'google'; connection_id; campaign is either
{existingId} (add to an existing campaign) or the new-campaign fields; adset
(Meta) / adGroup (Google); ad. For Meta a new campaign needs objective
(OUTCOME_SALES|OUTCOME_TRAFFIC|OUTCOME_LEADS|OUTCOME_ENGAGEMENT|OUTCOME_AWARENESS),
budgetMode ('campaign' = one budget on the campaign, or 'adset' = budget on
each ad set), and a dailyBudget on whichever level carries it; an
OUTCOME_SALES/OUTCOME_LEADS ad set REQUIRES a pixelId; a link ad needs pageId
+ primaryText + linkUrl. For Google a new Search campaign needs dailyBudget +
bidding + countries; an ad group needs 1+ keywords each with matchType
broad|phrase|exact; a responsive search ad needs 3-15 headlines (≤30 chars
each) and 2-4 descriptions (≤90 chars each) plus finalUrl. Server-side zod is
the authoritative validator — malformed launches come back as a structured
error listing the field problems.

**Inputs:** `platform` (required), `connection_id` (required), `campaign` (required), `adset`, `adGroup`, `ad`, `confirmToken`

**Example prompt:** Draft a paused Google Search campaign for 'longevity supplements' at $30/day (I'll review).

## `admaxxer_get_summary_kpis`

**Scope:** read · **Kind:** read-only

Get this workspace's headline analytics KPIs over a date range — the same
numbers the dashboard hero tiles show: total sales, order revenue, net profit,
blended ad spend, MER, blended ROAS, AOV, orders, sessions, unique visitors,
conversion rate, plus per-platform ad spend/ROAS (Meta, Google, TikTok) and
Klaviyo email revenue. All money is in the workspace's display currency, in
major units. Defaults to the last 7 days. Use this first for workspace-level
numbers. Do NOT use this for one campaign — use admaxxer_list_campaigns.
Typical timeout 30s.

**Inputs:** `date_from`, `date_to`, `website_id`

**Example prompt:** Give me my headline KPIs for the last 14 days.

## `admaxxer_get_attribution_breakdown`

**Scope:** read · **Kind:** read-only

Get channel-level revenue attribution for this workspace over a date range —
the same first-party (pixel) channel rows the dashboard's Sources &
Attribution grid shows. Returns one row per marketing channel with attributed
revenue, ad spend, ROAS, orders, and CPA, under the selected attribution
model. Money is in the workspace's display currency, major units. Defaults to
last 7 days and the last-click model. Do NOT pick a flattering model after the
fact — say the model you used. Typical timeout 30s.

**Inputs:** `date_from`, `date_to`, `model`, `website_id`

**Example prompt:** Break down last week's revenue by channel using last-click attribution.

## `admaxxer_get_web_analytics`

**Scope:** read · **Kind:** read-only

Get this workspace's web analytics over a date range: sessions, unique
visitors, pageviews, new users, pages per session, bounce rate, conversion
rate, and average session duration. These are the de-duplicated,
accuracy-corrected figures the dashboard shows. Defaults to the last 7 days.
Do NOT use this for ad-platform ROAS — use admaxxer_get_summary_kpis or
list_campaigns. Typical timeout 30s.

**Inputs:** `date_from`, `date_to`, `website_id`

**Example prompt:** How many sessions and visitors did the site get this week?

## `admaxxer_get_ai_search_visibility`

**Scope:** read · **Kind:** read-only

Get AI Search Visibility for this workspace: whether ChatGPT, Perplexity,
Gemini and other assistants mention the brand for each tracked prompt, with
30-day mention rate, position, and sentiment. Uses the customer's own
connected key; returns empty when no prompts have been run. Do NOT invent a
mention rate when the list is empty. Typical timeout 30s.

**Inputs:** none

**Example prompt:** Do AI assistants mention us when someone asks for tools like ours?

## `admaxxer_get_ai_citations`

**Scope:** read · **Kind:** read-only

Get the URLs and domains that AI assistants cited when answering this
workspace's tracked prompts over the last 30 days. Useful for seeing which
pages assistants treat as sources. Do NOT treat an empty list as 'nobody cites
you' if no prompts have been run — say that instead. Typical timeout 30s.

**Inputs:** none

**Example prompt:** Which pages do AI assistants cite when they answer about us?

## `admaxxer_list_shopify_orders`

**Scope:** read · **Kind:** read-only

List recent Shopify orders for a connected store (official Admin API, one
cached page). Returns id, created_at, total, currency, financial_status, and
line-item titles — not customer email or phone. Use when the person asks what
sold. Do NOT use this to refund, fulfill, or message a buyer (no such tool
exists). Do NOT loop pages to build a full export; page with next_cursor.
Typical timeout 30s.

**Inputs:** `connection_id` (required), `created_at_min`, `limit`, `cursor`

**Example prompt:** What did we sell on Shopify yesterday?

## `admaxxer_list_shopify_products`

**Scope:** read · **Kind:** read-only

List products in a connected Shopify store (official Admin API, one cached
page). Use when the person asks what is in the catalog. Do NOT use this to
create, update, or unpublish a product. Do NOT loop pages. Typical timeout
30s.

**Inputs:** `connection_id` (required), `limit`, `cursor`

**Example prompt:** What's in the Shopify catalog right now?

## `admaxxer_list_klaviyo_flows`

**Scope:** read · **Kind:** read-only

List Klaviyo flows on a connected account (official API, one cached page). Use
when the person asks which automations exist or whether a flow is live. Do NOT
use this to start, stop, or edit a flow — those writes are not exposed. Do NOT
paginate in a tight loop. Typical timeout 30s.

**Inputs:** `connection_id` (required), `limit`, `cursor`

**Example prompt:** Which Klaviyo flows are live?

## `admaxxer_list_klaviyo_lists`

**Scope:** read · **Kind:** read-only

List Klaviyo lists/segments on a connected account (official API, one cached
page). Use when the person asks which lists exist. Do NOT subscribe anyone or
send a campaign — those writes are not exposed. Typical timeout 30s.

**Inputs:** `connection_id` (required), `limit`, `cursor`

**Example prompt:** What Klaviyo lists do we have?

## `admaxxer_list_alerts`

**Scope:** read · **Kind:** read-only

List this workspace's alert rules and the last 20 firings (goal hit,
threshold, anomaly, trend). Use when the person asks what fired or what is
watched. Do NOT use this to create, edit, or send an alert — those writes are
not exposed over MCP (they email or Slack people). Typical timeout 30s.

**Inputs:** `website_id`

**Example prompt:** Which alerts fired this week?

## `admaxxer_get_audience_demographics`

**Scope:** read · **Kind:** read-only

Platform-reported age/gender of people the ads reached (Meta crossed grid;
Google reports age and gender separately). Same numbers as the Audience
demographics card. Use when the person asks who the ads are reaching. Do NOT
treat this as site-visitor demographics — the pixel cannot see age/gender.
Typical timeout 30s.

**Inputs:** `date_from`, `date_to`, `platform`

**Example prompt:** Who are the ads actually reaching by age and gender?

## `admaxxer_get_source_audience`

**Scope:** read · **Kind:** read-only

Countries, devices, and browsers of sessions behind one (source, medium) row —
the same drill as the attribution table. Use after
admaxxer_get_attribution_breakdown when the person asks who sat behind a
channel. Do NOT call this once per channel in a loop; pick the row that
matters. Typical timeout 30s.

**Inputs:** `source` (required), `medium` (required), `row_id`, `date_from`, `date_to`, `website_id`

**Example prompt:** Who sat behind facebook/paid this week — country and device?

## `admaxxer_export_report`

**Scope:** read · **Kind:** read-only

Return the dashboard's daily, weekly, or monthly report as JSON (sessions,
revenue, top pages, top referrers, optional AI-search line). Use when the
person asks for a report they can paste or file. Do NOT use this to email
anyone — it never sends. Typical timeout 30s.

**Inputs:** `frequency`

**Example prompt:** Export this week's report as JSON so I can paste it.
