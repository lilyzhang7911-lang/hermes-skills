---
name: web-semantic-auditor
description: Performs deep semantic analysis on web pages by identifying contradictions between logical structure (DOM/AX Tree) and visual perception (Saliency). Uses a Hegelian Dialectic framework to move from descriptive data to intent-driven insights.
---

# web-semantic-auditor

## Overview
A specialized cognitive skill for performing deep semantic analysis on web pages. It moves beyond simple text extraction by identifying "Semantic Contradictions"—discrepancies between how an element is structured in code and how it is perceived visually. This allows the agent to understand not just *what* is on a page, but the *intent* behind its design (e.g., detecting dark patterns or hidden calls-to-action).

## Core Logic: The Hegelian Semantic Model
The engine operates on the principle of **Aufhebung** (Sublation), resolving contradictions between three dimensions to reach a higher level of understanding:

1.  **Thesis (Logic/Structure)**: The "What" as defined by the Accessibility Tree and DOM (Roles, Hierarchy, Interactivity).
2.  **Antithesis (Perception/Visuals)**: The "How it feels" as perceived through visual saliency (Area, Position, Contrast).
3.  **Synthesis (Intent/Meaning)**: The "Why" derived from the tension between Logic and Perception.

## Analysis Layers

### Layer 1: Logical Skeleton Protocol
Extracts structured data from the Accessibility Tree to establish the page's structural backbone.
- **Key Attributes**: `id`, `role` (button, link, heading, etc.), `text_content`, `is_interactive` (boolean), `hierarchy_depth`.

### Layer 2: Visual Saliency Protocol
Quantifies visual importance into a continuous score ($0.0$ to $1.0$) using the following heuristic:
$$Score = (Area_{norm} \times 0.4) + (Position_{norm} \times 0.3) + (Contrast_{norm} \times 0.3)$$
- **Area**: Normalized relative to viewport size.
- **Position**: Proximity to the visual center of the screen.
- **Contrast**: Perceived brightness/color contrast against background.

### Layer 3: Semantic Conflict Resolution Engine (S.C.R.E.)
The final reasoning layer that classifies elements into four dialectical states based on the conflict between Layers 1 and 2:

| State | Logic (Thesis) | Visual (Antithesis) | Synthesis (Intent) |
| :--- | :--- | :--- | :--- |
| **Resonance** | High Interactivity | High Saliency | **Core CTA**: The intended primary action. |
| **Weight Mismatch** | Low Hierarchy / Non-interactive | High Saliency | **Hidden CTA**: Visual priming to drive attention to non-primary elements. |
| **Semantic Camouflage** | No Interactive Attributes | High Saliency | **UX Trap/Dark Pattern**: Elements designed to look interactive but lack functionality. |
| **Structural Fragmentation** | Deep Nesting / Disconnected | High Saliency | **Componentized Data**: Complex UI components split into fragmented DOM nodes. |

## Workflow for Execution
1.  **Capture**: Use `browser_navigate` and `browser_snapshot` to gather AX Tree data.
2.  **Perceive**: Use `vision_analyze` to capture visual layout and saliency features.
3.  **Quantify**: Calculate Saliency Scores for all salient elements.
4.  **Resolve**: Run the Conflict Detection Matrix to generate a **Semantic Intent Report**.

## Example Output Format
- **Element**: [Text/Role]
- **Detected Intent**: [Resonance / Weight Mismatch / Camouflage / Fragmentation]
- **Reasoning**: Brief explanation of the detected contradiction.
