# CLI Reference

Agent Coordinator provides two command aliases: `coord` (compact) and `agent-coordinator` (full).

```text
usage: coord [-h] {init,dispatch,parallel,guide,run,status,logs,clean} ...
```

---

## Command Summary

| Command | Arguments / Flags | Description |
|---|---|---|
| `coord init [path]` | `--force`, `--arch`, `--non-git` | Auto-scaffold or verify `AGENTS.md`, `CLAUDE.md`, `.agents/AGENTS.md`, and `.gitignore`. |
| `coord dispatch "<task>"` | `--agent claude\|agy\|opencode\|codex`<br>`--files "f1,f2"`<br>`--criteria "c1,c2"`<br>`--branch "<name>"`<br>`--no-worktree`<br>`--auto-merge`<br>`--timeout <sec>`<br>`--wait-gpu`<br>`--gpu-min-free <mb>` | Dispatch an implementation task to a background worker in an isolated worktree (default: `claude`). |
| `coord parallel` | `--task "Title:agent:files:criteria"` (repeatable)<br>`--auto-merge`<br>`--no-worktree`<br>`--timeout <sec>` | Run multiple tasks concurrently across different agents in parallel isolated worktrees. |
| `coord guide <task_id> "<feedback>"` | `--timeout <sec>` | Send code review comments to guide a worker agent on an existing task. |
| `coord run "<prompt>"` | `--mode single\|multi`<br>`--agent claude\|agy\|opencode\|codex\|auto`<br>`--no-worktree`<br>`--auto-merge`<br>`--timeout <sec>`<br>`--wait-gpu` | Run a single-agent or multi-agent task (default: `claude`). |
| `coord status` | `--path [dir]` | Display active worktrees, agent availability, GPU health, and recent reports. |
| `coord logs <task_id>` | `--tail <lines>` | Display or tail live execution logs for an active task. |
| `coord clean` | `--path [dir]` | Remove completed coordinator worktrees and prune metadata. |
