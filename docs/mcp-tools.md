# MCP Tool Reference

When the `coordinator` Model Context Protocol (MCP) server is registered with your agent harness, the following native tools are available:

---

### `coord_init`
Scaffolds or verifies repository instruction files for multi-agent coordination.

* **Parameters:**
  * `project_path` *(string, default: ".")*: Target repository or directory path.
  * `force` *(boolean, default: false)*: Overwrite existing instruction files.
  * `create_arch` *(boolean, default: false)*: Create an initial `ARCHITECTURE.md` template.
  * `non_git` *(boolean, default: false)*: Configure directory snapshot mode for non-git folders.
* **Returns:** Summary of files created or validated.

---

### `coord_dispatch`
Dispatches an autonomous implementation task to a background worker in an isolated worktree.

* **Parameters:**
  * `task` *(string, required)*: Implementation task prompt.
  * `agent` *(string, enum: ["agy", "opencode", "claude"], default: "agy")*: Harness to execute the task.
  * `files` *(string, optional)*: Comma-separated list of assigned files.
  * `criteria` *(string, optional)*: Success criteria and test commands.
  * `branch_name` *(string, optional)*: Custom Git branch name.
  * `use_worktree` *(boolean, default: true)*: Create an isolated Git worktree. Set to false for in-place execution.
  * `auto_merge` *(boolean, default: false)*: Automatically merge into active branch if tests pass.
  * `timeout` *(integer, default: 0)*: Timeout in seconds (`0` = unlimited duration).
  * `wait_gpu` *(boolean, default: false)*: Wait until GPU VRAM is free before starting.
  * `gpu_min_free` *(integer, default: 2048)*: Minimum free VRAM in MB required.
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Structured JSON task report (`TASK_REPORT.json`).

---

### `coord_dispatch_parallel`
Executes multiple independent tasks concurrently across any combination of agents in parallel isolated worktrees.

* **Parameters:**
  * `tasks` *(array of objects, required)*:
    * `title` *(string)*: Task title / prompt.
    * `agent` *(string, optional)*: "agy", "opencode", or "claude".
    * `files` *(array of strings, optional)*: Targeted files.
    * `criteria` *(array of strings, optional)*: Acceptance criteria.
  * `auto_merge` *(boolean, default: false)*: Auto-merge completed branches that pass verification.
  * `timeout` *(integer, default: 0)*: Timeout in seconds per task.
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Array of completion reports for all parallel tasks.

---

### `coord_guide`
Sends Watcher code review feedback to an implementer on an existing task, re-entering its isolated workspace to patch issues and re-verify.

* **Parameters:**
  * `task_id` *(string, required)*: Task identifier (e.g. "agy-1731234000").
  * `feedback` *(string, required)*: Specific code review instructions or bug descriptions.
  * `timeout` *(integer, default: 0)*: Guidance execution timeout.
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Updated `TASK_REPORT.json` reflecting the fixes and re-run tests.

---

### `coord_gpu_status`
Deterministically queries host GPU resources without consuming LLM tokens.

* **Parameters:** None.
* **Returns:** GPU name, total VRAM, free VRAM, utilization percentage, and a list of active compute processes.

---

### `coord_status`
Inspects active coordinator workspaces, running background tasks, and saved reports.

* **Parameters:**
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Status summary including active worktree paths, agent availability, and recent reports.

---

### `coord_plan`
Validates and saves a structured multi-agent implementation plan between Claude, Antigravity, and OpenCode.

* **Parameters:**
  * `goal` *(string, required)*: Overall objective of the multi-agent effort.
  * `tasks` *(array of objects, required)*: List of task definitions with assigned agents and file boundaries.
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Validation result and saved plan file path.

---

### `coord_merge`
Merges a completed task branch and safely cleans up its associated Git worktree.

* **Parameters:**
  * `branch_name` *(string, required)*: The task branch to merge (e.g. "agent/agy/task-1").
  * `into_branch` *(string, default: "main")*: Destination branch.
  * `task_id` *(string, optional)*: Associated task ID for worktree cleanup.
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Merge status and cleanup confirmation.

---

### `coord_clean`
Prunes and deletes all completed coordinator worktrees and associated metadata.

* **Parameters:**
  * `project_path` *(string, default: ".")*: Repository root directory.
* **Returns:** Count and list of pruned worktrees.
