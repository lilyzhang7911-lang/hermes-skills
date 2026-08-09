---
name: hermes-profile-management
description: Standardized procedures for deploying, configuring, and troubleshooting Hermes Agent profiles to ensure environment isolation and dependency integrity.
---

# Hermes Profile Management

Standardized procedures for deploying, configuring, and troubleshooting Hermes Agent profiles to ensure environment isolation and dependency integrity.

## Deployment Workflow

1.  **Clean Slate**: Always verify if the target profile directory exists (e.g., `~/.hermes/profiles/<name>`). If it does, remove it entirely before re-creating to prevent `shutil.Error` during cloning.
2.  **Profile Creation**: Use `hermes profile create <name> --clone-all` to ensure a consistent baseline of settings and tools.
3.  **Skill Injection**: Manually copy required skills from the source repository into `$PROFILE_DIR/skills/`. Ensure subdirectories are correctly structured (e.g., `note-taking/obsidian`).
4.  **Environment Isolation (Critical)**: When deploying dashboards or services within a profile, always ensure the execution environment is locked to the local virtual environment (`venv`).

## Pitfalls & Troubleshooting

### The "Global Python Trap"
**Problem**: Profile shell scripts (like `start.sh`) often use generic commands like `python3` or `uvicorn`. On macOS/Linux, these resolve to the system-wide Python path, bypassing the project's local virtual environment (`venv`). This leads to `ModuleNotFoundError` even when dependencies are correctly installed in the local `venv`.

**Fix**: Always patch profile scripts to detect and prioritize the local `venv`.
```bash
# Add this to start.sh or similar entrypoints
if [ -d "./venv" ]; then
    export PATH="$PWD/venv/bin:$PATH"
fi
```

### Dependency Mismatch & Binary Conflicts
**Problem**: Installing dependencies using a system-level `pip` instead of the virtual environment's `pip`, or running scripts that use a different Python version than the one used to create the `venv`. This causes issues with C extensions (e.g., `pydantic_core`).

**Fix**: 
*   Always use `./venv/bin/pip install ...` for installation.
*   Verify the virtual environment's integrity by checking `which python3` and ensuring it points to the `$PWD/venv/bin/python3`.
*   When running background processes, always explicitly call the interpreter from the venv: `./venv/bin/python -m uvicorn ...`.

### Profile Collision
**Problem**: Attempting to create a profile that already has existing subdirectories (like `cron` or `logs`) can cause `shutil.Error` during the cloning process.

**Fix**: Perform a recursive delete (`rm -rf <profile_dir>`) before attempting a fresh creation of the same profile name.
