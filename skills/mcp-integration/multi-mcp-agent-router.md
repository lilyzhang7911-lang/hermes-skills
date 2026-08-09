---
name: multi-mcp-agent-router
description: >-
  Use when the user needs specialized agents with domain-specific MCP tool routing.
  Implements a router pattern that classifies queries and dispatches to specialist agents (code reviewer, security auditor, researcher, BIM engineer), each with access only to their relevant MCP servers.
license: Apache-2.0
metadata:
  author: "Shubhamsaboo"
  version: "1.0.0"
  source: "https://github.com/Shubhamsaboo/awesome-llm-apps"
compatibility: >-
  Requires Python 3.10+, Anthropic API key, optional MCP servers (app works without them).
  Uses Streamlit for UI, Claude via Anthropic API, and stdio-based MCP client connections.
---

# Multi-MCP Agent Router (Agent Forge)

A Streamlit app that demonstrates the **multi-agent + MCP** pattern: specialized AI agents that each connect to different MCP servers to handle domain-specific tasks. Instead of one agent with all tools, the router sends your request to a **specialist**.

## Philosophy Mapping: Mirror of Thought + Hands of Work

This embodies the **router → specialist** pattern — a concrete manifestation of "思之镜" (Mirror of Thought) where perception (query classification) routes to the appropriate "hand" (specialized agent). Each agent has its own system prompt and tool access, demonstrating the principle that different tasks require different capabilities.

## Features

- **4 Specialized Agents**: Code Reviewer, Security Auditor, Researcher, BIM Engineer
- **MCP Tool Routing**: Each agent connects to different MCP servers based on domain expertise
- **Automatic Classification**: Router classifies intent and selects best agent
- **Manual Agent Selection**: User can override automatic routing
- **Streaming Responses**: Real-time output from Claude via Anthropic API
- **Conversation Memory**: Per-agent conversation history within a session

## Architecture

```
User Query
    |
    v
[Router] --> Classifies intent
    |
    +-- Code Review  --> GitHub MCP + Filesystem MCP
    +-- Security     --> GitHub MCP + Fetch MCP  
    +-- Research     --> Fetch MCP + Filesystem MCP
    +-- BIM/Revit    --> Custom MCP (named pipes)
```

## Tech Stack

- **Frontend**: Streamlit UI with conversation history display
- **Backend**: Python 3.10+, Anthropic Claude API, MCP stdio client
- **MCP Servers**: GitHub, Filesystem, Fetch, Custom (BIM/Revit)

## Quick Start

```bash
git clone https://github.com/Shubhamsaboo/awesome-llm-apps.git
cd awesome-llm-apps/mcp_ai_agents/multi_mcp_agent_forge
pip install -r requirements.txt
streamlit run agent_forge.py
```

### Configuration

Set environment variable:
```bash
export ANTHROPIC_API_KEY=your-anthropic-api-key
```

## Agent Definitions

| Agent | Domain | MCP Servers | Use Case |
|-------|--------|-------------|----------|
| `code_reviewer` | Code quality, bugs, anti-patterns | GitHub + Filesystem | Review PRs, analyze codebases |
| `security_auditor` | OWASP Top 10, injection, XSS, secrets | GitHub + Fetch | Security scanning, vulnerability assessment |
| `researcher` | Web research, data gathering | Fetch + Filesystem | Information retrieval, content extraction |
| `bim_engineer` | Building information modeling | Custom MCP (named pipes) | Revit/BIM analysis |

## Extending with New Agents

Add new agents by defining them in the `AGENTS` dictionary:

```python
AGENTS["my_agent"] = Agent(
    name="My Agent",
    description="Handles X tasks",
    system_prompt="You are an expert in X...",
    mcp_servers=[{"command": "npx", "args": ["-y", "@some/mcp-server"]}]
)
```

## When to Use This Skill

- Multi-domain tasks requiring different tool access patterns
- When you need specialized analysis (code review vs security audit vs research)
- Tasks that benefit from agent isolation and conversation memory per domain
- Demonstrating the multi-agent + MCP routing pattern

## When NOT to Use This Skill

- Simple single-domain queries (use a single agent directly)
- When you don't have an Anthropic API key
- Non-AI tasks that don't require LLM reasoning

## Integration with Hermes Agent

This skill complements our capabilities:
- **Mirror of Thought**: Router pattern enables intelligent query classification and routing
- **Hands of Work**: Each specialist agent has targeted tool access for efficient execution
- **Teleology**: Purpose-driven dispatch ensures the right agent handles each task type

## References

- [Agent Forge (cadre-ai)](https://github.com/WeberG619/cadre-ai) — Production multi-agent framework with 17 specialized agents, persistent memory, and desktop automation