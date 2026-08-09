---
name: high-dimensional-detection
description: Decision framework for interacting with "High-Dimensional Objects" (e.g., GPU-accelerated apps) that appear active but return empty semantic/visual data.
---

# High-Dimensional Object Detection & Response

## Overview
This skill provides a decision-making framework for interacting with "High-Dimensional Objects"—digital entities (like modern Web Apps or GPU-accelerated processes) that appear active and responsive to basic system queries but present as "empty" or "void" when subjected to standard semantic/visual scanning.

## Symptoms: The "Perceptual Void" Pattern
A subject is classified as a High-Dimensional Object if it meets the following criteria simultaneously:
1.  **Identity Confirmed**: `osascript` or similar direct commands return valid metadata (e.g., Window Title, URL).
2.  **Process Active**: The process is confirmed running in the OS task list.
3.  **Semantic Void**: `browser_snapshot` or Accessibility API returns `element_count: 0` or an empty tree.
4.  **Visual Void**: Vision-based analysis (screenshots) returns a blank/solid color canvas despite the window being in the foreground.

## Root Causes
*   **GPU Hardware Acceleration**: Content is rendered directly via GPU, bypassing the standard Accessibility/Screen Capture layers.
*   **Process Sandboxing**: The renderer process is isolated from the system's accessibility services.
*   **UI Thread Deadlock**: The application's main thread is blocked by heavy computational tasks (e.g., video decoding), preventing UI updates to the OS.

## Decision Matrix: Pivot Strategy

| Current Mode | Symptom | Recommended Action | Tactical Goal |
| :--- | :--- | :--- | :--- |
| **Dialogue Mode** (Accessibility/Vision) | `element_count: 0` / Blank Screenshot | **Switch to Infiltration Mode** | Bypass the UI layer entirely. |

### Phase 1: Infiltration (Direct Command Injection)
*   **Action**: Use `osascript` or direct CLI commands to trigger physical-level interactions (e.g., `key down`, `click at [x,y]`).
*   **Goal**: Force the subject to react to low-level system events to confirm "consciousness" without needing a semantic tree.

### Phase 2: Genomic Sequencing (Network/Data Layer)
*   **Action**: Monitor network traffic or use `curl` to fetch raw data streams directly from the source URL.
*   **Goal**: Extract information from the "bloodstream" (data packets) rather than the "face" (UI).

## Workflow for Agents
1.  **Verify Identity**: Confirm process and metadata via direct system calls.
2.  **Test Semantic Depth**: Attempt `browser_snapshot`. If empty, trigger this skill.
3.  **Diagnose & Pivot**: Identify if the void is visual or semantic; immediately switch to Infiltration/Genomic modes.
