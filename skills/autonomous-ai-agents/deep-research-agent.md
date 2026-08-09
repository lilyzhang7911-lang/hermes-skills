---
name: deep-research-agent
description: >-
  Use when the user wants to perform deep, multi-step web research with structured planning, file writing, and live progress visualization.
  Combines CopilotKit + LangGraph for autonomous research loops with Tavily search integration.
license: Apache-2.0
metadata:
  author: "Shubhamsaboo"
  version: "1.0.0"
  source: "https://github.com/Shubhamsaboo/awesome-llm-apps"
compatibility: >-
  Requires Node.js 18+, Python 3.12+, OpenAI API key, Tavily API key.
  Uses Next.js frontend + FastAPI backend with LangGraph agent orchestration.
---

# AI Deep Research Agent

A deep research assistant that plans, searches the web, writes to a virtual filesystem, and renders each tool call as a live card in a workspace pane. Built with CopilotKit, Deep Agents, AG-UI, and Tavily on top of Next.js + LangGraph (Python).

## Philosophy Mapping: Mirror of Thought + Teleology

This agent embodies **perceive → think → act** loops with persistent state:
- **Perceive**: Internet search via Tavily
- **Think**: Planning (write_todos), filesystem operations (read/write)
- **Act**: Structured output with live progress visualization

The agent maintains a virtual filesystem across turns, enabling multi-step research that builds on previous findings — a concrete manifestation of "思之镜" (Mirror of Thought) with persistent context.

## Features

- **Multi-step Research Planning**: Decomposes complex questions into actionable subtasks
- **Web Search Integration**: Uses Tavily for comprehensive internet searches
- **Filesystem Persistence**: Writes research findings to a virtual filesystem
- **Live Progress Visualization**: Each tool call renders as a status card in the workspace pane
- **Thread-isolated Sub-agents**: Deep Agent runs with isolated context per thread

## Architecture

```
[User asks research question]
        ↓
Next.js Frontend (CopilotChat + Workspace)
        ↓
CopilotKit Runtime → LangGraphHttpAgent
        ↓
Python Backend (FastAPI + AG-UI)
        ↓
Deep Agent (research_assistant)
    ├── write_todos        (planning, built-in)
    ├── write_file         (filesystem, built-in)
    ├── read_file          (filesystem, built-in)
    └── research(query)
            └── internal Deep Agent [thread-isolated]
                    └── internet_search (Tavily)
```

## Tech Stack

- **Frontend**: Next.js + CopilotKit + AG-UI
- **Backend**: Python FastAPI + LangGraph
- **AI**: OpenAI GPT-5.5 (default) with Tavily search
- **Runtime**: CopilotKit Deep Agents for thread isolation

## Quick Start

### Frontend Setup
```bash
npm install
```

### Backend Setup
```bash
cd agent
uv venv && source .venv/bin/activate
uv pip install -e .
cd ..
# Or with pip:
python -m venv .venv && source .venv/bin/activate
pip install -e .
cd ..
```

### Environment Configuration
```bash
cp .env.example .env
# Fill in OPENAI_API_KEY and TAVILY_API_KEY
```

### Run
```bash
# Terminal 1: Start agent
cd agent && uv run python main.py

# Terminal 2: Start frontend
npm run dev
# Open http://localhost:3000
```

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `OPENAI_API_KEY` | Yes | - | [OpenAI API key](https://platform.openai.com/api-keys) |
| `TAVILY_API_KEY` | Yes | - | [Tavily API key](https://app.tavily.com/home) |
| `OPENAI_MODEL` | No | `gpt-5.5` | Model to use |
| `LANGGRAPH_DEPLOYMENT_URL` | No | `http://localhost:8123` | Backend URL |
| `SERVER_HOST` | No | `0.0.0.0` | Backend host |
| `SERVER_PORT` | No | `8123` | Backend port |

## Tool Schema

The Deep Agent exposes four tools:

| Tool | Purpose |
|------|---------|
| `write_todos` | Plan and track research subtasks |
| `write_file` | Persist findings to virtual filesystem |
| `read_file` | Retrieve previous research context |
| `research(query)` | Execute web search via Tavily |

## When to Use This Skill

- Complex multi-source research questions requiring iterative exploration
- Tasks that benefit from persistent state across multiple turns
- When live progress visualization helps track research progress
- Deep dives into topics requiring structured planning and file-based organization

## When NOT to Use This Skill

- Simple one-shot queries (use web_search directly)
- Real-time data fetching without persistence needs
- When the user wants immediate results without multi-step exploration

## Integration with Hermes Agent

This skill enhances our capabilities:
- **Mirror of Thought**: Persistent filesystem enables context retention across research turns
- **Teleology**: Autonomous planning + execution loop
- **Hands of Work**: Structured output via live workspace visualization

## References

- [Deep Agents Documentation](https://docs.copilotkit.ai/integrations/langgraph/deep-agents)
- [Building Frontends for Deep Agents](https://www.copilotkit.ai/blog/how-to-build-a-frontend-for-langchain-deep-agents-with-copilotkit)
- [CopilotKit Documentation](https://docs.copilotkit.ai)
- [Tavily Documentation](https://docs.tavily.com/welcome)