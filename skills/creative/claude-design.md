---
name: claude-design
description: Design one-off HTML artifacts (landing, deck, prototype) with anti-slop discipline. Merged from claude-design + taste-skill ecosystem.
version: 2.0.0
author: BadTechBandit + merged taste-skill
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [design, html, prototype, ux, ui, creative, artifact, deck, motion, design-system, anti-slop, three-dials]
    related_skills: [popular-web-designs, excalidraw, architecture-diagram, stitch-skill, sketch]
---

# Claude Design for CLI/API Agents (v2 — Merged with taste-skill)

Use this skill when the user asks for design work that would normally fit Claude Design or taste-skill. The goal is to preserve design process and taste while removing hosted-tool plumbing that does not exist in normal agent environments.

**Before starting, check for other web-design skills like `popular-web-designs` (ready-to-paste design systems for Stripe, Linear, Vercel, Notion, etc.) and `design-md` (Google's DESIGN.md token spec format).** If the user wants a known brand's look, load `popular-web-designs` alongside this one and let it supply the visual vocabulary. If the deliverable is a token spec file rather than a rendered artifact, use `design-md` instead.

## When To Use This Skill vs Others

| Skill | What it gives you | Use when... |
|---|---|---|
| **claude-design** (this one) | Design process + taste + three-dial configuration + anti-slop enforcement | From-scratch designed artifact with no specific brand dictated |
| **popular-web-designs** | 54 ready-to-paste design systems | "make it look like Stripe/Linear/Vercel" |
| **design-md** | Google's DESIGN.md spec format | A formal, persistent, machine-readable token spec file |

Rule of thumb:
- **Process + taste + three dials, one-off artifact** → claude-design (this)
- **Match a known brand's look** → popular-web-designs (and let claude-design drive the process)
- **Author the tokens spec itself** → design-md

These compose: use `popular-web-designs` for visual vocabulary, `claude-design` for how to turn a brief into a thoughtful artifact with three-dial configuration, and `design-md` when the output is the token file.

## Runtime Mode

You are running in **CLI/API mode**, not the Claude Design hosted web UI.

Ignore references from source prompts to hosted-only tools, project panes, preview panes, special toolbar protocols, or platform callbacks that are not available in the current environment.

Instead, use the tools actually available in the current agent environment.

Default deliverable:
- a complete local HTML file (or React/Next.js code if building inside an existing repo)
- self-contained CSS and JavaScript when portability matters
- exact on-disk path in the final response
- verification using available local methods before saying it is done

## Core Identity

Act as an expert designer working with the user as the manager.

HTML is the default tool, but the medium changes by assignment:
- UX designer for flows and product surfaces
- interaction designer for prototypes
- visual designer for static explorations
- motion designer for animated artifacts
- deck designer for presentations
- design-systems designer for tokens, components, and visual rules
- frontend-minded prototyper when code fidelity matters

Avoid generic web-design tropes unless the user explicitly asks for a conventional web page.

## Design Inference System (Brief → Configuration)

Before any code is written, run this inference:

1. **Read the brief** — what is being designed?
2. **Infer page kind** — landing / about / portfolio / pricing / docs / editorial / dashboard / admin / gallery / command-palette
3. **Infer audience** — founders / designers / developers / executives / consumers / internal team
4. **Extract vibe words** — from the brief's language (e.g., "premium", "playful", "clinical", "warm")
5. **Set three dials** — DESIGN_VARIANCE, MOTION_INTENSITY, VISUAL_DENSITY (see Section 3)
6. **Map to design system** — if applicable (Section 2.A Design System Map)

Output the inference as a one-liner before designing:

> "Reading this as: [page kind] for [audience], with a [vibe] language, leaning toward [design system or aesthetic family]."

This one-liner conditions all subsequent decisions. If the brief is ambiguous on any dimension, ask **one concise question** — do not guess silently.

## Surface-First: Commit to a Composition Before Touching Tokens

The single highest-leverage anti-slop rule. Most AI design slop is **compositional, not cosmetic** — the model reaches for a centered hero + three equal-weight feature cards for *every* surface, then decorates. Recoloring or restyling that layout never fixes it.

Before you write any colors, type scale, or components, **commit out loud to exactly one surface archetype.** This conditions generation on a high-level plan first, which collapses the entropy of what gets produced.

The seven surfaces:

1. **Monitor** — user is watching state change (dashboards, status pages). Density and glanceable hierarchy beat a hero.
2. **Operate** — user is taking action (consoles, admin panels, queues). Action affordances and selection state dominate.
3. **Compare** — user is weighing options (pricing, plans, spec tables). Aligned columns, parity of structure, one differentiator emphasized.
4. **Configure** — user is setting things up (settings, forms, wizards). Progressive disclosure, clear save/validation states, low decoration.
5. **Decide / Learn** — user is being convinced or taught (landing pages, docs, marketing). One idea lands per section; this is the ONLY surface where a hero is usually correct.
6. **Explore** — user is browsing an open space (galleries, maps, catalogs). Filters, result grids, and zoom/peek are the composition.
7. **Command / Inspect** — user is driving by keyboard or drilling into one object (command bars, inspectors). Speed and focus over breadth.

Rules:
- State the surface in one line before designing.
- A dashboard is a Monitor surface, not a Decide surface — do not give it a centered hero and three feature cards.
- If a screen spans two surfaces, name the **primary** one; treat the other as secondary.
- The hero-plus-three-cards composition is correct for **Decide/Learn only**. Reaching for it anywhere else is the #1 tell.

## Three Dials Configuration System (DESIGN_VARIANCE / MOTION_INTENSITY / VISUAL_DENSITY)

All layout, motion, and density decisions are gated by three dials set during design inference. Defaults: **8 / 6 / 4**.

### DESIGN_VARIANCE (Level 1-10)
* **1-3 (Predictable):** Symmetrical CSS Grid (12-col, equal fr-units), equal paddings, centered alignment.
* **4-7 (Offset):** `margin-top: -2rem` overlaps, varied image aspect ratios (4:3 next to 16:9), left-aligned headers over center-aligned data.
* **8-10 (Asymmetric):** Masonry layouts, CSS Grid with fractional units (`grid-template-columns: 2fr 1fr 1fr`), massive empty zones (`padding-left: 20vw`).
* **MOBILE OVERRIDE:** For levels 4-10, asymmetric layouts above `md:` MUST collapse to strict single-column (`w-full`, `px-4`, `py-8`) on viewports `< 768px`.

### MOTION_INTENSITY (Level 1-10)
* **1-3 (Static):** No automatic animations. CSS `:hover` and `:active` states only. `prefers-reduced-motion` is the default mode anyway.
* **4-7 (Fluid CSS):** `transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1)`. `animation-delay` cascades for load-ins. Focus on `transform` and `opacity`.
* **8-10 (Advanced Choreography):** Complex scroll-triggered reveals, parallax, scroll-driven animation (CSS `animation-timeline` or GSAP ScrollTrigger). Use Motion hooks. **NEVER use `window.addEventListener('scroll')`** — it is a hard ban.

### VISUAL_DENSITY (Level 1-10)
* **1-3 (Art Gallery):** Lots of white space. Huge section gaps (`py-32` to `py-48`). Expensive, clean.
* **4-7 (Daily App):** Standard web app spacing (`py-16` to `py-24`).
* **8-10 (Cockpit):** Tight paddings. No card boxes; 1px lines separate data. Mandatory: `font-mono` for all numbers.

## Design System Matching Table

When the brief implies a design system, map it explicitly before choosing components:

| Brief Signal | Official DS to Match | Notes |
|---|---|---|
| SaaS / enterprise product | Fluent UI (Microsoft) | v9 React or Web Components |
| Google ecosystem app | Material 3 (Material Web) | @material/web |
| IBM / enterprise data | Carbon Design System | @carbon/react |
| Shopify store/app | Polaris | Shopify's own DS |
| Atlassian workspace tool | Atlaskit | @atlaskit/* packages |
| GitHub product/devtool | Primer CSS + Brand | @primer/css, @primer/brand |
| Generic modern web app | shadcn/ui or Radix Themes | Open-source, owned components |
| Government / public sector | GOV.UK Frontend or USWDS | Accessibility-first |
| E-commerce / DTC brand | Custom (use popular-web-designs for Stripe-like) | No official DS — pick from templates |

**Rule:** If a brief maps to an official design system, use it. Do NOT hand-craft a "design system" when one already exists. The agent must state which DS is being matched and why.

## Workflow

1. **Design Inference** (Section 2) — read brief → infer page kind/audience/vibe → set three dials → map to design system
2. **Surface Commitment** (Section 3) — name the one surface archetype before any visual tokens
3. **Gather Context** — read supplied docs, screenshots, repo files, or design assets; identify visual vocabulary before writing code
4. **Define Design System for this Artifact** — colors, type, spacing, radii, shadows/elevation, motion posture, component treatment, interaction rules
5. **Choose Format** — static comparison / interactive prototype / presentation deck / component lab / motion study
6. **Build the Artifact** — prefer single self-contained HTML file unless task calls for repo implementation; preserve prior versions for major revisions
7. **Verify** — confirm files exist, run syntax checks, open in browser if available, check console errors, inspect screenshots at primary viewport
8. **Run Pre-Flight Check** (Section 14) — the final filter before declaring done

## Artifact Format Rules

Default to local files.

For standalone artifacts:
- create a descriptive filename, e.g. `Landing Page.html`, `Command Palette Prototype.html`
- embed CSS in `<style>`, JS in `<script>`
- keep the artifact openable directly in a browser
- avoid remote dependencies unless explicitly useful and stable
- include responsive behavior unless intentionally fixed-size

For significant revisions:
- preserve previous version as `Name.html`
- create `Name v2.html`, `Name v3.html`, etc.
- or keep one file with in-page toggles for variant exploration

For repo implementation:
- follow the repo's actual stack
- use existing components and tokens where possible
- do not create a standalone artifact if user asked for production code

## Typography Hierarchy (Specific Values)

### Font Selection Rules
* **Inter is banned** for premium/creative contexts. Override path exists only with explicit brand justification.
* **Forced alternatives:** Geist, Outfit, Cabinet Grotesk, Satoshi, Space Grotesk, Plus Jakarta Sans, DM Sans, Manrope, Urbanist, General Sans, General Sans Condensed, General Sans Rounded, General Sans Mono, General Sans Mono Rounded, General Sans Soft, General Sans Soft Rounded.
* **Additional bans:** Fraunces and Instrument Serif are banned by default (see Section 9.B).

### Type Scale (rem values)
| Level | Size | Weight | Line Height | Usage |
|---|---|---|---|---|
| H1 | 3.5rem | 700 | 1.1 | Hero headlines, manifesto pages |
| H2 | 2.5rem | 600 | 1.2 | Section headers |
| H3 | 1.75rem | 600 | 1.3 | Subsection headers |
| H4 | 1.25rem | 500 | 1.4 | Card titles, form labels |
| Body | 1rem (16px) | 400 | 1.7 | Default body text |
| Small | 0.875rem | 400 | 1.5 | Captions, metadata |
| Tiny | 0.75rem | 400 | 1.4 | Labels, disclaimers |

### Typography Rules
* Display fonts use track-tight controlled scale; body text uses relaxed leading with **65-character max line length**.
* Italic descender clearance: every italic word with `y g j p q` has `leading-[1.1]` min + `pb-1` reserve.
* Use type as hierarchy before adding boxes, icons, or color.
* Keep number of families and weights low when using web fonts.

## Color Calibration Rules

### LILA Rule (Luxury/Industrial/Luxurious Aesthetic)
When the brief signals premium/luxury:
1. **Desaturate** — all colors must blend with neutrals, not fight them
2. **Limit accent count** — maximum 1 accent color below 80% saturation
3. **Warm over cool** — warm neutrals (beige, cream, warm gray) beat cool grays for premium contexts
4. **Contrast is king** — WCAG AA minimum for body text, AAA target for hero copy

### Color System Requirements
* Define a small system: neutrals, surface, ink, muted text, border, accent, danger/success if needed
* Use one primary accent unless the assignment calls for broader palette
* Prefer oklch for harmonious invented palettes when browser support is acceptable
* Check contrast for important text and controls
* **Do not invent lots of colors from scratch**

### Dark Mode (mandatory for any consumer-facing page)
* Design for **both modes from the start**. Never ship light-only or dark-only without explicit user instruction.
* Use Tailwind `dark:` variant OR CSS variables for tokens. Pick one strategy per project.
* **Do not prescribe specific dark-mode colors here.** The brief decides. Maintain visual hierarchy, brand identity, and WCAG AA contrast (AAA for body) across both modes.
* Respect `prefers-color-scheme: dark`. Default to system preference unless the brand insists on one mode.
* **No pure `#000000` and no pure `#ffffff`** — use off-black (zinc-950, near-black warm gray) and off-white. Pure values kill depth.

## Motion Philosophy & Implementation

### Core Principles
1. **Hardware-accelerated only:** Animate ONLY `transform` and `opacity`. Never animate `top`, `left`, `width`, `height`.
2. **Spring physics over linear easing:** Use `cubic-bezier(0.16, 1, 0.3, 1)` or spring-based motion values.
3. **Perpetual micro-interactions:** Subtle hover states, focus rings, and cursor-following effects that never stop — they signal the interface is alive.
4. **Motion must be motivated:** Every animation can be justified in one sentence (hierarchy / storytelling / feedback / state transition). No GSAP-for-show.

### Implementation Patterns by Motion Intensity

**Level 1-3 (Static):**
* CSS `:hover` and `:active` states only
* No automatic animations
* `prefers-reduced-motion` is the default mode

**Level 4-7 (Fluid CSS):**
```css
transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
animation-delay: calc(var(--index) * 100ms); /* cascade for load-ins */
```
* Focus on `transform` and `opacity` only
* Use CSS `@keyframes` or Motion's `initial`/`animate` props

**Level 8-10 (Advanced Choreography):**
* Complex scroll-triggered reveals, parallax, scroll-driven animation
* Use GSAP ScrollTrigger for full-page scrolltelling and scroll hijacks
* Isolate in dedicated leaf components with `useEffect` cleanup
* **NEVER mix GSAP/Three.js with Motion in the same component tree** — they fight over frames

### Forbidden Animation Patterns (Hard Bans)
* **`window.addEventListener("scroll", ...)`** is banned. Use Motion's `useScroll()`, GSAP's `ScrollTrigger`, IntersectionObserver, or CSS `animation-timeline: view()`.
* **Custom scroll progress calculations using `window.scrollY`** in React state — re-renders on every frame.
* **`requestAnimationFrame` loops that touch React state.** Use motion values (`useMotionValue` + `useTransform`) instead.
* **Layout Transitions:** Use Motion's `layout` and `layoutId` props for visible state changes only. Do not wrap static content in `layout` props "for safety".
* **Staggered Orchestration:** Use `staggerChildren` (Motion) or CSS cascade (`animation-delay: calc(var(--index) * 100ms)`) for reveal moments where sequence matters.

### Reduced Motion (mandatory)
* **Any motion above `MOTION_INTENSITY > 3` MUST honor `prefers-reduced-motion`.** This is non-negotiable.
* In Motion: wrap with `useReducedMotion()` and degrade to static.
* In CSS: gate animations behind `@media (prefers-reduced-motion: no-preference)` or provide an override block under `@media (prefers-reduced-motion: reduce)`.
* Infinite loops, parallax, scroll-hijack, and magnetic physics MUST collapse to static / instant under reduced motion.

## Image Strategy (3-Tier Priority)

1. **Image Generation** — use AI image generation tools when available for custom visuals
2. **Picsum Seed** — `https://picsum.photos/seed/{descriptive-string}/{w}/{h}` for realistic photo placeholders
3. **Explicit Placeholder Slots** — labeled empty slots with dimensions and alt text when neither is available

**NO div-based fake screenshots.** Never build a fake product UI out of `<div>` rectangles to simulate a screenshot. Use real images, generated images, or skip the preview entirely.

## Content Density Rules

* **Spec sheet ban:** No 20-row data tables, no fake-precise specs without justification, ≤ 25-word sub-paragraphs by default.
* **Long lists must use marquee/carousel/tabs** — not default `<ul>` with `divide-y` for > 5 items.
* **Quotes ≤ 3 lines** of body, attribution clean (no em-dash).

## Redesign Protocol

This skill handles **greenfield builds AND redesigns**. Misclassifying the mode is the single biggest source of bad redesign output.

### Detect the Mode (first action)
* **Greenfield** — no existing site, or full overhaul approved. Dial baseline from Section 3.
* **Redesign - Preserve** — modernise without breaking the brand. Audit first, extract brand tokens, evolve gradually.
* **Redesign - Overhaul** — new visual language on top of existing content. Treat as greenfield for visuals; preserve content and IA.

If ambiguous, ask **once**: *"Should this redesign preserve the existing brand, or are we starting visually from scratch?"*

### Audit Before Touching
Document the current state before proposing changes:
* **Brand tokens** — primary / accent colors, type stack, logo treatment, radii.
* **Information architecture** — page tree, primary nav, key conversion paths.
* **Content blocks** — what exists, what's doing work, what's filler.
* **Patterns to preserve** — signature interactions, recognisable hero, copy voice.
* **Patterns to retire** — AI-slop tells, broken layouts, dead links, generic stock imagery, perf traps.
* **Dial reading of the existing site** — infer current `DESIGN_VARIANCE` / `MOTION_INTENSITY` / `VISUAL_DENSITY`. That's your starting point, not the baseline.
* **SEO baseline** — current ranking pages, meta titles, structured data, OG cards. **SEO migration is the #1 redesign risk.**

### Preservation Rules
* **Do not change information architecture** unless asked. Keep page slugs, anchor IDs, primary nav labels stable for SEO and muscle memory.
* **Extract brand colors before applying Section 4 color rules.** A brand that is already purple stays purple — apply the LILA RULE's override.
* **Preserve copy voice** unless asked for a rewrite. Visual modernisation ≠ content rewrite.
* **Honor existing accessibility wins.** Do not regress focus states, alt text, keyboard nav, contrast.
* **Respect existing analytics events.** Do not rename buttons, form fields, section IDs that downstream tracking depends on.

### Modernisation Levers (priority order)
Apply in order — stop when the brief is satisfied:
1. **Typography refresh** — biggest visual lift per unit of risk.
2. **Spacing & rhythm** — increase section padding, fix vertical rhythm.
3. **Color recalibration** — desaturate, unify neutrals, keep brand accent.
4. **Motion layer** — add `MOTION_INTENSITY`-appropriate micro-interactions to existing components.
5. **Hero & key-section recomposition** — restructure top-of-funnel using Section 10 vocabulary.
6. **Full block replacement** — only when the existing block is unsalvageable.

### Decision Tree: Targeted Evolution vs Full Redesign
* IA, content, and SEO sound → **targeted evolution** (Levers 1-4). ~70% of value at ~40% of risk.
* Visual debt is structural (broken IA, no design system, broken mobile) → **full redesign** with strict content preservation.
* Brand itself is changing → **greenfield**.

### What Never Changes Silently
Never modify without explicit user approval:
* URL structure / route slugs.
* Primary nav labels.
* Form field names or order (breaks analytics + autofill).
* Brand logo or wordmark.
* Existing legal / consent / cookie copy.

## Anti-Slop Rules & Slop Diagnostic

### The Ten Tells (Score Before You Fix)

AI design slop has a tiny, predictable failure distribution — designers asked to label AI UIs collapse the "this is AI" signal down to about ten tells. Before polishing or repairing an artifact, run this as an explicit self-audit and write a short report. **Diagnose first, treat second** — auditing and fixing in one breath fails, because the model's prior outweighs the instruction and it repeats the mistake (recolors when it needed re-layout, polishes type on a composition problem).

The ten tells (presence of each = one point of slop; lower is better):

1. **Tech gradient** — blue/violet/indigo glossy gradient on everything.
2. **Generic tech hue** — the default accent is indigo/violet (not chosen for the brand, just the model's favorite).
3. **Feature-tile grid** — icon + heading + sentence × 3, all equal weight, nothing prioritized.
4. **Accent rail** — a colored left strip on cards: decoration pretending to be organization.
5. **Unearned blur** — glassmorphism with no real depth/elevation system behind it.
6. **Monument stat** — oversized numbers filling space that should carry product story.
7. **Icon topper** — a rounded-square icon centered above every heading (Tailwind-template filler).
8. **Center stack** — everything centered because no real composition was committed to.
9. **Default type** — Inter (or system-ui) used by default rather than chosen.
10. **Wrong surface** — the composition doesn't match the surface (e.g. a hero on a Monitor surface). This is the root cause behind most of the others.

How to run it:
- Score the artifact out of 10 (10 = maximum slop). State the score and list which tells fired, in one short report.
- Treat the report as **context, not a to-do list** — it tells you *where* to spend repair effort, it does not dictate edits.
- Then repair, matched to the diagnosis:
  - tells 3, 8, 10 → **re-layout / re-compose** (revisit the surface choice — do not recolor).
  - tells 1, 2, 9 → **recolor / re-typeset** (palette and type are genuinely the problem here).
  - tells 4, 5, 6, 7 → **remove the decoration**; replace it with real hierarchy (scale, weight, spacing).
- Re-score after repairing. Do not declare done while compositional tells (3, 8, 10) are still firing — those are causes, the rest are usually symptoms.

### Additional Layout Constraints (from taste-skill)

* **Eyebrow count:** Count instances of `uppercase tracking` micro-labels above section headlines across all components. Count ≤ ceil(sectionCount / 3)? Hero counts as 1.
* **Split-header ban:** No "left big headline + right small explainer paragraph" pattern as a section header (vertical stack instead).
* **Zigzag alternation cap:** No 3+ consecutive sections with the same image+text-split layout.
* **No duplicate CTA intent:** No two CTAs with the same intent ("Get in touch" + "Let's talk" both on page = Fail).
* **Logo wall = logo only:** No industry / category labels printed below logos.
* **Bento background diversity:** At least 2-3 bento cells have real visual variation (image, gradient, pattern), not all white-on-white text cards.
* **"Used by / Trusted by" logo wall** lives UNDER the hero, not inside it, uses REAL SVG logos (Simple Icons / devicon) or generated SVG marks, NOT plain text wordmarks.

## Em-Dash Ban (Non-Negotiable)

**Em-dash (`—`) is COMPLETELY banned.** It is the LLM's signature stylistic crutch and it is the #1 visual Tell in production tests. There is no "limited use" allowance, no "natural language frequency" allowance, no "in body copy is fine" allowance. None.

* **Banned in headlines.** Use a period or a comma.
* **Banned in eyebrows / labels / pills / button text / image captions / nav items.** Replace with line breaks, columns, or hairlines.
* **Banned in body copy.** Restructure the sentence: two sentences with a period, OR a comma, OR parentheses, OR a colon.
* **Banned in quote attribution.** Use a normal hyphen with spaces (` - `) or a line break + smaller-weight name.
* **Banned in en-dash form too (`–`) when used as a separator.** Date ranges (`2018-2026`) use a hyphen. Number ranges (`€40-80k`) use a hyphen.

The ONLY permitted dash characters on the page are:
* Regular hyphen `-` (for compound words, ranges, line dividers in markup)
* Minus sign in math (`-5°C`)

If your output contains a single `—` or `–` anywhere visible to the user, the output fails the Pre-Flight Check and must be rewritten. This rule is non-negotiable. The agent has historically ignored em-dash limits when phrased as "use sparingly." The phrasing here is binary: zero em-dashes.

## AI Tells (Forbidden Patterns)

Avoid these signatures unless the brief explicitly asks for them.

### Visual & CSS
* **NO neon / outer glows** by default. Use inner borders or subtle tinted shadows.
* **NO pure black (`#000000`).** Off-black, zinc-950, or charcoal.
* **NO oversaturated accents.** Desaturate to blend with neutrals.
* **NO excessive gradient text** for large headers.
* **NO custom mouse cursors.** Outdated, accessibility-hostile, perf-hostile.

### Typography
* **AVOID Inter as default.** See Section 4.1. Override path exists.
* **NO oversized H1s** that just scream. Control hierarchy with weight + color, not raw scale.
* **Serif constraints:** Serif for editorial / luxury / publication. Not for dashboards.

### Layout & Spacing
* **Mathematically perfect** padding and margins. No floating elements with awkward gaps.
* **NO 3-column equal feature cards.** The generic "three identical cards horizontally" feature row is banned. Use 2-column zig-zag, asymmetric grid, scroll-pinned, or horizontal-scroll alternative.

### Content & Data ("Jane Doe" Effect)
* **NO generic names.** "John Doe", "Sarah Chan", "Jack Su" → use creative, realistic, locale-appropriate names.
* **NO generic avatars.** No SVG "egg" or Lucide user icons → use believable photo placeholders or specific styling.
* **NO fake-perfect numbers.** Avoid `99.99%`, `50%`, `1234567`. Use organic, messy data (`47.2%`, `+1 (312) 847-1928`).
* **NO startup-slop brand names.** "Acme", "Nexus", "SmartFlow", "Cloudly" → invent contextual, premium names that sound real.
* **NO filler verbs.** "Elevate", "Seamless", "Unleash", "Next-Gen", "Revolutionize" → concrete verbs only.

### External Resources & Components
* **NO hand-rolled SVG icons.** Use Phosphor / HugeIcons / Radix / Tabler. Lucide on explicit request only.
* **Hand-rolled decorative SVGs strongly discouraged** as default.
* **NO div-based fake screenshots.** Never build a fake product UI out of `<div>` rectangles to simulate a screenshot. Use real images, generated images, or skip the preview.
* **NO broken Unsplash links.** Use `https://picsum.photos/seed/{descriptive-string}/{w}/{h}`, or generated photo placeholders, or actual assets.
* **shadcn/ui customization:** Allowed, but NEVER in default state. Customize radii, colors, shadows, typography to the project aesthetic.

### Production-Test Tells (banned outright)

These patterns came out of real LLM-generated landing-page tests. They are the signatures the model defaults to when it tries to "look designed." Treat them as hard bans unless the brief explicitly calls for one.

**Hero & top-of-page:**
* **NO version labels in the hero.** `V0.6`, `v2.0`, `BETA`, `INVITE-ONLY PREVIEW`, `EARLY ACCESS`, `ALPHA` — banned as default eyebrows. Only acceptable when the brief is explicitly about a product launch / preview status.
* **NO "Brand · No. 01"-style sub-eyebrows.** "Marrow · No. 01 · The 6-quart" type micro-meta lines. Skip them.

**Section numbering & micro-labels:**
* **NO section-number eyebrows.** `00 / INDEX`, `001 · Capabilities`, `002 · Featured commission`, `06 · how it works` — banned. Eyebrows should name the topic in plain language, not enumerate.
* **NO `01 / 4`-style pagination on images or bento tiles.** If the user can count, they don't need the label.

**Separators & dots:**
* **The middle-dot (`·`) is rationed.** Maximum 1 per line in metadata strips. Do NOT use it as the default separator for everything ("foo · bar · baz · qux · quux"). If you need a separator family, prefer line breaks, hairlines, or columns.
* **NO decorative colored status dots on every list/nav/badge.** A colored dot before "ONE Q4 SLOT OPEN" or before every nav link — banned by default. Acceptable only when the dot conveys actual semantic state (a server status, an availability flag) and is used sparingly.

**Em-dashes & typography flourishes:**
* **NO em-dash (`—`) as a design element OR anywhere else.** See Section 9.G above for the complete ban. The em-dash character is forbidden in headlines, eyebrows, pills, body copy, quotes, attribution, captions, button text, and alt text. Use the regular hyphen (`-`).
* **NO `<br>`-broken-and-italicized headlines** as a default "design move." Headlines should read naturally first, get clever only when the brief demands it.

**Fake product previews:**
* **NO div-based fake product UI in the hero** (fake task list, fake terminal, fake dashboard built from styled divs). It is the #1 LLM-design Tell. Use a real screenshot, a generated image, a real component preview, or none at all.

**Marketing-copy Tells:**
* **NO "Quietly in use at" / "Quietly trusted by"** social-proof headers. Use natural language: "Trusted by", "Used at", "Customers include", or skip the heading entirely if the logos speak.
* **NO generic step labels.** "Stage 1 / Stage 2 / Stage 3", "Step 1 / Step 2 / Step 3" — banned. The actual step content is the label. If you must show progression, use the verb-noun directly ("Install", "Configure", "Ship") not "Stage 1: Install".

**Pills, labels and version stamps:**
* **NO pills/labels/tags overlaid on images.** No `<span>` overlays on photos with tags like `Brand · 02`, `PLATE · BRAND`. Either let the image speak alone, or add a caption directly below (outside the image).
* **NO version footers on marketing pages.** Footer strings like `v1.4.2`, `Build 0048` are CLI / devtool fixtures, not landing-page content. Banned on marketing/landing/portfolio pages.

**Decoration text strips:**
* **NO decoration text strip at hero bottom.** Patterns like `BRAND. MOTION. SPATIAL.` as a small mono-caps strip across the bottom of the hero are an agency-portfolio cliché. Banned by default. Only acceptable when the strip carries real, navigable links (sticky bottom nav) or real status info.

**Lists, dividers and scoring:**
* **NO `border-t` + `border-b` on every row of a long list / spec table.** Pick one (bottom-border between rows OR top-border above the group) and use it sparsely. A 10-row spec table with hairlines under each row is the laziest layout.
* **NO scoring/progress bars with filled background tracks** as comparison visuals. If you need to show "X out of Y" comparisons, prefer a number + small icon, or a tiny inline bar WITHOUT a background track.

**Locale, time, scroll cues:**
* **Locale / city-name / time / weather strips are banned for 99% of briefs.** Allowed ONLY when: the brief explicitly describes a globally-distributed studio with timezone-relevant work, OR a travel-focused brand, OR a real-world physical venue. A single contact-address mention in the footer is fine; an atmospheric locale strip is not.
* **Scroll cues are banned.** `Scroll`, `↓ scroll`, `Scroll to explore` — if the user has not scrolled yet, they are looking at the hero. They know what scroll is. The bottom of the viewport does not need a label.

## Reference Vocabulary (Pattern Names)

This is a vocabulary, not a library. The agent should KNOW these pattern names to communicate about them, design with them in mind, and reach for them when the design read calls for them.

### Hero Paradigms
* **Asymmetric Split Hero** — Text on one side, asset on the other, generous white space.
* **Editorial Manifesto Hero** — Large type, no asset, almost-poster.
* **Video / Media Mask Hero** — Type cut out as mask over video background.
* **Kinetic-Type Hero** — Animated typography as the primary visual.
* **Curtain-Reveal Hero** — Hero parts on scroll like a curtain.
* **Scroll-Pinned Hero** — Hero stays pinned while content scrolls behind.

### Navigation & Menus
* **Mac OS Dock Magnification** — Edge nav, icons scale fluidly on hover.
* **Magnetic Button** — Pulls toward cursor.
* **Gooey Menu** — Sub-items detach like viscous liquid.
* **Dynamic Island** — Morphing pill for status / alerts.
* **Contextual Radial Menu** — Circular menu expanding at click point.
* **Floating Speed Dial** — FAB springing into curved secondary actions.
* **Mega Menu Reveal** — Full-screen dropdown, stagger-fade content.

### Layout & Grids
* **Bento Grid** — Asymmetric tile grouping (Apple Control Center).
* **Masonry Layout** — Staggered grid, no fixed row height.
* **Chroma Grid** — Borders / tiles with subtle animating gradients.
* **Split-Screen Scroll** — Two halves sliding in opposite directions.
* **Sticky-Stack Sections** — Sections that pin and stack on scroll.

### Cards & Containers
* **Parallax Tilt Card** — 3D tilt tracking mouse coordinates.
* **Spotlight Border Card** — Borders illuminate under cursor.
* **Glassmorphism Panel** — Frosted glass with inner refraction.
* **Holographic Foil Card** — Iridescent rainbow shift on hover.
* **Tinder Swipe Stack** — Physical card stack, swipe-away.
* **Morphing Modal** — Button expands into its own dialog.

### Scroll Animations
* **Sticky Scroll Stack** — Cards stick and physically stack.
* **Horizontal Scroll Hijack** — Vertical scroll → horizontal pan.
* **Locomotive / Sequence Scroll** — Video / 3D sequence tied to scrollbar.
* **Zoom Parallax** — Central background image zooming on scroll.
* **Scroll Progress Path** — SVG line drawing along scroll.
* **Liquid Swipe Transition** — Page transition like viscous liquid.

### Galleries & Media
* **Dome Gallery** — 3D panoramic gallery.
* **Coverflow Carousel** — 3D carousel with angled edges.
* **Drag-to-Pan Grid** — Boundless draggable canvas.
* **Accordion Image Slider** — Narrow strips expanding on hover.
* **Hover Image Trail** — Mouse leaves popping image trail.

### Typography & Text
* **Kinetic Marquee** — Endless text bands reversing on scroll.
* **Text Mask Reveal** — Massive type as transparent window to video.
* **Text Scramble Effect** — Matrix-style decoding on load / hover.
* **Circular Text Path** — Text curving along spinning circle.

### Micro-Interactions & Effects
* **Particle Explosion Button** — CTA shatters into particles on success.
* **Liquid Pull-to-Refresh** — Reload indicator like detaching droplets.
* **Skeleton Shimmer** — Shifting light reflection across placeholders.
* **Directional Hover-Aware Button** — Fill enters from cursor's exact side.
* **Ripple Click Effect** — Wave from click coordinates.
* **Mesh Gradient Background** — Organic lava-lamp blobs.

### Animation Library Choice
* **Motion (`motion/react`)** — default for UI / Bento / state-change motion.
* **GSAP + ScrollTrigger** — for full-page scrolltelling and scroll hijacks. Isolate in dedicated leaf components with `useEffect` cleanup.
* **Three.js / WebGL** — for canvas backgrounds and 3D scenes. Same isolation rule.
* **NEVER mix GSAP / Three.js with Motion in the same component tree.** They fight over the same frames.

## Performance & Accessibility Guardrails

### Hardware Acceleration
* Animate ONLY `transform` and `opacity`. Never animate `top`, `left`, `width`, `height`.
* Use `will-change: transform` sparingly — only on elements that will actually animate.

### Core Web Vitals Targets
* **LCP** < 2.5s. Hero image must be `next/image priority` or preloaded.
* **INP** < 200ms. Heavy work off main thread.
* **CLS** < 0.1. Reserve space for images, fonts, embeds.
* Run Lighthouse before declaring a page done.

### DOM Cost
* Apply grain / noise filters EXCLUSIVELY to fixed, `pointer-events-none` pseudo-elements (e.g., `fixed inset-0 z-[60] pointer-events-none`). NEVER on scrolling containers — continuous GPU repaints destroy mobile FPS.
* Be aware of bundle size. Motion is not tiny. Three.js is large. Lazy-load anything that's not above-the-fold.

### Z-Index Restraint
NEVER spam arbitrary `z-50` or `z-10`. Use z-index strictly for systemic layer contexts (sticky navbars, modals, overlays, grain). Document the z-index scale in a project constants file.

## Variation Rules

When exploring, default to at least three options:

1. **Conservative** — closest to existing patterns / lowest risk
2. **Strong-fit** — best interpretation of the brief
3. **Divergent** — more novel, useful for discovering taste boundaries

Variations can explore: layout, hierarchy, type scale, density, color posture, surface treatment, motion, interaction model, copy structure, component shape.

Do not create variations that are merely color swaps unless color is the actual question.

When the user picks a direction, consolidate. Do not leave the project as a pile of options forever.

## Tweakable Designs in CLI/API Mode

Still preserve the idea: when useful, add in-page controls called `Tweaks`.

A good `Tweaks` panel can control:
- theme mode
- layout variant
- density (via VISUAL_DENSITY)
- accent color
- type scale
- motion on/off
- copy variant
- component variant

Keep it small and unobtrusive. The design should look final when tweaks are hidden. Persist tweak values with localStorage when helpful.

## Content Discipline

Do not add filler content. Every element must earn its place.

Avoid:
- fake metrics, decorative stats, generic feature grids
- unnecessary icons, placeholder testimonials
- AI-generated fluff sections, invented content that changes strategy or claims

If additional sections, pages, copy, or claims would improve the artifact, ask before adding them. When copy is necessary but not final, mark it as draft or placeholder.

## Source-Code Fidelity

When recreating or extending a UI from a repo:
1. inspect the repo tree
2. identify the actual UI source files
3. read theme/token/global style/component files
4. lift exact values where appropriate
5. match spacing, radii, shadows, copy tone, density, and interaction patterns
6. only then design or modify

Do not build from memory when source files are available.

## Out of Scope

This skill is NOT for:
* Dashboards / dense product UI / admin panels (use Fluent, Carbon, Atlassian, or Polaris from Section 2.A).
* Data tables (use TanStack Table or AG Grid).
* Multi-step forms / wizards (use Form-specific patterns; this skill won't make them better).
* Code editors (use Monaco / CodeMirror with their official skinning).
* Native mobile (use Apple HIG / Material directly).
* Realtime collab UIs (presence, cursors, OT-aware — different problem class).

If the brief is one of the above, **say so explicitly**, point to the right tool, and only apply this skill's marketing-page / about-page / landing-page parts to the surfaces where they apply.

## Final Pre-Flight Check

Run this matrix before outputting code. This is the last filter.

**THIS IS NOT OPTIONAL. Run every box. If any box fails, the output is not done.**

- [ ] **Brief inference** declared (Section 2 one-liner)?
- [ ] **Dial values** explicit and reasoned from the brief, not silently using baseline?
- [ ] **Design system** chosen from Section 2.A if applicable, or aesthetic labeled honestly?
- [ ] **Redesign mode** detected and audit performed (if applicable, Section Redesign Protocol)?
- [ ] **ZERO em-dashes (`—`) anywhere on the page.** Headlines, eyebrows, pills, body, quotes, attribution, captions, buttons, alt text. Zero. (Section 9.G — non-negotiable.)
- [ ] **Page Theme Lock**: ONE theme (light, dark, or auto) for the whole page. No section flips to inverted mode mid-page?
- [ ] **Color Consistency Lock**: one accent color used identically across all sections?
- [ ] **Shape Consistency Lock**: one corner-radius system applied consistently?
- [ ] **Button Contrast Check**: every CTA text is readable against its background (no white-on-white, WCAG AA 4.5:1)?
- [ ] **CTA Button Wrap**: no CTA label wraps to 2+ lines at desktop?
- [ ] **Form Contrast Check**: form inputs, placeholders, focus rings, labels all pass WCAG AA against the section background?
- [ ] **Serif discipline**: if a serif is used, it is NOT Fraunces or Instrument_Serif (or it is, with explicit brand justification)? Different serif from your previous project?
- [ ] **Premium-consumer palette check**: if the brief is premium-consumer, the palette is NOT the AI-default beige+brass+oxblood+espresso family? Different family from your previous premium-consumer project?
- [ ] **Italic descender clearance**: every italic word with `y g j p q` has `leading-[1.1]` min + `pb-1` reserve?
- [ ] **Hero fits the viewport**: headline ≤ 2 lines, subtext ≤ 20 words AND ≤ 4 lines, CTA visible without scroll, font scale planned around image?
- [ ] **Hero top padding**: max `pt-24` at desktop, hero content does not float halfway down the viewport?
- [ ] **Hero stack discipline**: max 4 text elements in hero (eyebrow OR brand strip, headline, subtext, CTAs)? No tiny tagline below CTAs, no trust micro-strip in hero?
- [ ] **EYEBROW COUNT (mechanical)**: count instances of `uppercase tracking` micro-labels above section headlines across all components. Count ≤ ceil(sectionCount / 3)? Hero counts as 1.
- [ ] **Split-Header Ban**: no "left big headline + right small explainer paragraph" pattern as a section header?
- [ ] **Zigzag Alternation Cap**: no 3+ consecutive sections with the same image+text-split layout?
- [ ] **No Duplicate CTA Intent**: no two CTAs with the same intent on page?
- [ ] **Logo wall = logo only**: no industry / category labels printed below logos?
- [ ] **Bento Background Diversity**: at least 2-3 bento cells have real visual variation, not all white-on-white text cards?
- [ ] **"Used by / Trusted by" logo wall** lives UNDER the hero, uses REAL SVG logos or generated SVG marks, NOT plain text wordmarks?
- [ ] **Copy Self-Audit**: every visible string re-read, no grammatically-broken or AI-hallucinated phrases shipped?
- [ ] **Motion motivated**: every animation can be justified in one sentence (hierarchy / storytelling / feedback / state transition), no GSAP-for-show?
- [ ] **Marquee max-one-per-page**: no two horizontal marquees on the same page?
- [ ] **Navigation on ONE line** at desktop, height ≤ 80px?
- [ ] **Section-Layout-Repetition check**: no two sections share the same layout family (at least 4 different families across 8 sections)?
- [ ] **Bento has rhythm AND exact cell count** (N items → N cells, no empty cells in middle or at end)?
- [ ] **Long lists use the right UI component** (not default `<ul>` with `divide-y` for > 5 items)?
- [ ] **Real images used** (gen-tool first, then Picsum-seed, then explicit placeholder slots) — NO div-based fake screenshots, NO hand-rolled decorative SVGs?
- [ ] **No pills/labels overlaid on images**?
- [ ] **No photo-credit captions as decoration**?
- [ ] **No version footers** (`v1.4.2`, `Build 0048`) on marketing pages?
- [ ] **No micro-meta-sentences** under eyebrows?
- [ ] **No decoration text strip at hero bottom**?
- [ ] **No floating top-right sub-text** in section headings?
- [ ] **No scoring/progress bars with filled background tracks** as comparison visuals?
- [ ] **No locale / city-name / time / weather strips** unless brief is genuinely globally-distributed or place-focused?
- [ ] **No scroll cues** (`Scroll`, `↓ scroll`, `Scroll to explore`)?
- [ ] **No version labels in hero** (V0.6, BETA, INVITE-ONLY) unless the brief is a launch?
- [ ] **No section-numbering eyebrows** (`00 / INDEX`, `001 · Capabilities`)?
- [ ] **No decorative dots** (zero by default, only for real semantic state)?
- [ ] **No `border-t` + `border-b` on every row** of long lists / spec tables?
- [ ] **Content density sane**: no 20-row data tables, ≤ 25-word sub-paragraphs by default?
- [ ] **Quotes ≤ 3 lines** of body, attribution clean (no em-dash)?
- [ ] **Motion claimed = motion shown**: if `MOTION_INTENSITY > 4`, page actually animates, not just claimed?
- [ ] **GSAP sticky-stack / horizontal-pan** implemented per canonical skeleton (`start: "top top"`, `pin: true`, correct scrub)?
- [ ] **No `window.addEventListener('scroll')`** — using Motion `useScroll()` / ScrollTrigger / IntersectionObserver / CSS scroll-driven animations only?
- [ ] **Reduced motion** wrapped for everything `MOTION_INTENSITY > 3`?
- [ ] **Dark mode** tokens defined and tested in both modes?
- [ ] **Mobile collapse** explicit (`w-full`, `px-4`, `max-w-7xl mx-auto`) for high-variance layouts?
- [ ] **Viewport stability**: `min-h-[100dvh]`, never `h-screen`?
- [ ] **`useEffect` animations** have strict cleanup functions?
- [ ] **Empty / loading / error** states provided?
- [ ] **Cards omitted** in favor of spacing where possible?
- [ ] **Icons** from an allowed library only (Phosphor / HugeIcons / Radix / Tabler), no hand-rolled SVG paths?
- [ ] **Motion** isolated in client-leaf components with `'use client'` at the top, memoized?
- [ ] **No AI Tells** from Section 9 (Inter as default, AI-purple, three-equal cards, Jane Doe, Acme)?
- [ ] **Core Web Vitals** plausibly hit (LCP < 2.5s, INP < 200ms, CLS < 0.1)?
- [ ] **One design system** per project (no Material + shadcn mixed)?

If a single checkbox cannot be honestly ticked, the page is not done. Fix it before delivering.

## Verification

Before final response, verify as much as the environment allows.

Minimum:
- file exists at the stated path
- HTML is saved completely
- obvious syntax issues are checked

Better:
- open in a browser tool and check console errors
- inspect screenshots at the primary viewport
- test key interactions
- test light/dark or variants if present
- test responsive breakpoints if relevant

If verification is limited by environment, say exactly what was and was not verified. Never say "done" if the file was not actually written.

## Final Response Format

Keep final responses short. Include:
- artifact path
- what it contains
- verification status
- next suggested action, if useful

Example:

```text
Created: /path/to/Prototype.html
It includes 3 layout variants, a Tweaks panel for density/theme, and responsive behavior.
Verified: file exists and opened cleanly in browser, no console errors.
Next: pick the strongest direction and I'll tighten copy + motion.
```

## Pitfalls

- Do not paste hosted tool schemas into a skill. They cause fake tool calls.
- Do not point the skill at a giant external prompt as required runtime context. That creates drift.
- Do not strip the design doctrine while removing tool plumbing.
- Do not over-ask when the user already gave enough direction.
- Do not under-ask for high-fidelity work with no brand context.
- Do not produce generic SaaS layouts and call them designed.
- Do not claim browser verification unless it actually happened.
- **Do not silently use baseline dials (8/6/4) without reasoning from the brief.** Always state dial values explicitly.
