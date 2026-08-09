---
name: context-engineering
description: Context management strategies for long-running agentic workflows — memory, compaction, tool clearing. Use when dealing with extended conversations or multi-step agent tasks that exceed context limits.
---

# Context Engineering

## Trigger Conditions
- Long-running agent sessions approaching context window limits
- Need to manage conversation history efficiently
- Multi-step workflows requiring state persistence across turns

## Core Strategies (from Anthropic Cookbooks)

### 1. Automatic Context Compaction
```python
# Pattern: Compress conversation history automatically
# When context approaches limit, summarize rather than truncate
def compact_context(messages):
    """Compress old messages into summaries while preserving key facts"""
    # Keep recent exchanges verbatim
    # Summarize older turns into concise bullet points
    pass
```

**Reference**: `tool_use/automatic-context-compaction.ipynb`

### 2. Prompt Caching
- Cache and reuse prompt context for cost savings + faster responses
- Speculative caching: warm cache while user formulates queries
- **Key insight**: First token latency reduced by pre-warming

**Reference**: `misc/prompt_caching.ipynb`, `misc/speculative_prompt_caching.ipynb`

### 3. Session Memory Compaction
```python
# Background threading for instant compaction
# Combines prompt caching with async memory management
def session_compaction(session_id):
    """Manage long-running conversations with background compaction"""
    # Use prompt caching API to reduce token costs
    # Compact history into structured memory
    pass
```

**Reference**: `misc/session_memory_compaction.ipynb`

### 4. Context Engineering Tools (Comprehensive)
- Compare strategies for long-running agents
- Learn when each applies, cost analysis, composition patterns
- **Key insight**: Different strategies compose — use caching + compaction + tool clearing together

**Reference**: `tool_use/context_engineering/context_engineering_tools.ipynb`

## Hermes Integration Points

### With Existing Skills
- Enhances `hermes-agent` skill's context management
- Complements `dynamic-workflow` for state persistence
- Supports `advisor-orchestrator-worker` with memory management

### Practical Implementation
```yaml
# When to use which strategy:
# - Prompt Caching: Repeated system prompts, static context
# - Context Compaction: Long conversations, need to preserve facts
# - Tool Clearing: Reduce tool output noise in active turns
# - Session Memory: Cross-session state persistence (Obsidian)
```

## Pitfalls

1. **Over-compaction**: Summarizing too aggressively loses nuance
2. **Cache invalidation**: Stale cached prompts cause incorrect behavior
3. **Memory leaks**: Uncompacted sessions consume resources
4. **Tool output bloat**: Verbose tool responses waste context window

## Verification

- [ ] Context stays within model limits
- [ ] Key facts preserved across compaction
- [ ] No stale cache references
- [ ] Session memory persists correctly
