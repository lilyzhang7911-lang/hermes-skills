---
name: diagram-generation-from-natural-language
description: Generate architecture diagrams, flowcharts, and visualizations from natural language using local LLMs (LM Studio) + MCP servers. Supports draw.io, Mermaid, and other diagram formats.
tags: [diagram, architecture, LM-Studio, MCP, local-LLM, visualization]
---

# Diagram Generation from Natural Language

Generate structured diagrams (architecture, flowcharts, ERDs) by combining a local LLM with an MCP server that produces diagram markup.

## When to Use

- User asks for system architecture diagrams, component diagrams, or flowcharts
- Need to visualize relationships between services/components
- Want privacy/offline operation (no cloud API calls)
- Prefer natural language over manual diagram editing

## Architecture Pattern

```
User Request → LM Studio (local LLM) → Python Bridge Script → MCP Server → Diagram Markup (draw.io XML / Mermaid / etc.)
```

### Components

1. **LM Studio** — Local reasoning engine with OpenAI-compatible API (default port 1234)
   - Model: Ornith 35B or similar capable of structured output
   - Must enable Function Calling if using MCP servers with tool definitions

2. **Python Bridge Script** (`local-pipeline.py`) — Translates LLM output to MCP server input
   - Handles JSON quoting issues (LLM output often has nested quotes)
   - Calls MCP server's API endpoint with properly formatted requests

3. **MCP Server** — Generates diagram markup from structured prompts
   - `next-ai-draw-io` for draw.io XML generation
   - Can be extended for Mermaid, PlantUML, or other formats

## Implementation Steps

### 1. Install MCP Server

```bash
# Example: next-ai-draw-io
git clone <repo-url>
cd <project-dir>
npm install && npm run build
```

### 2. Configure LM Studio

- Load model (e.g., Ornith 35B)
- Note the OpenAI-compatible API endpoint (default: `http://localhost:1234/v1`)
- Verify Function Calling is enabled if using MCP servers with tool definitions

### 3. Create Bridge Script

See `templates/local-pipeline.py` for a starter template that:
- Connects to LM Studio's OpenAI-compatible API
- Formats prompts for diagram generation
- Handles JSON parsing and quoting issues
- Calls MCP server endpoints

### 4. Generate Diagram

```bash
python local-pipeline.py --prompt "Create a three-tier microservices architecture with frontend, API gateway, and database layers"
```

## Pitfalls & Fixes

### JSON Quoting Issues
**Problem**: LLM output contains nested quotes that break JSON parsing when passed to MCP server.
**Fix**: Use the bridge script's quote-handling logic (see `templates/local-pipeline.py` for escaping patterns).

### Browser Import Challenges
**Problem**: draw.io editor's JavaScript API may not accept XML via direct injection.
**Workarounds**:
1. Save `.drawio` file and open in draw.io desktop app
2. Use draw.io's "File → Open from" menu to load the saved file
3. Try importing via the editor's data URL parameter

### Model Capability Limits
**Problem**: Smaller models may not produce well-structured diagram markup.
**Fix**: Use models with strong instruction-following (≥7B parameters recommended). Ornith 35B works well for this task.

## Extending to Other Formats

To support Mermaid, PlantUML, or other diagram types:
1. Find/create an MCP server that generates the target format
2. Update the bridge script's output parser
3. Adjust the prompt template in `templates/local-pipeline.py`

## References

- `references/toolchain-config.md` — LM Studio + next-ai-draw-io configuration details
- `templates/local-pipeline.py` — Reusable bridge script template
- `scripts/verify-diagram.sh` — Validation script for generated diagram files
