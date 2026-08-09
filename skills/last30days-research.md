---
name: last30days-research
version: "1.0.0"
description: "Multi-source research engine for searching Reddit, X/Twitter, YouTube, TikTok, Polymarket, GitHub, and web sources in parallel. Synthesizes community engagement signals into actionable intelligence briefs."
author: "mvanhorn (adapted by Hermes)"
license: MIT
homepage: https://github.com/mvanhorn/last30days-skill
repository: https://github.com/mvanhorn/last30days-skill
tags: [research, multi-source, deep-research, reddit, twitter, youtube, tiktok, polymarket, github]
---

# Last30Days Research Engine

## Purpose
Search what people **actually say** about any topic in the last 30 days across multiple platforms. Aggregate engagement signals (upvotes, likes, comments, real-money bets) into one synthesized brief.

## Core Workflow

### Step 1: Resolve Host Web Search
Determine if agent session has web-search tool available. If yes, export `LAST30DAYS_NATIVE_SEARCH=1`. If no, skip Step 0.5/0.55 and add `--auto-resolve` to engine command.

### Step 2: First-Run Setup Wizard (One-Time)
Check for `SETUP_COMPLETE=true` in `~/.config/last30days/.env`:
- **Output `1`**: Setup complete, proceed to research
- **Output `FIRST_RUN_DETECTED`**: Run setup wizard before any research

Setup includes:
- Browser cookie extraction (X/Twitter auth)
- yt-dlp installation (YouTube)
- Digg CLI installation (AI 1000 leaderboard)
- ScrapeCreators API key (TikTok, Instagram, YouTube comments)

### Step 3: Pre-Research Handle Resolution
For person topics, resolve:
- X handles (@username)
- GitHub repos/usernames
- Subreddit names
- TikTok hashtags
- YouTube channel IDs

**Critical**: Named-entity topics REQUIRE `--plan` flag with JSON query plan. Never run bare engine call on named entities.

### Step 4: Parallel Source Search
Run `scripts/last30days.py` with appropriate flags:
```bash
python3 skills/last30days/scripts/last30days.py "topic" --emit=compact --plan "$QUERY_PLAN_FILE"
```

Sources searched in parallel:
- Reddit (with comments, upvote counts)
- X/Twitter (FROM/ABOUT lanes, interaction signals)
- YouTube (full transcripts + comments)
- TikTok (creator reach metrics)
- Polymarket (real-money odds)
- GitHub (PR velocity, star counts)
- Hacker News (developer consensus)
- Web search (editorial coverage)

### Step 5: Synthesis & Citation
Transform raw evidence into prose synthesis following LAWs:
1. **Badge**: `🌐 last30days v{VERSION} · synced {YYYY-MM-DD}` (first line)
2. **No Invented Titles**: Use `What I learned:` label
3. **No Section Headers**: Bold-lead-in paragraphs + numbered list
4. **Community Voice Weaving**: Include 2+ verbatim attributed comments
5. **Host-Split Citations**: Inline-link on hidden-link hosts, plain labels on visible-URL hosts

## Output Contract (LAWs)

### LAW 1: NO `Sources:` Block
Engine footer is the only citation. Do NOT append trailing Sources/References/Further reading blocks.

### LAW 2: NO INVENTED TITLES
Use `What I learned:` prose label, not section headers or invented titles.

### LAW 3: NO EM-DASHES
Use `-` instead of `—` or `–`. Only exception: quoted source content with em-dashes.

### LAW 4: NO SECTION HEADERS IN BODY
Bold-lead-in paragraphs + numbered list only. No `##` headers in synthesis body.

### LAW 5: ENGINE FOOTER PASS-THROUGH
Include verbatim emoji-tree footer (`✅ All agents reported back!`).

### LAW 6: NO RAW EVIDENCE CLUSTERS
Transform evidence into prose. Never emit raw `(score N, M items, sources: ...)` tuples.

### LAW 7: --plan IS MANDATORY ON NAMED ENTITIES
Generate JSON query plan first, pass to engine via `--plan "$QUERY_PLAN_FILE"`.

### LAW 8: HOST-SPLIT CITATIONS
- **Hidden-link** (Claude Code): `[text](url)` inline-links
- **Visible-URL** (Codex, Cursor, CLI): Plain labels `per @handle`, `per r/subreddit`

### LAW 9: WEAVE COMMUNITY VOICE
Include at least 2 verbatim attributed comments. Never narrate tooling behavior.

### LAW 10: FIRST-PARTY POSTS ARE FIRST-CLASS
Subject's own posts are primary signal. Don't lean on third-party coverage when first-party exists.

## Configuration

### Environment Variables
```bash
LAST30DAYS_MEMORY_DIR=~/Documents/Last30Days  # Output directory
LAST30DAYS_NATIVE_SEARCH=1                      # Use host web search (set if available)
SCRAPECREATORS_API_KEY=<key>                    # TikTok, Instagram, YouTube comments
XAI_API_KEY=<key>                               # X/Twitter search fallback
BSKY_HANDLE=<handle>                            # Bluesky search
BSKY_APP_PASSWORD=<password>                    # Bluesky auth
```

### Source Opt-In
```bash
INCLUDE_SOURCES=tiktok,instagram,youtube_comments,tiktok_comments,instagram_comments
# Or for everything: add threads,pinterest
```

## Hermes Integration Notes

### Installation
```bash
# Add to Hermes skills directory
cp -r /path/to/last30days-skill/skills/last30days ~/.hermes/skills/last30days-research/
```

### When to Use
- Pre-meeting research on people/companies/products
- Trend monitoring across social platforms
- Tool comparison with community engagement data
- Understanding public sentiment on topics

### Limitations
- Requires Python 3.12+
- X/Twitter needs browser cookies or API key
- YouTube needs yt-dlp installed
- TikTok/Instagram need ScrapeCreators API key (10k free calls)
- Network-dependent (GitHub, Reddit, Twitter APIs)

## Pitfalls & Corrections

### 文宁's Preference: Knowledge Over Tool Installation (2026-07-11)
**Signal**: User said "我们主要是沉淀思想和逻辑，至于技能，要看环境的" — prioritize思想/逻辑沉淀 over forcing tool installation when environment is blocked.

**Rule**: When a skill requires environment setup that is currently blocked:
1. Extract architecture patterns and knowledge → 沉淀 to Obsidian
2. Note environment constraints as blockers, not failures
3. Revisit skill activation only when constraints are resolved (e.g., network restored, Python upgraded)

### Environment Constraints (文宁's Setup)
- **翻墙不便**: Only browser-based翻墙 available, unstable — do NOT force X/Twitter + YouTube + TikTok setup
- **API Key需银行卡**: User declined to apply for API keys requiring bank card binding
- **Python 3.9 vs 3.12+**: Current system has Python 3.9.6, skill requires 3.12+

**Workaround**: Use free sources first (Reddit + HN + GitHub + Polymarket) which need no API keys. Reserve X/YouTube/TikTok for when network is stable.

### Python Version Gate
**Error**: `LAST30DAYS_PYTHON must point to Python 3.12+`
**Fix**: Install Python 3.12+: `brew install python@3.12` (macOS) or `sudo apt install python3.12` (Linux)

---

## Philosophy Alignment

### Mirror of Thought
- Aggregates **community signals** (what people actually engage with)
- Not editorial curation or SEO optimization
- Real engagement metrics (upvotes, likes, real-money bets)

### Hands of Work
- Parallel source search execution
- Engagement-based scoring algorithms
- Cross-platform cluster merging
- Structured output synthesis

### Teleology
- User provides topic/purpose
- AI executes perception → thought → intervention loop
- Delivers actionable intelligence briefs grounded in community data
