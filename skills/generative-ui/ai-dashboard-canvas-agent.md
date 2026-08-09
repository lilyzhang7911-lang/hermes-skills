---
name: ai-dashboard-canvas-agent
description: >-
  Use when the user wants to build interactive dashboards with live charts, metrics, and real-time data visualization.
  Implements an agentic canvas pattern where the agent writes into a persistent workspace — closer to a collaborator using a whiteboard than a chatbot returning replies.
license: Apache-2.0
metadata:
  author: "Shubhamsaboo"
  version: "1.0.0"
  source: "https://github.com/Shubhamsaboo/awesome-llm-apps"
compatibility: >-
  Requires Node.js 18+, Python 3.8+, Google Makersuite API Key.
  Uses Next.js + CopilotKit + AG-UI + Google ADK for dashboard generation and real-time data population.
---

# AI Dashboard Canvas Agent

An agent that populates **live charts, metrics, and real-time data** into a Canvas dashboard instead of just streaming text. Built with CopilotKit, AG-UI, and Google's ADK on top of Next.js.

## Philosophy Mapping: Hands of Work + Teleology

This embodies the **persistent workspace pattern** — the agent writes into a shared canvas that persists across turns. This is "Hands of Work" manifested as a collaborative whiteboard where charts, KPIs, and panels are addressable artifacts the agent can place, update, and rearrange. The dashboard becomes a living document rather than ephemeral chat output.

## Features

- **Live Data Visualization**: Real-time charts, metrics, and data panels
- **Persistent Canvas**: Dashboard state persists across conversation turns
- **Agent-Controlled Layout**: Agent places, updates, and rearranges visualization components
- **Real-time Updates**: Charts update dynamically as new data arrives
- **Collaborative Interface**: Closer to a whiteboard collaborator than a chatbot

## Architecture

```
[User requests dashboard analysis]
        ↓
Next.js Frontend (CopilotChat + Canvas)
        ↓
CopilotKit Runtime → Google ADK Agent
        ↓
Agent writes into Canvas:
    ├── Charts (line, bar, pie, area)
    ├── Metrics (KPIs, counters)
    └── Panels (tables, summaries)
```

## Tech Stack

- **Frontend**: Next.js + CopilotKit + AG-UI Protocol
- **Backend**: Google ADK (Agent Development Kit)
- **Visualization**: Dynamic chart components with real-time data binding
- **State Management**: Persistent canvas state across turns

## Quick Start

```bash
cd awesome-llm-apps/generative_ui_agents/ai-dashboard-canvas-agent
pnpm install        # or npm/yarn/bun
pnpm install:agent  # Install Python deps for ADK agent
cp .env.example .env
# Edit .env and set GOOGLE_API_KEY=...
pnpm run dev
```

### Available Scripts

| Script | Description |
|--------|-------------|
| `dev` | Start UI + agent (default) |
| `dev:debug` | Start with debug logging |
| `dev:ui` | Run just the Next.js app |
| `dev:agent` | Run just the ADK agent |
| `build / start` | Production build + server |

## Customization Points

- **Main UI**: `src/app/page.tsx` — Change theme/colors and sidebar appearance
- **Visualizations**: Add new chart components
- **Agent Logic**: Extend `/agent` directory for custom analysis patterns

## When to Use This Skill

- Building interactive dashboards with live data
- Creating persistent analytical workspaces that evolve across turns
- Visualizing metrics, KPIs, and real-time data streams
- Collaborative analysis where the agent builds upon previous visualizations

## When NOT to Use This Skill

- Simple text-based analysis (use standard chat)
- One-off reports without persistence needs
- When you don't have a Google Makersuite API key

## Integration with Hermes Agent

This skill enhances our capabilities:
- **Hands of Work**: Persistent canvas enables ongoing analytical collaboration
- **Teleology**: Purpose-driven dashboard building with continuous refinement
- **Mirror of Thought**: Visual thinking through interactive data exploration

## References

- [Google ADK Documentation](https://google.github.io/adk-docs/)
- [CopilotKit Documentation](https://github.com/CopilotKit/CopilotKit)
- [AG-UI Protocol](https://docs.ag-ui.com/introduction)