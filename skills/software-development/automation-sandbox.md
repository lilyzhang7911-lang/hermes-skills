---
name: automation-sandbox
description: A workflow for safely testing new CLI tools, MCP servers, or complex automation scripts in an isolated environment.
---

# Automation Sandbox & Tool Validation

A workflow for safely testing new CLI tools, MCP servers, or complex automation scripts without polluting the primary workspace or risking production environments.

## Principles
- **Isolation (Sandboxing)**: Always create a dedicated directory (e.g., `[tool-name]-lab`) within the project root to serve as a controlled environment for testing. This allows for easy cleanup and prevents accidental modification of source files.
- **Step-by-Step Execution**: When debugging new tools or complex automation, avoid "one-liner" shell pipes in `execute_code`. Use discrete, sequential tool calls to ensure error visibility and prevent silent failures (e.g., the "false success" pattern).
- **Redundancy & Risk Mitigation**: Prioritize local simulation of workflows before attempting cloud/remote deployment to mitigate network-related failures or anti-bot triggers.

## Workflow
1.  **Setup**: Create a dedicated directory and initialize it (e.g., `git init`).
2.  **Local Validation**: Test the tool's basic functionality in the sandbox.
3.  **Remote Linkage**: Once local behavior is verified, link the sandbox to a remote repository for full-scale integration testing.

## Pitfalls
- **The "False Success" Trap**: Be wary of commands that return exit code 0 but fail to produce the expected side effects (e.g., API calls that don't actually create resources). Always verify state with a follow-up `list` or `status` command.
- **Complex Piping**: Avoid using complex shell pipes (`|`, `xargs`) inside `execute_code` when testing new tools; the error messages from failed sub-commands are often lost in the pipe, making debugging nearly impossible.