# Workflows & Prompting Guide

This guide provides practical examples of conversational prompts and CLI commands for common multi-agent development patterns.

---

## 1. Auto-Scaffold Project Instructions

Before running multi-agent tasks, establish clear token efficiency protocols and scope constraints:

### Conversational Prompt (in Claude Code or Cursor)
> *"Initialize this repository for multi-agent coordination with agy and opencode."*

### CLI Equivalent
```bash
coord init
```

### What Happens:
The coordinator automatically creates:
* `AGENTS.md` (canonical multi-agent protocols and token rules)
* `CLAUDE.md` (instructions for Claude as Watcher/Architect)
* `.agents/AGENTS.md` (instructions for Antigravity as Implementer)
* `.gitignore` (excludes `.coordinator/`, `.worktrees/`, `TASK_REPORT.json`, and agent logs)

---

## 2. Dispatch a Single Background Task

Offload a specific feature, bugfix, or test suite to an autonomous worker:

### Conversational Prompt
> *"Have agy implement the CAN frame decoder in `src/can/parser.c` with tests in `tests/test_parser.py`. Ensure all unit tests pass."*

### CLI Equivalent
```bash
coord dispatch "Implement CAN frame parser" \
  --agent agy \
  --files "src/can/parser.c,tests/test_parser.py" \
  --criteria "pytest tests/test_parser.py" \
  --timeout 0
```

---

## 3. Concurrent Multi-Agent Parallel Execution

Run multiple independent tasks simultaneously across different agents without working-tree collisions:

### Conversational Prompt
> *"Divide this feature into three tasks: have agy implement the backend API, have opencode scaffold the React dashboard, and have claude write end-to-end tests. Run them in parallel."*

### CLI Equivalent
```bash
coord parallel \
  --task "Build backend API:agy:server/api.py:pytest tests/test_api.py" \
  --task "Scaffold React Dashboard:opencode:ui/Dashboard.tsx:npm run build" \
  --task "Author integration tests:claude:tests/test_e2e.py:pytest tests/test_e2e.py" \
  --auto-merge
```

Each task runs concurrently in an isolated worktree. If `--auto-merge` is set and tests pass, the coordinator automatically merges the task branch cleanly into the base branch.

---

## 4. GPU Resource Gating

For machine learning training, PyTorch pipelines, or heavy 3D rendering jobs:

### Conversational Prompt
> *"Dispatch model evaluation to agy, but wait until at least 3GB of GPU VRAM is free."*

### CLI Equivalent
```bash
coord dispatch "Train visual embeddings" \
  --agent agy \
  --wait-gpu \
  --gpu-min-free 3000 \
  --timeout 0
```
The coordinator deterministically checks `nvidia-smi` at zero token cost and queues execution until sufficient VRAM is available.

---

## 5. Watcher Guidance & Feedback Loop

If an implementer completes a task but the Watcher notices an edge-case bug in the review:

### Conversational Prompt
> *"Review the diff for task agy-1045. The bitmask fails for 29-bit identifiers; guide agy to fix it."*

### CLI Equivalent
```bash
coord guide agy-1045 "The bitmask fails for 29-bit identifiers; fix in src/can/parser.c and re-run tests."
```

The worker agent re-enters its isolated worktree, applies the targeted fix, re-runs verification tests, and updates `TASK_REPORT.json` without requiring the Watcher to rewrite code manually.
