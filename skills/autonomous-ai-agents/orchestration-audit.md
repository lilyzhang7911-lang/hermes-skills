---
name: orchestration-audit
description: A workflow for validating complex project goals and task decompositions using specialized sub-agents (Architect, Engineer, Librarian) to ensure logical integrity, technical feasibility, and knowledge management utility before execution begins.
---

# orchestration-audit

A workflow for validating complex project goals and task decompositions using specialized sub-agents (Architect, Engineer, Librarian) to ensure logical integrity, technical feasibility, and knowledge management utility before execution begins.

## Workflow

1. **Decomposition**: Break the high-level goal into an "Atomic Task Pipeline" where each task's output is a valid input for the next.
2. **Dispatch (The Audit Committee)**: Spawn three parallel sub-agents via `delegate_task` with specific personas:
   - **Architect**: Audits logical integrity and dependency chains.
   - **Engineer**: Evaluates technical feasibility and LLM implementation risks.
   - **Librarian**: Validates knowledge management utility (e.g., Obsidian integration, metadata).
3. **Synthesis**: The orchestrator agent collects the sub-agent reports, resolves conflicts, and presents a consolidated "Audit Report" to the user.
4. **Approval/Refinement**: The user reviews the report and either approves execution or requests refinements.

## Personas & Audit Focus

### Architect
- **Focus**: Logical completeness, dependency matching (Input $\rightarrow$ Output), single points of failure in the chain.

### Engineer
- **Focus**: Technical feasibility, accuracy risks of LLM reasoning steps, robustness of data transformations.

### Librarian
- **Focus**: Knowledge utility, metadata schema adherence, integration with user's specific knowledge management tools (e.g., Obsidian).

## Best Practices
- Use this workflow for any task involving complex multi-step logic or high-stakes project planning.
- Always present the "Synthesis" as a single, structured report to avoid overwhelming the user with raw sub-agent output.