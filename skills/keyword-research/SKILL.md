---
name: keyword-research
description: Build an App Store / Google Play keyword shortlist around a seed term, with ASOSpy's keyword metrics (Apple popularity, Google volume, CPC, competition) and the apps that rank for each keyword. Use when the user asks for keywords, search terms or ASO keyword ideas for an app or niche. Needs the ASOSpy MCP server.
---

# Keyword research

## When to use

- "Find keywords for my meditation app", "what do people search for around X", "which keywords should I target".
- Before an ASO audit or a title/subtitle rewrite.
- To size demand for a niche (pair with app-discovery).

## Inputs

| Input | Default | Notes |
|---|---|---|
| Seed | required | One short term or phrase. Run several seeds for broad topics. |
| Store | `all` | Picks which store's top apps each keyword row shows. |
| App | none | Optional: the user's app, to check which keywords it already uses. |

## Tools

- `search_keywords` — related keywords with metrics; sort by `apple_popularity`, `volume`, `cpc`, `competition` or `keyword`; filters `contains`, `min_/max_apple_popularity`, `min_/max_volume_score`, `min_/max_cpc`, `min_/max_competition`.
- `search_niches` — the seed's market size (apps, developers, installs) plus related keywords.
- `get_app_detail` — the user's app title, subtitle / short description and description.
- `live_keyword_metrics` — one keyword measured live: `source: "apple"` for Apple popularity in a storefront, `source: "google"` for Google volume, CPC and competition. At most 10 per minute per workspace.

How `search_keywords` matches the seed: a word of up to 4 letters (for example `vpn`) matches only exactly, with no typo matching. A 5-letter word also matches with 1 typo, and a longer word with up to 2.

## Workflow

1. `search_keywords` with `query: <seed>`, `sort: "apple_popularity"`, `limit: 50`.
2. `search_keywords` again with `sort: "volume"`, `limit: 50`.
3. For long-tail ideas, `search_keywords` with `query: <seed>`, `contains: <seed>` and `sort: "apple_popularity"`.
4. Merge, drop duplicates and off-topic terms. Keep 15–25 keywords.
5. Classify: head terms (high popularity), long-tail terms (specific, lower popularity), brand terms (a competitor's name — usually not worth targeting).
6. If an app was given: `get_app_detail`, then mark each keyword as used in the title, used in the subtitle / short description, or missing.
7. Optional: `search_niches` with the seed for the market size.
8. Optional: `live_keyword_metrics` for the 3–5 shortlisted keywords that matter most when a figure is missing or looks stale, or when the user targets a storefront other than the US (`source: "apple"`, `storefront`). `measured: false` means the value is unknown, not 0.

## Output

1. A table: keyword, `applePopularity`, `volume`, `competition`, `cpc`, top ranking apps, type (head / long-tail / brand), used by the app (if given).
2. The 5 keywords to target first, each with one line on why (demand vs. how strong the top apps are).
3. Keywords to avoid and why.

## Limitations

- `applePopularity` from `search_keywords` is Apple Search Ads popularity for the **US** App Store only; `live_keyword_metrics` can measure another storefront, one keyword at a time. `volume`, `cpc` and `competition` are Google Ads figures, not store search counts.
- `competitionLevel` is `null` when Google has no data for the keyword; that means "unknown", not "low".
- The keyword list has no per-country split, and the top apps shown are from the US store.
- The backend works on the 10,000 highest-volume matches for a seed, so a very broad seed can hide rare long-tail terms. Use narrower seeds or `contains`.
- Seed words of up to 4 letters get no typo matching, so check their spelling.
- A `live_keyword_metrics` lookup that Apple or Google answers with an error (`details.reason` `rate_limited` or `upstream_error`) still uses its quota unit; other failures, including the 10-per-minute limit, are free.
