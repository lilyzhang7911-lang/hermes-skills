---
name: ai-mcp-app-builder
description: >-
  Use when the user wants to describe an MCP app in chat and get a live, sandboxed instance back.
  The agent authors a brand-new MCP app at runtime, provisions an E2B sandbox to host it, and renders inline with full bidirectional tool access.
license: Apache-2.0
metadata:
  author: "Shubhamsaboo"
  version: "1.0.0"
  source: "https://github.com/Shubhamsaboo/awesome-llm-apps"
compatibility: >-
  Requires Node.js 20+, pnpm, OpenAI API key (OPENAI_API_KEY), optional E2B_* keys for sandbox provisioning.
  Uses Next.js + CopilotKit + AG-UI + Mastra agent + E2B sandboxes.
---

# AI MCP App Builder

Describe an MCP app in chat and get a live, sandboxed instance back. Built with CopilotKit, AG-UI, Mastra, and E2B sandboxes.

## Philosophy Mapping: Teleology + Hands of Work

This is the **ultimate "Hands of Work" pattern** — the agent doesn't just pick from a fixed catalogue of components; it *authors a brand-new MCP app at runtime*. The "component" the agent emits *is a whole app*. This represents the highest level of autonomous capability: generating new tools rather than using existing ones.

## Features

- **Agent-Generated Apps**: Describe any MCP app in natural language, get a live sandboxed instance
- **E2B Sandbox Provisioning**: Each generated app runs in an isolated E2B sandbox
- **Bidirectional Tool Access**: The generated app can call tools back to the agent
- **Dynamic MCP UI Sidebar**: Add/remove MCP servers by URL with built-in defaults (Excalidraw)
- **Mobile/Desktop Layouts**: Responsive design with chat + tools tabs on mobile, sidebar layout on desktop

## Architecture

```
[User describes MCP app]
        ↓
Next.js Web UI (CopilotKit v2 chat)
        ↓
Mastra Agent (/api/mastra-agent)
        ↓
E2B Sandbox Provisioning
        ↓
mcp-use-server Template (running in sandbox)
        ↓
Inline rendering with full bidirectional tool access
```

## Tech Stack

- **Frontend**: Next.js + CopilotKit v2 + AG-UI Protocol
- **Agent**: Mastra framework
- **Sandboxing**: E2B (E2B.dev) for isolated runtime environments
- **Workspace**: pnpm monorepo with Turbo

## Quick Start

```bash
cd awesome-llm-apps/generative_ui_agents/ai-mcp-app-builder
pnpm i
Copy-Item .env.example .env
# Edit .env: set OPENAI_API_KEY=sk-proj-... at minimum; add E2B_* for sandbox provisioning
pnpm dev
```

### Run Pieces Individually

| Goal | Command |
|------|---------|
| Web app only | `pnpm --filter web dev` or `cd apps/web && pnpm dev` |
| Three.js MCP sample (local sidebar default) | `cd apps/threejs-server && pnpm dev` |
| mcp-use-server (local MCP, not E2B image) | `cd apps/mcp-use-server && pnpm dev` |

## Environment Variables (E2B)

| Variable | Description |
|----------|-------------|
| `E2B_API_KEY` | From [e2b.dev/dashboard](https://e2b.dev/dashboard) |
| `E2B_TEMPLATE` | `templateId` from `Template.build` output after `build.dev.ts` / `build.prod.ts` |
| `E2B_REPO_URL` | Used when `E2B_TEMPLATE` is empty — clones repo into sandbox (slower cold start). Default: `mcp-use-server-template` GitHub URL |

## Dynamic MCP UI (Sidebar)

- **MCP Servers**: Add/remove by URL (+ optional `serverId`); list sent as `x-mcp-servers`. Built-in default: Excalidraw (`https://mcp.excalidraw.com`). Override via `NEXT_PUBLIC_DEFAULT_MCP_SERVERS` / `DEFAULT_MCP_SERVERS`.
- **Tools**: Compact list; open a tool for detail + preview in a modal (not a third mobile tab).
- **Chat**: CopilotKit v2 chat with suggestions.

### Mobile Layout

- **Tabs**: Chat and Tools (servers + tool list). Tool preview/detail opens in a modal.
- **Desktop**: Sidebar + chat column (`md+`).
- **Chat UX**: Spacing and bottom padding so the composer does not cover the latest messages.

## When to Use This Skill

- Generating custom MCP apps on-the-fly for specific tasks
- Needing isolated sandboxed environments for app execution
- Building dynamic tool ecosystems where new capabilities are generated at runtime
- Prototyping new agent capabilities without manual server setup

## When NOT to Use This Skill

- Using pre-built, static MCP servers (use those directly)
- When you don't need sandbox isolation
- Simple one-shot tasks that don't require app generation

## Integration with Hermes Agent

This skill represents the pinnacle of "Hands of Work" — generating new capabilities rather than using existing ones. It complements our existing skills by enabling autonomous tool creation.

## References

- [CopilotKit Documentation](https://docs.copilotkit.ai)
- [Next.js Documentation](https://nextjs.org/docs)
- [MCP Apps / UI](https://mcpui.dev/guide/introduction)