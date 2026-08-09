---
name: project-graveyard
description: >-
  Use when a project is failing, stalled, or has been abandoned.
  Diagnose the root cause using the causes-of-death taxonomy, decide whether to revive or retire it, and archive artifacts with a structured summary for future reference.
license: Apache-2.0
metadata:
  author: "Shubham Saboo"
  version: "1.0.0"
  source: "https://github.com/Shubhamsaboo/awesome-llm-apps"
compatibility: >-
  Runs locally with Python 3.10+. Uses the `gravedigger` CLI (Python package) for automated analysis, or falls back to manual diagnosis using the taxonomy in references/causes-of-death.md. No network calls required.
---

# Project Graveyard

A structured approach to diagnosing why projects fail and deciding whether to revive or retire them. Instead of silently dropping failed projects, this skill helps you understand what went wrong, learn from it, and make informed decisions about continuation.

## Why This Matters

Projects don't just "stop working" — they die for specific, diagnosable reasons. By cataloging causes of death and making revival/retirement decisions explicit, you:
- Avoid repeating the same failures
- Preserve valuable partial work for future reference
- Make intentional choices about resource allocation
- Build institutional knowledge about what works and what doesn't

## The Causes of Death Taxonomy

Use `references/causes-of-death.md` to diagnose the root cause. Common categories include:

| Category | Description | Revival Probability |
|----------|-------------|---------------------|
| **Scope Creep** | Features added beyond original vision | Medium |
| **Resource Exhaustion** | Time, money, or compute depleted | Low |
| **Technical Debt** | Accumulated complexity without refactoring | High (if addressed) |
| **Abandonment** | No active maintainer | Variable |
| **Dependency Rot** | Outdated libraries breaking compatibility | Medium |
| **Architecture Mismatch** | Wrong tool for the job | Low |

## The Decision Framework

### Revive vs. Retire

Ask these questions in order:

1. **Is the core value proposition still valid?**
   - Yes → Continue to step 2
   - No → Archive immediately

2. **What's the root cause of failure?**
   - Fixable (scope, debt, dependencies) → Consider revival
   - Unfixable (abandonment, wrong architecture) → Retire

3. **What would it take to revive?**
   - Minimal effort → Revive with documented lessons
   - Major rewrite needed → Archive and start fresh

4. **Are there partial artifacts worth preserving?**
   - Yes → Extract and archive with context
   - No → Full retirement

## How to Use This Skill

### Automated Diagnosis (Preferred)

```bash
# Install the gravedigger CLI
pip install gravedigger

# Analyze a project directory
gravedigger diagnose /path/to/project

# Generate a cause-of-death report
gravedigger report /path/to/project --output graveyard-report.md
```

The CLI will:
1. Scan the project for red flags (stale dependencies, large unrefactored files, missing tests)
2. Cross-reference with the causes-of-death taxonomy
3. Generate a structured diagnosis with revival recommendations

### Manual Diagnosis

When the CLI isn't available or you need deeper analysis:

1. **Gather evidence**: Check git history, issue trackers, CI/CD logs
2. **Identify symptoms**: What stopped working? When did it degrade?
3. **Trace to root cause**: Use the taxonomy in `references/causes-of-death.md`
4. **Assess revival feasibility**: Can the root cause be addressed?
5. **Make decision**: Revive with lessons learned, or retire and archive

## Archiving Process

When retiring a project:

1. **Create an archive directory**: `archive/<project-name>-<date>/`
2. **Copy all artifacts**: Code, configs, docs, data
3. **Write a summary**: 
   - What was the goal?
   - What actually happened?
   - What were the root causes of failure?
   - What partial value remains?
4. **Tag with lessons learned**: Extract reusable patterns or warnings

Example archive structure:
```
archive/
  ai-travel-planner-2026-07-11/
    ├── code/              # Original source
    ├── docs/              # Documentation
    ├── config/            # Configuration files
    └── SUMMARY.md         # Lessons learned
```

## Integration with Other Skills

This skill complements:
- **self-improving-agent-skills**: Learn from past failures to improve future skills
- **advisor-orchestrator-worker**: Use advisor consults to validate revival decisions
- **hermes-agent**: Archive Hermes-specific project failures

## References

- `references/causes-of-death.md` — Detailed taxonomy of common failure modes
- `scripts/graveyard.py` — Automated analysis script (if available)

## When NOT to Use This Skill

- Active projects with clear paths forward
- Projects you're currently debugging
- One-off experiments that served their purpose
- When the user explicitly wants to continue without analysis

Use this skill when a project is stuck, failing repeatedly, or has been abandoned and you need to make an informed decision about its future.