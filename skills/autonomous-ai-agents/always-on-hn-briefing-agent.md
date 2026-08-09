---
name: always-on-hn-briefing-agent
description: >-
  Use when the user wants to monitor Hacker News for AI/ML/coding agent news and generate structured daily briefings.
  Implements an always-on scheduler that scans HN front page, filters for relevant stories, ranks by signal, and delivers formatted briefs via email/webhook.
license: Apache-2.0
metadata:
  author: "Shubhamsaboo"
  version: "1.0.0"
  source: "https://github.com/Shubhamsaboo/awesome-llm-apps"
compatibility: >-
  Requires Python 3.10+, Gemini API key (for ADK Web), optional Gmail OAuth for email delivery, optional webhook URL for Slack/Linear/Jira integration.
---

# Always-on Hacker News Briefing Agent (AgentScout)

An always-on Hacker News briefing agent built with Google ADK that scans HN for high-signal stories about AI agents, MCP, coding agents, workflow automation, and LLM apps — then turns the best links into a concise engineering brief.

## Philosophy Mapping: Teleology (目的论)

This is a **purpose-driven autonomous loop**: perceive (scan HN) → think (filter + rank) → intervene (deliver briefing). The agent operates continuously without explicit user prompting, embodying the "action freely" (行动自如) autonomy standard. It transforms raw signal into structured intelligence on a schedule.

## Features

- **Hacker News monitoring**: Finds AI agent, MCP, coding agent, automation, and LLM app stories from Hacker News
- **Signal ranking**: Scores stories by relevance, points, comments, and front-page position
- **Brief generation**: Produces clean text and HTML briefings with summaries, links, and next actions
- **Google ADK agent**: Exposes a `root_agent` so users can request briefs in ADK Web
- **Scheduler-ready backend**: Includes HTTP and Pub/Sub endpoints for Cloud Scheduler or other automation systems
- **Gmail and webhook delivery**: Sends briefs through Gmail API or a generic webhook (Slack, Linear, Jira, GitHub Issues, SendGrid)
- **Safe delivery flow**: Defaults to dry-run mode; skips delivery unless credentials are explicitly configured

## How It Works

1. AgentScout collects stories from deterministic sample data or the live Hacker News front page
2. Filters for AI agent and LLM app topics using keyword matching (agent, agents, agentic, automation, autonomous, coding, framework, llm, mcp, orchestration, tool, workflow)
3. Ranks the most useful stories for engineers and product builders
4. Renders a daily briefing in text and HTML
5. ADK Web, an HTTP trigger, or a Pub/Sub push endpoint returns the result
6. If delivery is enabled, the scheduler API sends the brief through Gmail or posts to `AGENTSCOUT_WEBHOOK_URL`

## Architecture

```
always_on_hn_briefing_agent/
├── scout.py              # HN front page parser + story extraction
├── agent.py              # ADK root agent for interactive queries
├── scheduler_api.py      # HTTP/Pub/Sub endpoints for Cloud Scheduler
├── delivery.py           # Email and webhook delivery
├── requirements.txt
└── README.md
```

## Tech Stack

- **Backend**: Python 3.10+, Google ADK, FastAPI
- **AI**: Gemini (via ADK) for story analysis and brief generation
- **Delivery**: Gmail API, generic webhooks (Slack, Linear, Jira, GitHub Issues, SendGrid)
- **Scheduling**: Cloud Scheduler, HTTP triggers, Pub/Sub

## Quick Start

```bash
git clone https://github.com/Shubhamsaboo/awesome-llm-apps.git
cd awesome-llm-apps/always_on_agents/always_on_hn_briefing_agent
pip install -r requirements.txt
export GOOGLE_API_KEY="your_gemini_api_key"
```

### Option 1: ADK Web (interactive)
```bash
adk web .
# Open ADK Web UI and select always_on_hn_briefing_agent
```

### Option 2: Scheduled Backend
```bash
# Set up environment
cp .env.example .env
# Edit .env with your API keys

# Run the scheduler
python scheduler_api.py
```

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/brief` | Generate and return a briefing |
| `POST` | `/api/deliver` | Trigger delivery via webhook/email |
| `GET` | `/health` | Health check |

## Configuration

- **Dry-run mode**: Defaults to dry-run; set `DELIVERY_ENABLED=true` for actual delivery
- **Webhook URL**: Set `AGENTSCOUT_WEBHOOK_URL` for Slack/Linear/Jira integration
- **Gmail OAuth**: Optional — configure for direct email delivery

## Keyword Filtering

The scout uses these agent-related keywords:
```python
AGENT_KEYWORDS = {
    "agent", "agents", "agentic", "automation", "autonomous",
    "coding", "framework", "llm", "mcp", "orchestration",
    "tool", "workflow"
}
NOISE_WORDS = {"ask hn: who is hiring", "freelance", "hiring"}
```

## Integration with Hermes Agent

This skill complements our existing capabilities:
- **Mirror of Thought**: The agent perceives (HN scanning) and thinks (filtering/ranking) autonomously
- **Teleology**: Purpose-driven continuous operation without explicit prompting
- **Hands of Work**: Delivers structured briefings via multiple channels (email, webhook, chat)

## When NOT to Use This Skill

- One-time HN lookups (use web_search instead)
- Non-AI/ML topic monitoring
- When the user wants manual control over briefing content

Use this skill when you need continuous, automated monitoring of Hacker News for AI/agent-related developments with structured output delivery.