---
name: repository-infiltration
description: A class-level skill for extracting source code and configuration from repositories when standard high-level tools (like `git clone`) fail due to network restrictions or proxy issues.
---

# Software Development: Repository Infiltration

This skill provides a structured approach for "Fragmented Infiltration"—extracting critical files via low-level tools like `curl` when traditional Git operations are blocked by network instability, SSL errors, or server timeouts.

## Overview
When working in environments with restricted connectivity (e.g., firewalls, broken proxies, or unstable international links), attempting a full `git clone` is often a high-risk/low-reward operation. This skill prioritizes "precision extraction" to acquire the minimum viable information needed to understand a project's architecture and dependencies.

## Strategy: Fragmented Infiltration (The "Sniper" Approach)
Instead of downloading entire histories, focus on acquiring the "genetic markers" required for analysis or local reconstruction.

### 1. Phase I: Reconnaissance (Identify Markers)
Before attempting downloads, identify what defines the project's structure using `curl` to check for existence:
- **Dependency/Entry Points**: `package.json`, `pyproject.toml`, `Cargo.toml`.
- **Workspace Definitions**: `pnpm-workspace.yaml`, `lerna.json`, `go.work`.
- **Build Configs**: `tsconfig.json`, `Makefile`, `docker-compose.yml`.

### 2. Phase II: Precision Extraction (The "Sniper" Method)
Use low-level tools like `curl` to target specific raw file URLs. This bypasses the overhead of the Git protocol and its complex handshakes which often trigger timeouts in restricted environments.

**Target Priority List:**
1. **Configuration Files**: To understand the environment and entry points.
2. **Core Logic Entry Points**: (e.g., `src/index.ts`) To understand execution flow.
3. **Workspace Definitions**: To map out Monorepo dependencies.

### 3. Phase III: Reconstruction & Analysis
Once markers are acquired, use them to:
- Determine if a local build is possible via package managers.
- Identify missing internal dependencies (e.g., `@repo/*` packages in Monorepos).
- Map out the required environment variables and proxy settings needed for a successful full clone.

## Pitfalls & Troubleshooting
- **SSL/TLS Errors**: If `curl` fails with SSL errors, it often indicates an intercepting proxy or a mismatch in local CA certificates. 
- **403/404 on Raw URLs**: Ensure you are targeting the correct raw content domain (e.g., `raw.githubusercontent.com` for GitHub).
- **Monorepo Dependency Hell**: If a sub-package requires `@repo/*` dependencies, a simple `npm install` in that folder will fail. You *must* acquire the root workspace configuration first to understand the dependency graph.

## Reference Files
- `references/browsermcp-case-study.md`: Detailed log of the BrowserMCP infiltration attempt (2026-07-08).
