# Architecture & Token Economy

This document details the core architectural mechanisms behind Agent Coordinator: Git worktree isolation, snapshot workspaces, and token conservation.

---

## 1. The Multi-Agent Working-Tree Dilemma

When multiple autonomous agents operate in a single codebase, two problems emerge:

1. **State Corruption:** If Agent A modifies a file while Agent B is testing or committing, tests fail unpredictably and Git indexes become locked.
2. **Token Bleed:** If the primary agent monitors a secondary agent by streaming its entire terminal transcript into context, token consumption explodes (often 20,000–50,000 tokens per subtask).

---

## 2. Solution: Git Worktree Isolation

Agent Coordinator utilizes **Git worktrees** to provide strict physical isolation:

```text
Project Root (Active Working Directory)
├── src/
├── tests/
└── .coordinator/
    └── worktrees/
        ├── agy-1731234000/          <-- Isolated Worktree for Antigravity
        │   └── (Branch: agent/agy/agy-1731234000)
        └── opencode-1731234001/     <-- Isolated Worktree for OpenCode
            └── (Branch: agent/opencode/opencode-1731234001)
```

### The Worktree Lifecycle
1. **Branching:** When a task is dispatched, the coordinator branches from the current HEAD:
   ```bash
   git worktree add .coordinator/worktrees/<task-id> -b agent/<agent>/<task-id>
   ```
2. **Headless Execution:** The worker agent launches headlessly inside that directory. Any edits, intermediate commits, and test outputs occur strictly within this worktree.
3. **Report Generation:** When tests pass, the worker generates a structured `TASK_REPORT.json` at the worktree root.
4. **Clean Merge & Pruning:** Once the Watcher verifies the diff, the branch is merged into the active branch and the worktree is detached:
   ```bash
   git worktree remove --force .coordinator/worktrees/<task-id>
   ```

---

## 3. Non-Git Directory Mode (Snapshot Workspaces)

For projects without Git:
* Passing `--non-git` (or `--no-worktree`) switches the coordinator to directory snapshot isolation.
* Tasks execute either in-place or inside `.coordinator/workspaces/<task-id>/`.
* This ensures zero friction for data science notebooks, scripts, or single-file utilities.

---

## 4. Token Economy Comparison

The table below demonstrates the token savings of Agent Coordinator compared to standard multi-agent setups:

| Step | Standard Multi-Agent Setup | Agent Coordinator | Token Savings |
|---|---|---|---|
| **Task Dispatch** | Full conversation history (~15k tokens) | Scoped task + explicit file list (~250 tokens) | **~98%** |
| **Worker Monitoring** | LLM poll loop (~2k tokens/min) | Python deterministic process check (**0 tokens**) | **100%** |
| **GPU / Resource Wait** | LLM sleeps or polls API | Python `nvidia-smi` parser (**0 tokens**) | **100%** |
| **Completion Transfer** | Full raw terminal transcript (~20k tokens) | Compact `TASK_REPORT.json` (~100 tokens) | **~99%** |
| **Code Review** | Re-reads all project source files (~25k tokens) | Reviews only `git diff HEAD~1` (~500 tokens) | **~98%** |
| **Review Feedback** | Prompts master agent to rewrite code | One-line targeted guidance via `coord guide` (~50 tokens) | **~90%** |

**Net Result:** Overall token consumption per task is reduced by **85% – 95%**.
