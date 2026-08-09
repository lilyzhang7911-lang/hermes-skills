---
name: github-mcp-agent
description: >-
  Use when the user wants to interact with GitHub repositories using natural language via MCP (Model Context Protocol).
  Supports querying issues, pull requests, repository activity, and code analysis through a Streamlit interface.
license: Apache-2.0
metadata:
  author: "Shubhamsaboo"
  version: "1.0.0"
  source: "https://github.com/Shubhamsaboo/awesome-llm-apps"
compatibility: >-
  Requires Python 3.8+, OpenAI or Anthropic API key, and GitHub Personal Access Token.
  Uses Streamlit for UI, Agno framework, and MCP Tools for GitHub integration.
---

# GitHub MCP Agent

Explore GitHub repositories with natural language using the Model Context Protocol (MCP). Query issues, pull requests, repository activity, and perform code analysis through an intuitive Streamlit interface.

## Features

- **Natural Language Queries**: Ask questions about repositories in plain English
- **Issue Management**: Find issues by label, status, or discussion activity
- **PR Tracking**: View recent merged PRs, pending reviews, and merge status
- **Repository Analytics**: Analyze code quality trends, activity patterns, and health metrics
- **MCP Integration**: Leverages GitHub MCP Server for direct API access

## Setup

### Requirements

- Python 3.8+
- OpenAI or Anthropic API Key
- GitHub Personal Access Token (with repo scope)

### Installation

```bash
git clone https://github.com/Shubhamsaboo/awesome-llm-apps.git
cd awesome-llm-apps/mcp_ai_agents/github_mcp_agent
pip install -r requirements.txt
```

### Configuration

Set environment variables:
```bash
export OPENAI_API_KEY=your-openai-api-key
export GITHUB_TOKEN=your-github-token
```

Or use `mcp_agent.secrets.yaml`:
```yaml
openai:
  api_key: your-openai-api-key
github:
  token: your-github-token
```

### Running

```bash
streamlit run github_agent.py
```

## Usage Examples

**Issues:**
- "Show me issues labeled as bugs"
- "What issues are being actively discussed?"
- "Find all open issues with label 'enhancement'"

**Pull Requests:**
- "What PRs need review?"
- "Show me recent merged PRs"
- "List pending pull requests"

**Repository Activity:**
- "Show repository health metrics"
- "Analyze code quality trends"
- "Display recent commit activity"

## Architecture

```
github_mcp_agent/
├── github_agent.py      # Streamlit app with GitHub MCP integration
├── requirements.txt     # Python dependencies (agno, streamlit)
└── README.md
```

## Key Components

1. **MCPTools**: Provides access to GitHub MCP Server capabilities
2. **Agent (Agno)**: Handles natural language understanding and query formulation
3. **Streamlit UI**: Interactive interface for querying repository data
4. **GitHub API Integration**: Direct access to issues, PRs, and repository metadata

## Query Types

| Type | Description | Example |
|------|-------------|---------|
| Issues | Filter by label, status, assignee | "Show bug issues" |
| Pull Requests | View merged/review status | "Recent merged PRs" |
| Repository Activity | Analytics and trends | "Code quality metrics" |
| Custom | Free-form natural language queries | Any GitHub-related question |

## Limitations

- Requires valid GitHub token with appropriate scopes
- API rate limits apply based on authentication method
- Complex multi-step operations may require multiple queries

## References

- [GitHub MCP Server](https://github.com/github/github-mcp-server)
- [Agno Framework](https://docs.agno.com/)