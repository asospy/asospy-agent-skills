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

You need an ASOSpy workspace with API access and an **MCP key** (it starts with `asp_mcp_`). Create one in ASOSpy under **Account > API keys > MCP keys**. MCP keys are separate from the REST API keys.

- Server URL: `https://mcp.asospy.dev/api` (Streamable HTTP)
- Header: `Authorization: Bearer <your MCP key>`

Claude Code:

```bash
export ASOSPY_API_KEY="asp_mcp_..."
claude mcp add --transport http --scope user asospy https://mcp.asospy.dev/api \
  --header "Authorization: Bearer ${ASOSPY_API_KEY}"
```

Setup for Cursor, Codex, Claude Desktop, Gemini CLI, VS Code, Cline, Continue and Devin Desktop: <https://asospy.dev/mcp#setup>.

## 2. Install the skills

Copy the folders you want from `skills/` into your agent's skills directory:

```bash
git clone https://github.com/asospy/asospy-agent-skills.git
# Claude Code, for every project:
cp -R asospy-agent-skills/skills/* ~/.claude/skills/
# or for one project:
mkdir -p .claude/skills && cp -R asospy-agent-skills/skills/* .claude/skills/
```

Other agents that support `SKILL.md` skills load them from their own skills folder. For agents without skills support, paste the body of a `SKILL.md` into the chat or the system prompt.

## 3. Use them

Ask in plain words, for example:

- "Find meditation apps that are growing fast on iOS."
- "Do keyword research for a sleep sounds app."
- "Compare Calm with Headspace and two other competitors."
- "Why did com.whatsapp's installs change in the last 60 days?"
- "What do users complain about in Duolingo's reviews?"
- "Run an ASO audit of https://apps.apple.com/app/id571800810."

The server also has built-in prompts with the same names (`app_discovery`, `keyword_research`, `competitor_analysis`, `growth_analysis`, `review_analysis`, `aso_audit`, and `market_research`). Clients that show MCP prompts list them in their prompt or slash-command menu.

## What the data is (and is not)

- **iOS installs and all revenue figures are ASOSpy estimates.** Apple and Google do not publish them. Revenue is `null` when there is no estimate.
- `ratingsCount` is a number of ratings; `ratingScore` is the 1–5 average.
- `applePopularity` is Apple Search Ads popularity for the **US** App Store. `volume`, `cpc` and `competition` are Google Ads figures.
- Ads data covers app-ads.txt publisher lines and scraped Apple Search Ads appearances (iOS, beta). There are no ad creatives, spend or impressions.
- Every call counts against your workspace's monthly API quota, and the plan's per-minute limit applies.

## Tools used by the skills

`search_apps`, `get_app_detail`, `get_app_historicals`, `list_store_rankings`, `get_store_ranking_history`, `search_keywords`, `get_app_reviews`, `search_ads`, `get_supported_countries`, `search_niches`. Every tool is read-only. Your MCP client shows each tool's full argument list.
