---
name: ai-shadcn-component-generator
description: >-
  Use when the user wants to generate interactive React components using shadcn/ui design system.
  Describes a UI in plain English and receives live, exportable React code built from real shadcn primitives (cards, charts, forms, menus, layouts).
license: Apache-2.0
metadata:
  author: "Shubhamsaboo"
  version: "1.0.0"
  source: "https://github.com/Shubhamsaboo/awesome-llm-apps"
compatibility: >-
  Requires Node.js 18+, OpenAI API key, Tavily API key (optional).
  Uses pnpm monorepo with React + Vite frontend, Hono runtime, and FastAPI + LangGraph backend.
---

# AI Shadcn Component Generator (Shadify)

Describe a UI in plain English. Get a live, interactive [shadcn/ui](https://ui.shadcn.com/) component back. Export it as clean React code.

## Philosophy Mapping: Hands of Work + Mirror of Thought

This embodies **schema-driven composition** — the agent knows exactly which primitives exist (cards, charts, forms, menus, layouts), what props they take, and how they nest. The output is a structured tree streamed to the browser as real React components. This is "Hands of Work" manifested through design-system-aware generation rather than ad-hoc HTML.

## Features

- **Natural Language UI Description**: Describe any component in plain English
- **Live Interactive Preview**: Components render in real-time as you describe them
- **Exportable Code**: Clean, production-ready React code with shadcn/ui primitives
- **Schema-Guided Generation**: Full component schema passed as agent context ensures valid compositions
- **Accessible & Polished**: Every generated component uses the same accessible primitives as `npx shadcn add`

## Architecture

Three-service pnpm monorepo:

```
UI (React + Vite)  →  Runtime (Hono + CopilotKit)  →  Agent (FastAPI + LangGraph)
```

| Service | Path | Purpose |
|---------|------|---------|
| `ui` | `apps/ui` | Chat interface, component rendering, code export |
| `runtime` | `apps/runtime` | CopilotKit runtime, routes messages to agent |
| `agent` | `apps/agent` | LangGraph agent with search tools, returns structured UI |

## Tech Stack

- **Frontend**: React + Vite (via pnpm)
- **Runtime**: Hono + CopilotKit
- **Agent**: FastAPI + LangGraph
- **Design System**: shadcn/ui primitives (cards, charts, forms, menus, layouts)
- **Protocol**: AG-UI for agent ↔ UI communication

## Quick Start

```bash
git clone https://github.com/Shubhamsaboo/awesome-llm-apps.git
cd awesome-llm-apps/generative_ui_agents/ai-shadcn-component-generator
pnpm install
```

### Environment Configuration

```bash
# apps/runtime/.env
OPENAI_API_KEY=sk-...

# apps/agent/.env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
```

### Run Development Server

```bash
pnpm dev
# UI at http://localhost:5173
# Runtime on port 4000
# Agent on port 8123
```

## Deployment

Deploy all three services from a single `render.yaml` Blueprint. Push to main and Render wires service URLs together automatically via `fromService` references.

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy)

## When to Use This Skill

- Generating reusable React components for projects
- Prototyping UI designs with shadcn/ui design system
- Creating accessible, production-ready component code
- Building from a catalog of known primitives (cards, charts, forms, menus, layouts)

## When NOT to Use This Skill

- Non-React frameworks (Vue, Svelte, etc.)
- Backend-only logic without UI components
- When you need custom CSS rather than design-system primitives

## Integration with Hermes Agent

This skill enhances our capabilities:
- **Hands of Work**: Generates exportable React code for project integration
- **Mirror of Thought**: Schema-aware generation ensures valid component compositions
- **Teleology**: Purpose-driven UI generation aligned with design system constraints

## References

- [shadcn/ui Documentation](https://ui.shadcn.com/)
- [CopilotKit Documentation](https://docs.copilotkit.ai)
- [AG-UI Protocol](https://github.com/ag-ui-protocol/ag-ui)
- [LangGraph Documentation](https://www.langchain.com/langgraph)
