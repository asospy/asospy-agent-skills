# ASOSpy Agent Skills

Skills that teach AI agents (ChatGPT, Claude, Claude Code, Codex, Cursor and others) how to do App Store and Google Play research with the [ASOSpy MCP server](https://asospy.com/mcp).

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

- Server URL: `https://mcp.asospy.dev/api` (Streamable HTTP)
- You need an ASOSpy workspace whose plan includes API access, and you must be an owner or admin of it.
- Every tool call counts toward that workspace's monthly API quota, shared with the REST API.

There are two ways to sign in. Pick the one your client supports.

### ChatGPT and Claude: OAuth sign-in (no API key)

Add ASOSpy as a custom connector with the server URL above and OAuth sign-in. The client registers itself, so there is no client ID or secret to enter. Then connect, sign in to ASOSpy, pick the workspace and choose **Allow**.

- **ChatGPT:** add ASOSpy as a custom connector (app) with OAuth. Whether you can add custom connectors depends on your ChatGPT plan and workspace settings.
- **Claude (web, desktop, mobile):** Customize → Connectors → Add custom connector. On Team and Enterprise plans, an Owner adds it under Organization settings → Connectors. Leave the OAuth client ID and secret empty, and do not choose "Use Claude's published identity".

Full guide for admins: <https://asospy.com/docs/mcp/admin-setup>.

### Agents in your terminal or IDE: API key

Claude Code, Codex, Cursor, Gemini CLI, VS Code, Cline, Continue and Devin Desktop send an **API key** (it starts with `ask_`). Create one in ASOSpy under **AI Agents > API Keys**. The same key works for the MCP server and the REST API.

- Header: `Authorization: Bearer <your API key>`

API key rules:

- Only a workspace owner or admin can create keys.
- Changing your password, resetting it, or signing out of every device at once ("sign out everywhere") switches off every API key you created.
- After that, your agent gets HTTP 401 (`INVALID_API_KEY`, "Unknown or revoked API key."). Create a new key and put it in your MCP client.

Claude Code:

```bash
export ASOSPY_API_KEY="ask_..."
claude mcp add --transport http --scope user asospy https://mcp.asospy.dev/api \
  --header "Authorization: Bearer ${ASOSPY_API_KEY}"
```

Setup for every client: <https://asospy.com/mcp#setup>.

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
- `search_keywords` rows say when each figure was last saved: `updatedAt` for the Google figures and `appleUpdatedAt` for `applePopularity`. Use them to spot stale values.
- Ads data covers app-ads.txt lines (the ad networks and publisher IDs an app lists) and scraped Apple Search Ads appearances (iOS, beta). There are no ad creatives, spend or impressions.

## Quota and limits

- A successful tool call uses one unit of your workspace's monthly API quota, the same quota as REST API calls. This includes a result with no rows. Connecting, listing tools and prompts, and reading a prompt are free.
- `live_keyword_metrics` costs more when it goes to the source:

  | Result | Units |
  |---|---|
  | Measured live at Apple or Google (`cached: false`), also a row with `measured: false` | 3 |
  | Answered from the cache (`cached: true`) | 1 |
  | Apple or Google refused or failed the lookup, or ASOSpy did not run it: no working session, the live service is busy, live lookups are switched off, a per-minute limit below, or the monthly live refreshes are used up | 0 |

- Each new live measurement (`cached: false`) also uses one of the workspace's monthly live keyword refreshes, shared with the live refresh on the ASOSpy website: 50 on Individual, 500 on Start-up and 2,000 on Business. A cached answer and a failed lookup do not use one. A result shows what is left in `meta.allowance` (`limit`, `remaining`, `resetsAt`). When they are used up, the tool returns `INSUFFICIENT_QUOTA` with `details.reason: "allowance_used"`, `details.limit` and `details.resetsAt`. The monthly API quota is separate, so the other tools keep working.
- Failed calls are free, with one exception: a name lookup that returns `APP_NOT_FOUND` with `details.candidates` uses its unit, because the candidates are search results.
- The plan's per-minute limit applies to every request to the server. Over it, the server answers HTTP 429 `RATE_LIMITED` with `details.retryAfter` in seconds. With no quota left, a tool returns `INSUFFICIENT_QUOTA` with `details.resetsAt`.
- `live_keyword_metrics` has its own limits: 10 new measurements per minute per user and 10 per minute per workspace, shared with the live keyword refresh on the ASOSpy website. Cached answers do not count. Over a limit, the tool returns `RATE_LIMITED` with `details.reason: "throttled"`, `details.scope` (`user` or `workspace`) and `details.retryAfter`; that refusal is free.

## Errors

A tool error has `error.code`, `error.message`, `error.hint` and `error.details`.

- `INVALID_PARAMETER`: a missing or wrong value, or an unknown or misspelled argument. For an unknown argument, `error.message` names it and `error.hint` lists the tool's supported arguments.
- `APP_NOT_FOUND`: no app matched. When a name fits several apps, `details.candidates` lists them, most popular first; call again with the right `id`. A name that exists on both stores (for example "Headspace") needs `store: "ios"` or `store: "android"`.
- `COUNTRY_NOT_SUPPORTED` / `STORE_NOT_SUPPORTED`: the tool does not cover that country or store, or the app is on the other store than the `store` you passed.
- `RATE_LIMITED` / `INSUFFICIENT_QUOTA`: see Quota and limits. A `RATE_LIMITED` tool error can also mean too many parallel calls from one workspace; wait and retry.
- `live_keyword_metrics` failures are free (0 units, no live refresh used):
  - Apple or Google asked to slow down: `RATE_LIMITED` with `details.reason: "rate_limited"` and `details.retryAfter`.
  - The live service is busy: `RATE_LIMITED` with `details.reason: "busy"` and `details.retryAfter`.
  - No working session, or Apple or Google refused the stored session: `INTERNAL_ERROR` with `details.reason` `"no_session"` or `"session_rejected"`.
  - Apple or Google did not answer: `INTERNAL_ERROR` with `details.reason: "upstream_error"`.
  - Live lookups switched off: `INTERNAL_ERROR` with `details.reason: "disabled"`.
  - Read `error.message`, use `search_keywords` for the stored values, and try again later.
- `INTERNAL_ERROR`: a server-side failure; try again shortly.
- `UNAUTHORIZED`: the workspace's plan does not include the feature (a tool error), or, as HTTP 401, the key's holder is no longer a workspace owner or admin, the account is suspended, or the plan has no API access. A new key does not fix these.
- `INVALID_API_KEY` (HTTP 401): the key is missing, wrong or switched off (see API key rules).

## Tools

The server has 11 tools. No tool can change your account, your workspace or your tracked apps. Ten tools only read ASOSpy data; `live_keyword_metrics` also saves the values it measures (below). Your MCP client shows each tool's full argument list.

| Tool | Returns |
|---|---|
| `search_apps` | Apps that match a query and filters, sorted by installs, revenue, release date and more. |
| `get_app_detail` | One app's full profile. |
| `get_app_historicals` | Daily `installs`, `revenue`, `ratings`, `reviews` and `score`; at most 366 days per call (default: the last 90 days). |
| `list_store_rankings` | One top chart (free, paid or grossing) for a store, country and category. |
| `get_store_ranking_history` | An app's chart ranks over time. |
| `search_keywords` | Related keywords with Google `volume`, `cpc`, `competition`, US `applePopularity`, when they were saved (`updatedAt`, `appleUpdatedAt`) and the top apps. |
| `get_app_reviews` | Store reviews: most recent, highest rated or lowest rated first. |
| `search_ads` | Ads data in four modes (below). |
| `get_supported_countries` | The storefronts ASOSpy tracks. |
| `search_niches` | A keyword's market (apps, developers, installs, top apps) plus related keywords. |
| `live_keyword_metrics` | One keyword measured live at Apple or Google, and saved in ASOSpy (below). |

`search_ads` modes:

- `publisher_apps` (`query`: a publisher ID from app-ads.txt): the apps whose app-ads.txt lists that ID.
- `keyword_advertisers` (`query`: a keyword): apps seen in Apple Search Ads results for it (iOS).
- `app_keywords` (`app`: an App Store app): the keywords the app was seen advertising on (iOS).
- `app_ad_networks` (`app`: an app on either store): the ad networks and publisher IDs in the app's app-ads.txt, networks with DIRECT lines (the app owner's own accounts) first. Pass a publisher ID to `publisher_apps` to see the other apps on that account.

`live_keyword_metrics` takes `keyword` (up to 120 characters), `source` (`apple` or `google`) and, for Apple, an optional `storefront` (two letters, default `US`):

- `apple` returns the keyword's Apple Search Ads popularity (5–100) in that storefront, plus up to 10 related keywords.
- `google` returns `vol` (Google Keyword Planner monthly searches, worldwide), `cpc` (USD) and `competition` (0–1).
- `measured: false` means Apple or Google did not answer for this exact keyword: its value is unknown, not 0.
- Every result has `cached`, `saved` and `measuredAt`:
  - `cached: true`: ASOSpy measured the same keyword recently and returns that measurement instead of asking again. Apple results are kept for 24 hours per storefront, Google results for 7 days. `measuredAt` says when the value was measured.
  - `saved: true`: the new value is now stored in ASOSpy, so `search_keywords` and the ASOSpy website show it to every user. Google values are saved every time they are measured. Apple popularity is saved only for the `US` storefront; other storefronts are returned and cached but not saved.
  - ASOSpy never saves a value that was not measured, and it does not save an Apple popularity of 0 over a stored value above 10 (`saved: false`).
- Use it for a keyword that `search_keywords` does not have or shows as stale (old `updatedAt` / `appleUpdatedAt`), or for Apple popularity outside the US. For lists of keywords, use `search_keywords`.
- One keyword per call, 10 new measurements per minute per user and per workspace. A live measurement costs 3 units and one monthly live refresh, a cached answer 1 unit, a failed lookup nothing (see Quota and limits).
