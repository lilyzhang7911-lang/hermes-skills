---
name: macos-automation
description: Specialized workflows for navigating macOS security (TCC), permission mismatches, and perception failures in AI automation.
---

# macOS Automation & TCC Troubleshooting

Specialized workflows for navigating macOS security (TCC), permission mismatches, and perception failures in AI automation.

## Core Philosophy: The Semantic Pivot
When visual/pixel-based perception fails despite permissions being granted, do not attempt to "force" the pixel capture. Instead, pivot to **Semantic Perception** using direct application scripting. This moves from "seeing pixels" to "reading structure."

## Troubleshooting Workflow (The Perception Loop)
1.  **Verify Permissions**: Run `hermes computer-use doctor` to confirm Accessibility and Screen Recording are GRANTED.
2.  **Diagnose Failure Mode**: If permissions are OK but snapshots are empty, identify if it's a "Window Layering" issue or a "TCC Shell Restriction".
3.  **The Semantic Pivot (Fallback)**:
    - **Avoid `System Events` via shell**: Direct calls to `System Events` from a terminal often trigger `-1719` errors due to shell-level TCC restrictions.
    - **Target Applications Directly**: Use AppleScript to talk directly to the target app (e.g., Chrome, Safari) rather than through the system's event manager.
    - **Example Command**: `osascript -e 'tell application "Google Chrome" to get name of every window'`
4.  **Verify Semantic Success**: Use retrieved semantic data (titles, URLs) to confirm context before attempting complex interactions.

## Pitfalls & Gotchas
- **The Shell Restriction (-1719)**: `osascript` calls from a terminal/shell session often lack the necessary "Accessibility" entitlement even if the parent app has it. Direct application targeting is more reliable.
- **Empty Snapshots**: Often caused by window focus or rendering layers (e.g., Chrome's GPU process) being invisible to standard capture methods.

### PostgreSQL@16 `brew services start` Fails with Exit Code 5

When `brew services start postgresql@16` returns:
```
Bootstrap failed: 5: Input/output error
Error: Failure while executing; '/bin/launchctl bootstrap gui/501 ...' exited with 5.
```

**Root cause**: The data directory (`/opt/homebrew/var/postgresql@16`) does not exist or was never initialized by `initdb`. This is NOT a permission issue — it's a missing initialization step.

**Fix sequence:**
```bash
# 1. Stop any stuck service
brew services stop postgresql@16
sleep 2

# 2. Create the data directory with correct permissions
mkdir -p /opt/homebrew/var/postgresql@16
chown $(id -u):$(id -g) /opt/homebrew/var/postgresql@16
chmod 700 /opt/homebrew/var/postgresql@16

# 3. Initialize the database cluster (if not already done)
/opt/homebrew/opt/postgresql@16/bin/initdb \
  -D /opt/homebrew/var/postgresql@16 \
  --locale=en_US.UTF-8 -E UTF-8

# 4. Start the service
brew services start postgresql@16

# 5. Verify
pg_isready -h localhost -p 5432
# Should return: "localhost:5432 - accepting connections"
```

**Verification**: Check logs at `/opt/homebrew/var/log/postgresql@16.log` — if you see `could not access directory "/opt/homebrew/var/postgresql@16": No such file or directory`, that confirms the data dir is missing.

### Homebrew Install Timeout Pattern

When `brew install <package>` times out (300s/600s limits):
- **Try with explicit timeout**: Set a longer terminal timeout before running, e.g., `timeout=600` in terminal tool calls.
- **Retry without workdir override**: Some brew installs fail with "getcwd: cannot access parent directories" when run from certain working directories — use default workdir or `/Users/<username>`.
- **Build from source fallback**: If bottle download keeps timing out, try `brew install --build-from-source <package>` (slower but bypasses network bottle fetch).

### Firecrawl Self-Hosted Dependencies (No Docker)

Firecrawl API server requires three services when running without Docker Compose:

| Service | Purpose | Install Command |
|---------|---------|-----------------|
| **Redis** | Job queue backend (BullMQ) | `brew install redis` → `brew services start redis` |
| **PostgreSQL@16** | Database + nuq-postgres queue | `brew install postgresql@16` → fix data dir if needed → `brew services start postgresql@16` |
| **RabbitMQ** | AMQP message broker (amqplib) | `brew install rabbitmq` → `brew services start rabbitmq` |

**Verification checklist before running Firecrawl API:**
```bash
redis-cli ping                    # Should return PONG
pg_isready -h localhost -p 5432   # Should return "accepting connections"
rabbitmqctl status                 # Should show running node
```

If `brew install rabbitmq` also times out, same pattern as PostgreSQL — increase timeout or try `--build-from-source`.

## Electron App Installation on macOS — The `ditto` Rule
When installing Electron-based apps from DMG releases, **never use `cp -R`** — it strips resource forks and code signatures, causing TCC sandbox failures where the app runs but produces no visible window (process exists in Activity Monitor but `visible is false`).

### Correct Installation Pattern
```bash
# 1. Mount DMG
hdiutil attach ~/Desktop/<app>.dmg
# 2. Copy with ditto (preserves resource forks + signatures)
ditto "/Volumes/<mount>/App.app" /Applications/App.app
# 3. Unmount
hdiutil detach "/Volumes/<mount>"
```

### Troubleshooting: App Runs But No Window
If the Electron app process is visible in Activity Monitor but no window appears:

1. **Check TCC permissions**: System Settings → Privacy & Security → Accessibility → ensure the app is checked.
2. **Grant via Finder**: Right-click `/Applications/App.app` → Open → confirm the "Are you sure?" dialog.
3. **Verify with AppleScript**: `osascript -e 'tell application "System Events" to get visible of every process whose name contains "<App>"'` — returns `false` if TCC blocks it.
4. **Kill and relaunch** after granting permissions: `killall -9 <AppName>` then re-open from Finder.

### FreeLLMAPI v0.5.0 Specifics
- **Port changed**: v0.5.0 uses port **31415** (not 3001 like the old source-mode version). Verify with `lsof -iTCP:31415 -sTCP:LISTEN`.
- **Legacy process conflict**: The old source-mode FreeLLMAPI runs on :3001. Kill it (`kill <pid>`) before letting v0.5.0 take over, or the new app won't bind to its port.

References:
- `references/semantic-pivot-recipe.md` - Detailed case study of the Pixel $\rightarrow$ Semantic pivot on macOS.