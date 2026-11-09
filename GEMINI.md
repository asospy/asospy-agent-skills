# ASOSpy

ASOSpy gives you App Store and Google Play market data through the `asospy` MCP server: app search, app profiles with installs and revenue estimates, install and rating history, top charts and chart rank history, keyword research, a live keyword measurement, user reviews, and app-ads.txt and Apple Search Ads lookups.

When you use it:
- `store` is "ios" (App Store), "android" (Google Play) or "all". iOS app ids are numeric; Android ids are package names.
- `country` is a two-letter code such as "us"; `get_supported_countries` lists them.
- iOS installs and all revenue figures are ASOSpy estimates; say so when you report them.
- Lists are paged: call again with `cursor` = `pagination.nextCursor` while `hasMore` is true.
- Errors come back as `{"error": {"code", "message", "hint"}}`; follow the hint.
- Every tool call counts toward the workspace's monthly ASOSpy quota, so do not repeat calls you do not need.

For full analyses, use the skills in `skills/`: app-discovery, keyword-research, competitor-analysis, growth-analysis, review-analysis and aso-audit.
