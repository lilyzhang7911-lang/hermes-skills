---
name: notion-mcp-agent
description: >-
  Use when the user wants to interact with Notion databases and pages using natural language via MCP (Model Context Protocol).
  Supports querying, creating, updating, and managing Notion content through a terminal or API interface.
license: Apache-2.0
metadata:
  author: "Shubhamsaboo"
  version: "1.0.0"
  source: "https://github.com/Shubhamsaboo/awesome-llm-apps"
compatibility: >-
  Requires Python 3.8+, OpenAI API key, and Notion API token (with database read/write scopes).
  Uses Agno framework with MCP Tools for Notion integration.
---

# Notion MCP Agent

Interact with Notion databases and pages using natural language through the Model Context Protocol (MCP). Query, create, update, and manage Notion content programmatically.

## Features

- **Natural Language Queries**: Ask questions about Notion pages and databases
- **Database Operations**: Read, filter, sort, and aggregate database entries
- **Page Management**: Create, update, and delete pages
- **MCP Integration**: Leverages Notion MCP Server for direct API access
- **Terminal Interface**: Command-line usage with optional Streamlit UI

## Setup

### Requirements

- Python 3.8+
- OpenAI API Key
- Notion API Token (with `read`, `write`, `users:read` scopes)

### Installation

```bash
git clone https://github.com/Shubhamsaboo/awesome-llm-apps.git
cd awesome-llm-apps/mcp_ai_agents/notion_mcp_agent
pip install -r requirements.txt
```

### Configuration

Set environment variables:
```bash
export OPENAI_API_KEY=your-openai-api-key
export NOTION_API_KEY=your-notion-token
```

Or use `.env` file:
```bash
cp .env.example .env
# Edit .env and add your Notion API key
```

### Running

**Terminal mode:**
```bash
python notion_mcp_agent.py [page_id]
```

**With Streamlit UI (if available):**
```bash
streamlit run app.py
```

## Usage Examples

**Query Pages:**
- "Show all pages in database [id]"
- "Find pages with status 'In Progress'"
- "List all tasks assigned to me"

**Create Content:**
- "Create a new page titled 'Meeting Notes' with content..."
- "Add a new entry to the database with title='Project X', status='Active'"

**Update Content:**
- "Update page [id] with new content: ..."
- "Change status of page [id] to 'Completed'"

**Filter and Sort:**
- "Show pages sorted by due_date descending"
- "Filter entries where priority is 'High' and status is 'Open'"

## Architecture

```
notion_mcp_agent/
├── notion_mcp_agent.py    # Main agent with Notion MCP integration
├── requirements.txt       # Python dependencies (agno, mcp)
└── README.md
```

## Key Components

1. **MCPTools**: Provides access to Notion MCP Server capabilities
2. **Agent (Agno)**: Handles natural language understanding and query formulation
3. **Notion API Integration**: Direct access to pages, databases, and users
4. **SQLite Cache** (optional): Local caching for frequently accessed data

## Notion API Scopes Required

- `read` - Read content from Notion
- `write` - Create and update content
- `users:read` - Access user information

## Limitations

- Requires valid Notion API token with appropriate scopes
- Rate limits apply (5 requests per second for free tier)
- Complex nested databases may require multiple queries
- File uploads not supported via MCP (use native API for attachments)

## References

- [Notion MCP Server](https://github.com/notionhq/notion-mcp-server)
- [Notion API Documentation](https://developers.notion.com/)