---
name: growth-analysis
description: Explain how an app grew or declined over a period, using ASOSpy installs, revenue, ratings and chart-rank history, and find likely causes in reviews and release dates. Use when the user asks why an app's downloads, revenue or rank changed, or wants a growth timeline. Needs the ASOSpy MCP server.
---

# Growth analysis

## When to use

- "Why did X's downloads drop last month?", "how has X grown this year?", "when did X take off?".
- To check whether a launch, a price change or an update moved the numbers.

## Inputs

| Input | Default | Notes |
|---|---|---|
| App | required | Any identifier (id, store URL, ASOSpy URL, name). |
| Period | last 90 days | `date_from` / `date_to` as YYYY-MM-DD, at most 366 days per call. |
| Country | `us` | For chart ranks only. |

## Tools

`get_app_detail`, `get_app_historicals` (`metrics`: `installs`, `revenue`, `ratings`, `reviews`, `score`), `get_store_ranking_history`, `get_app_reviews`.

## Workflow

1. `get_app_detail`: current numbers, release date, last update, version.
2. `get_app_historicals` with `metrics: ["installs", "revenue", "ratings", "score"]` and the period. For more than 366 days, make one call per year-sized window.
3. `get_store_ranking_history` for the same period and country.
4. Find change points: days where daily installs, revenue or rank moved sharply and stayed moved (ignore one-day spikes).
5. `get_app_reviews` with `sort: "most_recent"` (add `sort: "lowest_rated"` if the rating fell) and read the reviews around each change point.
6. Link each change point to evidence: an update (`updatedAt`, versions in reviews), a rating change, a chart move, or complaints.

## Output

1. The overall trend in one paragraph: start value, end value, change in %.
2. A dated list of change points: date, metric, size of the change, likely cause, evidence.
3. Mark every cause as a hypothesis unless a review or detail field shows it directly.

## Limitations

- iOS installs and all revenue are ASOSpy estimates. The shape of the curve is more reliable than single days.
- Revenue can be missing for some days or months (no estimate); do not read gaps as zero.
- iOS ratings, reviews and score history come from the app's main storefront. Google Play review history is one storefront too.
- There is no data on marketing spend, featuring or press, so name these only as possibilities.
