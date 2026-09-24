---
name: competitor-analysis
description: Compare an app with its competitors on installs, revenue estimates, ratings, chart ranks and Apple Search Ads keywords, and turn the gaps into actions. Use when the user names an app and asks how it compares, who its competitors are, or what competitors do better. Needs the ASOSpy MCP server.
---

# Competitor analysis

## When to use

- "Compare my app with its competitors", "who are X's competitors", "how does X stack up against Y and Z".
- Before a positioning, pricing or ASO decision.

## Inputs

| Input | Default | Notes |
|---|---|---|
| App | required | ASOSpy id, App Store id, package name, store URL, ASOSpy URL or name. |
| Competitors | found automatically | 3–5 apps. |
| Country | `us` | Two letters; used for chart ranks. |

If a name is ambiguous, the tool returns `APP_NOT_FOUND` with `details.candidates`. Pick the right one (ask the user if unsure) and call again with its `id`.

## Tools

`get_app_detail`, `search_apps`, `get_app_historicals`, `get_store_ranking_history`, `search_ads` (`mode: "app_keywords"`, iOS only).

## Workflow

1. `get_app_detail` for the app: store, category, installs, revenue, `ratingScore`, `ratingsCount`, price model.
2. No competitors given: `search_apps` on the same store with 1–2 core words from the title or use case, the same category, `sort: "installs"`. Skip the app itself and clones. Keep 3–5.
3. `get_app_detail` for each competitor.
4. `get_app_historicals` for every app with `metrics: ["installs", "revenue", "score"]` (last 90 days).
5. `get_store_ranking_history` for every app with the country: best rank, current rank, stability.
6. iOS apps: `search_ads` with `mode: "app_keywords"` and `app: <id>` to see which Apple Search Ads keywords they appear on.

## Output

1. Comparison table: app, installs, daily installs, revenue (estimate), `ratingScore`, `ratingsCount`, best chart rank, 90-day installs change, last update.
2. For each competitor: two things it does better and one weakness, each backed by a number from the table.
3. Five actions for the user's app, most important first.

## Limitations

- iOS installs and all revenue are estimates; compare trends more than exact values.
- Chart ranks exist only while an app is in a chart; no rank is not the same as rank 0.
- Apple Search Ads data is a scraped sample (iPhone, short history, beta). No row does not prove an app does not advertise. There are no spend, impressions or creatives.
- `get_app_historicals` has no per-country split.
