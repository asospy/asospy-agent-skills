# ASOSpy Agent Skills

Skills that teach AI agents (Claude Code, Codex, Cursor and others) how to do App Store and Google Play research with the [ASOSpy MCP server](https://asospy.dev/mcp).

Each skill is a folder with a `SKILL.md` file. It says when to use the skill, what to ask for, which ASOSpy tools to call in which order, what to return, and what the data cannot tell you.

| Skill | Use it to |
|---|---|
| [app-discovery](skills/app-discovery/SKILL.md) | Find and shortlist apps for a goal, and spot the ones growing now |
| [keyword-research](skills/keyword-research/SKILL.md) | Build a keyword shortlist around a seed term |
| [competitor-analysis](skills/competitor-analysis/SKILL.md) | Compare an app with its competitors |
| [growth-analysis](skills/growth-analysis/SKILL.md) | Explain how an app grew or declined over a period |
| [review-analysis](skills/review-analysis/SKILL.md) | Summarise what users love and hate in store reviews |
| [aso-audit](skills/aso-audit/SKILL.md) | Audit a store listing and list prioritised fixes |

## 1. Connect the ASOSpy MCP server

You need an ASOSpy workspace with API access and an **API key** (it starts with `ask_`). Create one in ASOSpy under **AI Agents > API Keys**. The same key works for the MCP server and the REST API, and both share your plan's monthly quota.

- Server URL: `https://mcp.asospy.dev/api` (Streamable HTTP)
- Header: `Authorization: Bearer <your API key>`

API key rules:

- Only a workspace owner or admin can create keys.
- Creating a key and rotating a key both ask for your ASOSpy account password. If you sign in with Google or Apple and have no password yet, set one first with **Forgot password?** on the sign-in page. Setting it this way is a password reset, so it also switches off the keys you already have.
- Changing your password, resetting it, or signing out of every device at once ("sign out everywhere") switches off every API key you created.
- After that, your agent gets HTTP 401 (`INVALID_API_KEY`, "Unknown or revoked API key."). Create a new key and put it in your MCP client.

Claude Code:

```bash
export ASOSPY_API_KEY="ask_..."
claude mcp add --transport http --scope user asospy https://mcp.asospy.dev/api \
  --header "Authorization: Bearer ${ASOSPY_API_KEY}"
```

Setup for Cursor, Codex, Claude Desktop, Gemini CLI, VS Code, Cline, Continue and Devin Desktop: <https://asospy.dev/mcp#setup>.

## 2. Install the skills

Copy the folders you want from `skills/` into your agent's skills directory:

```bash
git clone https://github.com/asospy/asospy-agent-skills.git
# Claude Code, for every project:
mkdir -p ~/.claude/skills && cp -R asospy-agent-skills/skills/* ~/.claude/skills/
# or for one project:
mkdir -p .claude/skills && cp -R asospy-agent-skills/skills/* .claude/skills/
```

Other agents that support `SKILL.md` skills load them from their own skills folder. For agents without skills support, paste the body of a `SKILL.md` into the chat or the system prompt.

## 3. Use them

Ask in plain words, for example:

- "Find meditation apps that are growing fast on iOS."
- "Do keyword research for a sleep sounds app."
- "Compare Calm with Headspace and two other competitors on iOS."
- "Why did com.whatsapp's installs change in the last 60 days?"
- "What do users complain about in Duolingo's iOS reviews?"
- "Run an ASO audit of https://apps.apple.com/app/id571800810."

The server also has 7 built-in prompts: one per skill (`app_discovery`, `keyword_research`, `competitor_analysis`, `growth_analysis`, `review_analysis`, `aso_audit`) plus `market_research`. Clients that show MCP prompts list them in their prompt or slash-command menu.

## What the data is (and is not)

- **iOS installs and all revenue figures are ASOSpy estimates.** Apple and Google do not publish them. Revenue is `null` when there is no estimate; do not read `null` as zero.
- Install figures under the floor (100 on Google Play, 1,000 on the App Store) come back as text, `"< 100"` or `"< 1k"`, not as numbers. `get_app_historicals` is the exception: it keeps numbers so days can be added up, and gives the floor in `meta.metrics.installs.floor`; quote a day under it as `"< 100"` or `"< 1k"`.
- `ratingsCount` is a number of ratings; `ratingScore` is the 1–5 average.
- `applePopularity` in `search_keywords` and `search_niches` is Apple Search Ads popularity for the **US** App Store; in a `live_keyword_metrics` row it is for the `storefront` you asked for. `volume`, `cpc` and `competition` are Google Ads figures.
- Ads data covers app-ads.txt lines (the ad networks and publisher IDs an app lists) and scraped Apple Search Ads appearances (iOS, beta). There are no ad creatives, spend or impressions.

## Quota and limits

- A successful tool call uses one unit of your workspace's monthly API quota, the same quota as REST API calls. This includes a result with no rows, or a `live_keyword_metrics` row with `measured: false`. Connecting, listing tools and prompts, and reading a prompt are free.
- Failed calls are free, with two exceptions:
  - A `live_keyword_metrics` lookup that Apple or Google refused (`details.reason` is `rate_limited` or `upstream_error`) still uses its unit.
  - A name lookup that returns `APP_NOT_FOUND` with `details.candidates` uses its unit, because the candidates are search results.
- The plan's per-minute limit applies to every request to the server. Over it, the server answers HTTP 429 `RATE_LIMITED` with `details.retryAfter` in seconds. With no quota left, a tool returns `INSUFFICIENT_QUOTA` with `details.resetsAt`.
- `live_keyword_metrics` has its own limit: 10 lookups per minute per workspace, shared with the live keyword lookup on the ASOSpy website. Over it, the tool returns `RATE_LIMITED` with `details.retryAfter`; that refusal is free.

## Errors

A tool error has `error.code`, `error.message`, `error.hint` and `error.details`.

- `INVALID_PARAMETER`: a missing or wrong value, or an unknown or misspelled argument. For an unknown argument, `error.message` names it and `error.hint` lists the tool's supported arguments.
- `APP_NOT_FOUND`: no app matched. When a name fits several apps, `details.candidates` lists them, most popular first; call again with the right `id`. A name that exists on both stores (for example "Headspace") needs `store: "ios"` or `store: "android"`.
- `COUNTRY_NOT_SUPPORTED` / `STORE_NOT_SUPPORTED`: the tool does not cover that country or store, or the app is on the other store than the `store` you passed.
- `RATE_LIMITED` / `INSUFFICIENT_QUOTA`: see Quota and limits. A `RATE_LIMITED` tool error can also mean too many parallel calls from one workspace; wait and retry.
- `live_keyword_metrics` refused by Apple or Google: `RATE_LIMITED` with `details.reason: "rate_limited"` (no `retryAfter`; try again in a minute) or `INTERNAL_ERROR` with `details.reason: "upstream_error"`. Both still use their unit. Other failures of this tool, such as an expired crawler session, are free.
- `INTERNAL_ERROR`: a server-side failure; try again shortly.
- `UNAUTHORIZED`: the workspace's plan does not include the feature (a tool error), or, as HTTP 401, the key's holder is no longer a workspace owner or admin, the account is suspended, or the plan has no API access. A new key does not fix these.
- `INVALID_API_KEY` (HTTP 401): the key is missing, wrong or switched off (see API key rules).

## Tools

The server has 11 tools. Every tool is read-only. Your MCP client shows each tool's full argument list.

| Tool | Returns |
|---|---|
| `search_apps` | Apps that match a query and filters, sorted by installs, revenue, release date and more. |
| `get_app_detail` | One app's full profile. |
| `get_app_historicals` | Daily `installs`, `revenue`, `ratings`, `reviews` and `score`; at most 366 days per call (default: the last 90 days). |
| `list_store_rankings` | One top chart (free, paid or grossing) for a store, country and category. |
| `get_store_ranking_history` | An app's chart ranks over time. |
| `search_keywords` | Related keywords with Google `volume`, `cpc`, `competition`, US `applePopularity` and the top apps. |
| `get_app_reviews` | Store reviews: most recent, highest rated or lowest rated first. |
| `search_ads` | Ads data in four modes (below). |
| `get_supported_countries` | The storefronts ASOSpy tracks. |
| `search_niches` | A keyword's market (apps, developers, installs, top apps) plus related keywords. |
| `live_keyword_metrics` | One keyword measured live at Apple or Google (below). |

`search_ads` modes:

- `publisher_apps` (`query`: a publisher ID from app-ads.txt): the apps whose app-ads.txt lists that ID.
- `keyword_advertisers` (`query`: a keyword): apps seen in Apple Search Ads results for it (iOS).
- `app_keywords` (`app`: an App Store app): the keywords the app was seen advertising on (iOS).
- `app_ad_networks` (`app`: an app on either store): the ad networks and publisher IDs in the app's app-ads.txt, networks with DIRECT lines (the app owner's own accounts) first. Pass a publisher ID to `publisher_apps` to see the other apps on that account.

`live_keyword_metrics` takes `keyword`, `source` (`apple` or `google`) and, for Apple, an optional `storefront` (default `US`):

- `apple` returns the keyword's Apple Search Ads popularity (5–100) in that storefront, plus up to 10 related keywords.
- `google` returns `vol` (Google Keyword Planner monthly searches), `cpc` (USD) and `competition` (0–1).
- `measured: false` means Apple or Google did not answer for this exact keyword: its value is unknown, not 0.
- Use it for a keyword that `search_keywords` does not have or shows as stale, or for Apple popularity outside the US. For lists of keywords, use `search_keywords`.
- One keyword per call, 10 per minute per workspace (see Quota and limits).
