---
name: ai-financial-coach-agent
description: >-
  Use when the user wants multi-agent financial coaching including budget analysis, savings planning, and debt-payoff strategies.
  Implements a tool-rendered component pattern where each chat turn routes to specialized agents (budget/savings/debt) with interactive UI cards in a report tab.
license: Apache-2.0
metadata:
  author: "Shubhamsaboo"
  version: "1.0.0"
  source: "https://github.com/Shubhamsaboo/awesome-llm-apps"
compatibility: >-
  Requires Node.js 18+, Python 3.12+, Google Makersuite API Key.
  Uses Next.js + CopilotKit + AG-UI + Google ADK for multi-agent financial analysis with interactive UI rendering.
---

# AI Financial Coach Agent

A multi-agent financial coach that analyzes your budget, plans your savings, and builds debt-payoff strategies — rendered as interactive UI cards in a separate report tab. Built with CopilotKit, AG-UI, and Google's ADK on top of Next.js.

## Philosophy Mapping: Teleology + Hands of Work

This embodies **purpose-driven financial planning** — the agent decomposes complex financial goals into actionable phases (Budget → Savings → Debt), routes each turn to the appropriate specialist tool, and renders progress as persistent UI cards. This is "Teleology" manifested through structured goal decomposition with continuous refinement.

## Features

- **Multi-Agent Routing**: Top-level coach dispatches to budget/savings/debt specialists
- **Interactive UI Cards**: Each tool call streams status pills into chat while materializing cards in report tab
- **Natural Language Input**: Update financial profile from plain English ("my income is $8k")
- **Phase-Based Analysis**: Run single phase or full Budget→Savings→Debt sequence
- **Persistent State**: Financial profile persists across conversation turns

## Architecture

```
[User: "Update my budget to $5000/month"]
        ↓
CopilotKit Runtime → Google ADK Agent (Router)
        ↓
Specialist Agents:
    ├── Budget Planner  → Update income/expenses
    ├── Savings Planner → Calculate optimal savings rate
    └── Debt Payoff     → Build debt elimination strategy
        ↓
Interactive UI Cards rendered in report tab
```

## Tech Stack

- **Frontend**: Next.js + CopilotKit + AG-UI Protocol
- **Backend**: Google ADK (Agent Development Kit)
- **State Management**: Persistent financial profile across turns
- **UI Rendering**: Tool-rendered components with status pills

## Quick Start

```bash
cd awesome-llm-apps/generative_ui_agents/ai-financial-coach-agent
npm install
npm run install:agent  # Install Python deps for ADK agent
export GOOGLE_API_KEY="your-google-api-key-here"
npm run dev
# Starts both UI and agent servers concurrently
```

### Available Scripts

| Script | Description |
|--------|-------------|
| `dev` | Start both UI + agent (default) |
| `dev:debug` | Start with debug logging |
| `dev:ui` | Run just the Next.js app |
| `dev:agent` | Run just the ADK agent |
| `build / start` | Production build + server |

## Financial Analysis Phases

| Phase | Description | Example Input |
|-------|-------------|---------------|
| Budget | Analyze income vs expenses | "My income is $8k/month" |
| Savings | Calculate optimal savings rate | "I want to save 20% of income" |
| Debt | Build debt-payoff strategies | "I have $15k in student loans" |

## When to Use This Skill

- Multi-phase financial planning with persistent state
- Interactive budget/savings/debt analysis with UI visualization
- Natural language financial profile updates
- Goal decomposition into actionable steps

## When NOT to Use This Skill

- Simple one-time calculations (use calculator directly)
- Real-time market data analysis
- Tax preparation or legal financial advice

## Integration with Hermes Agent

This skill enhances our capabilities:
- **Teleology**: Purpose-driven financial goal decomposition
- **Hands of Work**: Persistent state management across conversation turns
- **Mirror of Thought**: Multi-agent routing for specialized financial analysis

## References

- [Google ADK Documentation](https://google.github.io/adk-docs/)
- [CopilotKit Documentation](https://docs.copilotkit.ai)
- [Next.js Documentation](https://nextjs.org/docs)