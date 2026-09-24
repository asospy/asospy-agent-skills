---
name: review-analysis
description: Summarise what users love and hate about an app from its App Store or Google Play reviews, with themes, frequencies, short quotes and version-specific issues. Use when the user asks about complaints, feature requests, user sentiment or reviews of an app. Needs the ASOSpy MCP server.
---

# Review analysis

## When to use

- "What do users complain about in X?", "what do people like about X?", "what are the top feature requests?".
- After a rating drop, to find the cause.
- To mine competitors' reviews for product ideas.

## Inputs

| Input | Default | Notes |
|---|---|---|
| App | required | Any identifier. |
| Country | all | Optional two-letter filter; many reviews have no country. |
| Focus | none | Optional topic, e.g. "pricing", "crashes", "sync". |

## Tools

- `get_app_reviews` — `sort`: `most_recent`, `highest_rated` or `lowest_rated`; `rating` 1–5 filter; `limit` up to 50 per page with `cursor`.
- `get_app_detail` — current version and rating.
- `get_app_historicals` — `metrics: ["score", "ratings"]` for the rating trend.

## Workflow

1. `get_app_detail`: current version, `ratingScore`, `ratingsCount`.
2. `get_app_reviews` with `sort: "lowest_rated"`, `limit: 50`, then one more page with `cursor`.
3. `get_app_reviews` with `sort: "highest_rated"`, `limit: 50`.
4. `get_app_reviews` with `sort: "most_recent"`, `limit: 50`.
5. Group the reviews into themes (bugs, pricing, missing features, UX, support, ads...). Count each theme and pick one short quote.
6. Check whether a theme is tied to one version or date.
7. `get_app_historicals` with `metrics: ["score", "ratings"]` to see whether the rating moved.

## Output

1. Top complaints and top praises: theme, count in the sample, one short quote (under 20 words).
2. Version- or date-specific issues.
3. Five prioritised fixes or product ideas.
4. State the sample size ("based on 200 reviews"). This is a sample, not every review.

## Limitations

- There are no developer replies and no "helpful" votes in the data.
- The sample is what the tool returns for each sort, not a statistically random sample.
- Quote briefly; do not paste long reviews.
