---
name: aso-audit
description: Audit an App Store or Google Play listing — title, subtitle or short description, description, keyword coverage, category, ratings, reviews and chart ranks — and return a checklist with prioritised fixes. Use when the user asks for an ASO audit, a listing review or how to rank better. Needs the ASOSpy MCP server.
---

# ASO audit

## When to use

- "Audit my app's store listing", "how can X rank higher", "review my ASO".
- Before rewriting a title, subtitle or description.

## Inputs

| Input | Default | Notes |
|---|---|---|
| App | required | Any identifier. |
| Country | `us` | The storefront to audit (listing text and chart ranks). |

## Tools

`get_app_detail`, `search_keywords`, `get_store_ranking_history`, `get_app_reviews`, `search_apps`, `search_ads` (`mode: "app_keywords"`, iOS only).

## Workflow

1. `get_app_detail` with `country`: title, subtitle / short description, description, category, `ratingScore`, `ratingsCount`, last update, size, price.
2. Take 3–5 core terms from the title and subtitle. `search_keywords` for each: are they terms with real demand (`applePopularity`, `volume`)?
3. `search_keywords` for the app's core use case (2–3 seeds) to find strong keywords the listing does not use.
4. `search_apps` for the top 3 apps in the same category (`sort: "installs"`) and compare their titles and subtitles with `get_app_detail`.
5. `get_store_ranking_history` with `country`: where the app charts and how stable it is.
6. `get_app_reviews` with `sort: "lowest_rated"`, `limit: 50`: problems that pull the rating down.
7. iOS: `search_ads` with `mode: "app_keywords"`: keywords the app already appears on in Apple Search Ads.

## Output

A checklist. For each area give a status (good / improve / missing), the finding, and the evidence:

- Title (length, main keyword, brand)
- Subtitle (iOS) or short description (Android)
- Description (first lines, keyword use, readability)
- Keyword coverage (strong keywords used vs. missing)
- Category fit
- Ratings and reviews (score, volume, recurring complaints)
- Chart ranks (where, how stable)
- Update frequency

Then the 5 most important fixes in order, each with the expected benefit.

## Limitations

- The audit cannot see screenshots, the icon, the preview video or the iOS hidden keyword field. Say so rather than guessing.
- Keyword metrics: `applePopularity` is US App Store only; Google figures are not store search counts.
- Conversion rate, impressions and page views are not in the data.
