# Coordinator MCP Tool Reference

When the `coordinator` Model Context Protocol (MCP) server is registered with an agent harness (Claude Code, Antigravity, OpenCode, or OpenAI Codex), the following native tools are available.

The server operates under a **Dynamic Master-Worker model**: any interactive agent can call these MCP tools to architect, dispatch, monitor, and merge tasks, and any harness can be dispatched into an isolated workspace as a worker. By default, tasks route to `claude`.

---

### 1. `coord_init`
Scaffolds or verifies repository instruction files for multi-agent coordination.

* **Parameters:**
  * `project_path` *(string, default: ".")*: Target repository or directory path.
  * `force` *(boolean, default: false)*: Overwrite existing instruction files.
  * `create_arch` *(boolean, default: false)*: Create an initial `ARCHITECTURE.md` template.
  * `non_git` *(boolean, default: false)*: Skip Git checks and configure directory snapshot mode.
* **Returns:** Summary of files created or validated (`AGENTS.md`, `CLAUDE.md`, `.agents/AGENTS.md`, `.gitignore`).

---

### 2. `coord_dispatch`
Dispatches an autonomous implementation task to a background worker in an isolated worktree (default: `claude`).

* **Parameters:**
  * `task` *(string, required)*: Clear implementation task description.
  * `agent` *(string, enum: ["claude", "agy", "opencode", "codex"], default: "claude")*: Harness to execute the task.
  * `model` *(string, optional)*: Specific LLM model to use (falls back to each harness's default model: `claude-3-7-sonnet` for claude, `gemini-3.8-flash-high` for agy, `openrouter/openrouter/free` for opencode, `o3-mini` for codex).
  * `files` *(array of strings or comma-separated string, optional)*: Files assigned to this task.
  * `criteria` *(array of strings or comma-separated string, optional)*: Success criteria and test commands (e.g. "pytest tests/").
  * `branch_name` *(string, optional)*: Custom Git branch name.
  * `use_worktree` *(boolean, default: true)*: Create an isolated Git worktree. Set to false for in-place execution.
  * `auto_merge` *(boolean, default: false)*: Automatically merge into active branch if tests pass.
  * `timeout` *(integer, default: 0)*: Timeout in seconds (`0` = unlimited duration).
  * `wait_gpu` *(boolean, default: false)*: Wait until sufficient GPU VRAM is free before starting.
  * `gpu_min_free` *(integer, default: 2048)*: Minimum free VRAM in MB required when `wait_gpu=true`.
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Structured JSON task report (`TASK_REPORT.json`).

---

### 3. `coord_dispatch_parallel`
Executes multiple independent tasks concurrently across any combination of agents in parallel isolated worktrees.

* **Parameters:**
  * `tasks` *(array of objects, required)*:
    * `title` *(string)*: Task title / prompt.
    * `description` *(string, optional)*: Detailed task description.
    * `agent` *(string, enum: ["claude", "agy", "opencode", "codex"], default: "claude")*: Target harness.
    * `files` *(array of strings, optional)*: Targeted files.
    * `criteria` *(array of strings, optional)*: Acceptance criteria.
  * `auto_merge` *(boolean, default: false)*: Auto-merge completed branches that pass verification.
  * `timeout` *(integer, default: 0)*: Timeout in seconds per task.
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Array of completion reports for all parallel tasks.

---

### 4. `coord_guide`
Sends Master code review feedback to an implementer on an existing task, re-entering its isolated workspace to patch issues and re-verify.

* **Parameters:**
  * `task_id` *(string, required)*: Task identifier (e.g. "claude-20261110-104500-a1b2").
  * `feedback` *(string, required)*: Specific code review instructions or bug descriptions.
  * `timeout` *(integer, default: 0)*: Guidance execution timeout.
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Updated `TASK_REPORT.json` reflecting the fixes and re-run tests.

---

### 5. `coord_gpu_status`
Deterministically queries host GPU resources without consuming LLM tokens.

* **Parameters:** None.
* **Returns:** GPU name, total VRAM, free VRAM, utilization percentage, and a list of active compute processes.

---

### 6. `coord_status`
Inspects active coordinator workspaces, running background tasks, harness availability, and saved reports.

* **Parameters:**
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Status summary including active worktree paths, agent availability, and recent reports.

---

### 7. `coord_logs`
Tails execution logs of an active or recent task without consuming large LLM tokens.

* **Parameters:**
  * `task_id` *(string, required)*: The task identifier to inspect.
  * `tail` *(integer, default: 40)*: Number of lines from the end of the log to return.
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Recent log tail with update timestamp and total line count.

---

### 8. `coord_plan`
Validates and saves a structured multi-agent implementation plan between Claude, Antigravity, OpenCode, and Codex.

* **Parameters:**
  * `goal` *(string, required)*: Overall objective of the multi-agent effort.
  * `tasks` *(array of objects, required)*: List of task definitions with assigned agents and file boundaries.
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Validation result and saved plan file path (`.coordinator/plan.json`).

---

### 9. `coord_merge`
Merges a completed task branch and safely cleans up its associated Git worktree.

* **Parameters:**
  * `branch_name` *(string, required)*: The task branch to merge (e.g. "agent/claude/task-1").
  * `into_branch` *(string, default: "main")*: Destination branch.
  * `task_id` *(string, optional)*: Associated task ID for worktree cleanup.
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Merge status and cleanup confirmation.

---

### 10. `coord_clean`
Prunes and deletes all completed coordinator worktrees and associated metadata.

* **Parameters:**
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Count and list of pruned worktrees.
