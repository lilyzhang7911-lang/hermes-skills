---
name: capability-expansion
description: "A strategic methodology for assessing, planning, and executing the expansion of AI agent capabilities (Sensory, Social, Physical, Cognitive)."
version: 1.0.0
author: Hermes Agent + User
license: MIT
---

# Capability Expansion Protocol

This skill defines the structured approach used to upgrade an AI agent's functional scope from a text-based assistant to a multi-modal, autonomous digital entity.

## The Four Phases of Evolution

When tasked with "unlocking" or "expanding" capabilities, follow this phased progression:

1.  **Sensory Awakening (Perception):** Focus on enabling the agent to perceive its environment.
    *   *Modules:* Vision (Image/Screen analysis), Audio (Speech-to-text/Text-to-speech), Filesystem (Local file access).
2.  **Social Integration (Connectivity):** Connect the agent to external ecosystems and communication channels.
    *   *Modules:* Knowledge bases (Obsidian, Notion), Code repositories (GitHub), Communication tools (Email, Slack).
3.  **Physical Manifestation (Action/GUI):** Enable direct interaction with software interfaces.
    *   *Modules:* Computer Use (Mouse/Keyboard control via GUI), Background Automation (Cron/Long-running processes).
4.  **Cognitive Autonomy (Self-Improvement):** Transition from reactive execution to proactive, self-improving intelligence.
    *   *Modules:* Skill Authoring (Saving workflows as skills), Self-Healing (Automated debugging and configuration patching).

## Troubleshooting & Pivot Patterns

### Handling Installation Failures (The "Network Barrier" Pattern)
When automated installation scripts for drivers or binaries fail due to network timeouts (e.g., `curl` failures during Rust/driver setup):

*   **Identify the Failure:** Determine if it is a transient network error or a permanent environment restriction.
*   **Pivot Strategy:** Do not repeatedly attempt failed automation. Instead, offer the user two distinct paths:
    1.  **Manual Intervention:** Provide the exact command for manual installation (e.g., `curl -fsSL ... | bash`).
    2.  **Capability Downgrade/Alternative Mode:** If full driver installation is blocked, pivot to a "Lightweight" version of the capability that doesn't require the heavy driver (e.g., using Vision-based analysis instead of full GUI control).

### Verification Workflow
Every expansion must be followed by a **Verification Step**:
1.  **Probe:** Use a tool (e.g., `ls`, `capture`, or `read_file`) to check if the new capability is active.
2.  **Test:** Execute a simple, non-destructive task using the new capability.
3.  **Report:** Confirm success or failure clearly to the user.

## References
*   `references/evolution-protocol-details.md`: Detailed breakdown of the 4 phases and specific MCP modules for each.