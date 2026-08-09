---
name: generative-ui-starter-project
description: >-
  Use when the user wants a chat-driven shared state pattern for UI components, or needs an A2UI (Agent-to-User Interface) starter.
  Implements bidirectional sync between agent state and React via useAgent(), with A2UI declarative rendering support.
license: Apache-2.0
metadata:
  author: "Shubhamsaboo"
  version: "1.0.0"
  source: "https://github.com/Shubhamsaboo/awesome-llm-apps"
compatibility: >-
  Requires Node.js 18+, Python 3.8+, OpenAI API key, uv (Python package manager).
  Uses Next.js + CopilotKit + AG-UI + LangGraph with A2UI specification support.
---

# Generative UI Starter Project (A2UI)

A chat-driven kanban board where you and the agent work the same task list. Built with CopilotKit, AG-UI, and LangGraph on top of Next.js. Also doubles as a starter for declarative gen UI via A2UI (Agent-to-User Interface).

## Philosophy Mapping: Mirror of Thought + Hands of Work

This embodies **shared state** — the board lives in the agent and syncs bidirectionally with React via `useAgent()`. Both sides observe and react to the same state. This is "Mirror of Thought" manifested as a shared mental model between human and AI, where changes on either side are immediately reflected on the other.

## Features

- **Shared Agent State**: Board (To Do / Done columns) lives in agent, syncs bidirectionally with React
- **A2UI Support**: Declarative UI generation — agent sends JSON descriptions of UI to render as real components
- **Two Patterns**: Fixed schema (pre-defined layout, changing data) and Dynamic schema (LLM generates both components and data)
- **Bidirectional Sync**: Agent moves cards through tool calls; user clicks, edits, and reorders in the UI
- **No Separate Frontend Store**: Both sides observe the same state — no manual sync needed

## Architecture

```
├── src/                         # Next.js frontend source
│   ├── app/page.tsx             # Main page
│   └── api/copilotkit/         # CopilotKit API route
│   ├── components/
│   │   ├── example-canvas/      # Todo list UI
│   │   ├── example-layout/      # Layout: chat + canvas side-by-side
│   │   └── generative-ui/       # Example generative UI components
│   └── hooks/
├── agent/                       # LangGraph Python agent
│   ├── main.py                  # Agent entry point
│   └── src/
│       ├── todos.py             # Todo tools and state schema
│       └── query.py             # Example data query tool
├── scripts/                     # Agent setup and run scripts
│   ├── setup-agent.sh / .bat
│   └── run-agent.sh / .bat
├── public/                      # Static assets
├── next.config.ts
├── tsconfig.json
└── package.json
```

## A2UI — Agent-to-User Interface

A2UI uses three concepts:

1. **Catalog** — component definitions (schema) paired with React renderers. Registered once in `layout.tsx` via `<CopilotKitProvider a2ui={{ catalog: demonstrationCatalog }}>`.
2. **Surface** — a rendered UI instance. The agent creates a surface, sets its components, and binds data to it.
3. **Operations** — the agent returns `a2ui.render(operations=[...])` from a tool, which the middleware streams to the frontend.

### Two Patterns

| Pattern | Description | Agent Tool | Frontend |
|---------|-------------|------------|----------|
| Fixed schema | Pre-defined component layout. Only data changes per invocation. | `search_flights` | Schema in `a2ui/schemas/flight_schema.json` |
| Dynamic schema | Secondary LLM generates both components and data based on conversation. | `generate_a2ui` | Components decided at runtime |

Both patterns use the same catalog on the frontend — the difference is where the component tree comes from.

### Adding a Custom Component

1. **Define** in `definitions.ts`:
   ```typescript
   MyWidget: {
     description: "A brief description for the agent.",
     props: z.object({ title: z.string(), value: z.number() }),
   },
   ```

2. **Render** in `renderers.tsx`:
   ```typescript
   MyWidget: ({ props }) => (
     <div>{props.title}: {props.value}</div>
   ),
   ```

3. **Use it** from the agent — automatically available to both fixed-schema templates and dynamic-schema LLM.

### Adding a New Fixed-Schema Tool

1. Create a JSON schema file in `agent/src/a2ui/schemas/` describing the component tree.
2. Create a Python tool that loads the schema with `a2ui.load_schema()` and returns `a2ui.render(operations=[...])`.

### Showcase Mode

`showcase.json` controls which suggestion pills are visually highlighted. Set `"showcase": "a2ui"` to highlight A2UI demos, or `"showcase": "default"` for no highlights.

## Quick Start

```bash
cd awesome-llm-apps/generative_ui_agents/generative-ui-starter-project
npm install  # Also installs Python agent deps via uv sync
cp .env.example .env
# Edit .env and add OPENAI_API_KEY=your-openai-api-key-here
npm run dev
# Starts both UI and agent servers concurrently
```

### Available Scripts

| Script | Description |
|--------|-------------|
| `dev` | Start both UI + agent (default) |
| `dev:debug` | Start with debug logging |
| `dev:ui` | Run just the Next.js app |
| `dev:agent` | Run just the LangGraph agent |
| `build / start` | Production build + server |

## Key Files

| Purpose | Path |
|---------|------|
| Catalog definitions (Zod schemas) | `src/app/declarative-generative-ui/definitions.ts` |
| Catalog renderers (React components) | `src/app/declarative-generative-ui/renderers.tsx` |
| Catalog registration | `src/app/layout.tsx` |
| Fixed-schema agent tool | `agent/src/a2ui_fixed_schema.py` |
| Dynamic-schema agent tool | `agent/src/a2ui_dynamic_schema.py` |
| Flight schema JSON | `agent/src/a2ui/schemas/flight_schema.json` |

## When to Use This Skill

- Building chat-driven UIs with shared agent state
- Prototyping A2UI patterns for declarative component generation
- Creating kanban boards, todo lists, or other collaborative UIs
- Learning the A2UI specification and CopilotKit integration patterns

## When NOT to Use This Skill

- Simple static pages without agent interaction
- Non-React frameworks
- When you don't need bidirectional state sync

## Integration with Hermes Agent

This skill enhances our capabilities:
- **Mirror of Thought**: Shared mental model between human and AI via bidirectional sync
- **Hands of Work**: Agent-generated UI components rendered in real-time
- **Teleology**: Purpose-driven UI generation aligned with user goals

## References

- [A2UI Specification](https://a2ui.org/specification/)
- [CopilotKit Documentation](https://docs.copilotkit.ai)
- [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)