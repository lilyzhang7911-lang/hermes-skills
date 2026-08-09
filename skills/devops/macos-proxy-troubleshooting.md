---
name: macos-proxy-troubleshooting
description: Diagnose and fix VPN/proxy connectivity issues on macOS, especially with SOCKS5 vs HTTPS proxy behavior differences.
tags: [proxy, vpn, network, troubleshooting]
---

# macOS Proxy/VPN Troubleshooting

Diagnosing proxy and VPN connectivity issues on macOS, focusing on the common SOCKS5 vs HTTPS proxy behavioral differences.

## Core Pattern: SOCKS5 vs HTTPS Proxy

macOS proxy tools (Lantern Pro, ClashX, V2RayN, etc.) typically expose two ports:
- **SOCKS5** — usually UDP-based, used by browsers natively
- **HTTPS/HTTP** — CONNECT tunnel via TCP, more compatible with curl/wget/certified TLS clients

### Key Finding (Lantern Pro Case Study)

| Proxy Type | Behavior | Notes |
|-----------|----------|-------|
| SOCKS5 (e.g. 127.0.0.1:49287) | Tunnel establishes (`SOCKS5 request granted`) but TLS handshake may timeout on certain sites | UDP resolution path; some exit nodes restrict this |
| HTTPS/HTTP (e.g. 127.0.0.1:53595) | CONNECT tunnel succeeds, full TLSv1.3 handshake completes | TCP-based; more universally compatible |

**Rule of thumb:** If SOCKS5 works for some sites but not others (especially Google/YouTube), switch to HTTPS proxy — it's almost always the fallback that works.

## Diagnostic Workflow

### Step 1: Verify Proxy is Running
```bash
# Check process
ps aux | grep -i <proxy-name>

# Check port is listening
lsof -iTCP -sTCP:LISTEN -P | grep <port>
```

### Step 2: Test Connectivity with Both Protocols
```bash
# SOCKS5 test (verbose to see handshake)
curl -x socks5h://127.0.0.1:<socks_port> --connect-timeout 10 -v https://www.google.com 2>&1 | head -30

# HTTPS proxy test
curl -x http://127.0.0.1:<https_port> --connect-timeout 10 -v https://www.google.com 2>&1 | head -40
```

### Step 3: Interpret Results
- **`SOCKS5 request granted` + TLS timeout** → Tunnel works, exit node restricts specific sites via SOCKS path. Try HTTPS proxy instead.
- **`CONNECT tunnel established, response 200` + full TLS handshake** → HTTPS proxy is fully functional.
- **Connection refused / no process** → Proxy app not running or crashed. Restart it.

### Step 4: Check Firewall (if both fail)
```bash
/usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
/usr/libexec/ApplicationFirewall/socketfilterfw --listapps | grep -i <proxy-name>
```

## Pitfalls & Gotchas

- **VPN "代理所有内容" hijacking localhost ports** — Lantern and similar VPN apps with a "Proxy All Content" / "代理所有内容" setting will proxy ALL local listening ports, not just internet traffic. This means services like LM Studio (port 1234), local API servers, and even llama-server's dynamic inference port get flooded with hundreds of VPN proxy connections. Symptoms: lightweight endpoints (`/v1/models`) respond but heavy endpoints (`/v1/chat/completions`) time out because the proxy connections queue ahead of real requests. Diagnosis: `lsof -i :<port> | wc -l` — if >100 connections, check `lsof -i :<port> -c Lantern | wc -l` for VPN占比. Fix: disable "代理所有内容" in Lantern settings (no need to fully close VPN), then restart the affected service to clear zombie connections. After disabling, apps that depend on VPN for internet (e.g. HuggingFace Hub access) will time out — they'll retry and eventually fall back to local resources.
- **macOS TCC.db is locked** — You cannot directly query `~/Library/Application Support/com.apple.TCC/TCC.db` from terminal without Full Disk Access. Use System Settings GUI or `system_profiler SPPrivacyDataType` instead.
- **Accessibility ≠ Network** — macOS Accessibility (辅助功能) permissions control UI automation, NOT network access. Don't confuse them when troubleshooting proxy issues. However, some VPN apps internally use Accessibility for system tray integration — check if the app's own settings panel shows connectivity status.
- **Node switching matters** — Different exit nodes (Seoul vs Singapore vs US) have different site-blocking policies. If one node fails, try another in the VPN app UI before changing local config.
- **Global proxy risk** — Setting system-wide proxy can break background services (iCloud, system updates). Prefer per-app or environment-variable-based proxy usage (`export https_proxy=...`).

## User Preferences (from session)

- User prefers **on-demand proxy use** over global settings to avoid instability when VPN disconnects.
- When setting up proxy temporarily: `export https_proxy=http://127.0.0.1:<port>` and `export http_proxy=http://127.0.0.1:<port>`.

## References
- `references/lantern-pro-proxy-config.md` — Lantern Pro specific configuration details from session.
