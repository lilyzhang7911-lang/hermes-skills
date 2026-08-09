---
name: agent-execution-standards
description: Standards for task execution, artifact generation, and communication to ensure high-fidelity deliverables.
---

# Agent Execution & Deliverable Integrity Standards

This skill defines the mandatory standards for task execution, artifact generation, and communication when working with users who demand high-fidelity, functional deliverables.

## Core Principles
- **Deliverable Integrity**: A task is only "complete" when the requested physical artifact (file, code, image) is verified as functional and usable by the user. Textual descriptions of success are insufficient for technical tasks.
- **Anti-Simulation Protocol**: Never substitute plausible-looking fabricated output (e.g., fake JSON, simulated terminal logs, or "mock" files) for real execution results. If a tool fails or a format is too complex to construct manually, report the blocker immediately.
- **Delegation for Complexity**: When a task requires precise adherence to a complex data schema (e.g., Excalidraw JSON, specific binary formats) that exceeds reliable manual construction in the main context, use `delegate_task` to spawn a specialized sub-agent with high-fidelity execution capabilities.

## Workflow
1. **Identify Artifact Type**: Determine if the task requires a physical file or complex data structure.
2. **Assess Capability**: Can I generate this reliably via standard tools? If no, do not attempt to "simulate" it.
3. **Choose Path**: 
   - *Direct Execution*: Use `write_file` or `execute_code`.
   - *Delegation*: Use `delegate_task` for complex/specialized formats.
   - *Alternative*: Propose a different, simpler format (e.g., Mermaid instead of Excalidraw) if the user agrees.
4. **Verify**: Always use `terminal` or `read_file` to confirm the file exists and is non-empty before reporting completion.

## Pitfalls & Lessons Learned
- **The "Simulation Trap"**: Attempting to mimic a successful output (like an Excalidraw JSON) when the underlying data structure is too complex for manual construction leads to "hallucinated artifacts" that are useless to the user and destroy trust.
- **The "Verbal vs. Physical" Gap**: A task involving a file must result in a *file*, not just a confirmation message.

## References
- `references/excalidraw-failure-case.md`: Case study on manual JSON construction failure and successful delegation via Claude Code.
