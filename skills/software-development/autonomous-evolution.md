---
name: autonomous-evolution
description: A closed-loop workflow for autonomously optimizing code performance through a cycle of observation, mutation, and verification.
---

# autonomous-evolution

## Description
A closed-loop workflow for autonomously optimizing code performance through a cycle of observation, mutation, and verification. It aims to move from a baseline state to an optimized state by iteratively applying intelligent transformations.

## Trigger Conditions
- When the user wants to optimize a specific piece of Python code's performance.
- When tasked with "improving" or "refactoring for efficiency" in a controlled environment.
- During experimental validation of new optimization heuristics.

## Workflow (The Evolution Loop)
1. **Initialization**: Define the absolute path to the `target_file` and the shell command for the `verifier`.
2. **Baseline Establishment**: Execute the `verifier` to record the initial performance metric (e.g., execution time).
3. **Iterative Mutation**:
    - **Mutation**: Apply a code transformation (via regex or string replacement) to the target file.
    - **Verification**: Run the `verifier` command and parse its output for the new metric.
    - **Comparison**: Compare the new metric against the current best.
    - **Decision**: If improved, commit the change; otherwise, stop or revert.
4. **Finalization**: Report the final optimized state and the total improvement factor.

## Pitfalls & Best Practices
- **Absolute Paths Only**: Always use `os.path.abspath()` for all file paths to avoid CWD mismatches between the runner and sub-processes.
- **Robust Parsing**: The verifier must be resilient to extra output (e.g., using regex instead of strict line matching) to handle non-standard stdout/stderr.
- **Environment Isolation**: Run tests in a controlled directory (e.g., `evolution_lab/`) to prevent accidental corruption of the main project.
- **Atomic Mutations**: Ensure mutations are targeted and do not introduce syntax errors.

## Verification Steps
- Confirm that `target_file` has been modified as expected.
- Verify that the final reported metric is lower (for time/error) or higher (for throughput) than the baseline.
