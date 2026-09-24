---
name: app-discovery
description: Find and shortlist App Store and Google Play apps for a goal (a use case, category or feature), with installs, revenue estimates and the apps growing fastest right now. Use when the user asks "which apps do X", "find apps like Y", "what's growing in Z" or wants a list of apps to study. Needs the ASOSpy MCP server.
---

# App discovery

## When to use

- The user wants a list of apps for a use case, category or feature ("habit trackers with subscriptions", "AI photo editors on Android").
- The user asks which apps in a space are growing now, or which are new.
- As the first step of competitor, market or keyword work when no app list exists yet.

Not for one known app: use `get_app_detail` directly, or the competitor-analysis skill.

## Inputs

Ask for these only when they matter and are missing:

| Input | Default | Notes |
|---|---|---|
| Goal | required | Turn it into 1–3 short search queries (2–3 words each). |
| Store | `all` | `ios`, `android` or `all`. |
| Apps or games | both | Maps to `type: "apps"` or `"games"`. |
| Category | none | `categories` takes store codes; the `search_apps` schema lists them. |
| Price | any | `free: true` for free apps only. |

## Tools

- `search_apps` — search and filter apps; sort by `installs`, `daily_installs`, `monthly_installs`, `monthly_revenue`, `released`, `updated`, `ratings_count` or `reviews_count`.
- `get_app_detail` — full profile of one app, including `ratingScore`.
- `get_app_historicals` — daily time series (`installs`, `revenue`, `ratings`, `reviews`, `score`).

## Workflow

1. `search_apps` with the main query, `sort: "installs"`, `limit: 20`. Add `search_in: ["title", "short_description"]` when the query is a feature, not a name.
2. `search_apps` with the same filters and `sort: "daily_installs"`: the apps growing now.
3. Optional: `sort: "released"` with `released_after` (YYYY-MM-DD) to find new entrants.
4. Remove off-topic rows (a query word can match an unrelated title). Keep 5–10 apps.
5. `get_app_detail` for each kept app: category, `ratingScore`, `ratingsCount`, price, last update.
6. For the 3 most interesting apps, `get_app_historicals` with `metrics: ["installs", "revenue"]` (the last 90 days by default) to confirm the trend.

Use `pagination.nextCursor` only when the first page is not enough. Each page is one call against the quota.

## Output

1. A table: app, store, developer, installs, daily installs, revenue (estimate), `ratingScore`, release date, link (`asospyUrl`).
2. "Growing now": the apps with the highest daily installs relative to total installs, and their 90-day trend.
3. Three short takeaways (for example, a crowded top, a subscription-heavy category, a recent entrant growing fast).

## Limitations

- iOS installs and all revenue are ASOSpy estimates. Revenue can be `null` (no estimate); do not read `null` as zero.
- Small apps show installs as text such as `"< 1k"`. Say so; do not turn it into a number.
- `search_apps` has no country filter; titles are the US listing.
- `min_installs`/`max_installs` do not work with `store: "ios"`. Use daily or monthly installs instead.
- Removed apps are hidden unless `include_removed: true`.
