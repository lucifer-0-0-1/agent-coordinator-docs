# Agent Coordinator: Technical Architecture

## 1. Dynamic Master-Worker Architecture

Agent Coordinator uses a **flexible, peer-to-peer multi-harness model**. Rather than fixing one harness permanently as the master and others as workers:
- **Any active interactive agent** (Claude Code, Google Antigravity, OpenCode, or OpenAI Codex) can act as the **Master** (Architect, Watcher, Reviewer).
- **Any supported harness** can be dispatched as an **Autonomous Worker** (Implementer, Reporter) into an isolated Git worktree or workspace.
- By default, tasks route to **Claude Code** (`claude`), but can be dispatched to `agy`, `opencode`, or `codex` based on task requirements, model specialization, and hardware needs.

```
                    ACTIVE HARNESS (Master / Watcher)
              (Claude Code · Antigravity · OpenCode · Codex)
                     Architect · Planner · Reviewer
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         ▼                         ▼                         ▼
   WORKER HARNESS            WORKER HARNESS            WORKER HARNESS
   (e.g. Antigravity)        (e.g. OpenCode)           (e.g. Codex)
  Gemini 3.8 / Pro GPU       Zero-Cost / Free          OpenAI Models
         │                         │                         │
Worktree 1 (Isolated)     Worktree 2 (Isolated)     Worktree 3 (Isolated)
`agent/<worker>/<id>`     `agent/<worker>/<id>`     `agent/<worker>/<id>`
         │                         │                         │
         └─────────────────────────┼─────────────────────────┘
                                   ▼
                       Structured TASK_REPORT.json
                                   │
                                   ▼
                   Diff Review & Merge by Master
```

### Harness Specialization Matrix
| Harness | Primary Strength | Recommended Use Cases | Default Role |
|---|---|---|---|
| **Claude Code (`claude`)** | Reasoning, architecture, code review | Complex refactoring, architecture design, API contracts | Default Master & Worker |
| **Antigravity (`agy`)** | Extreme context window (1M+ tokens), GPU tasks | CUDA/ML jobs (`--wait-gpu`), full-repo migrations, deep test suites | Worker / Implementer |
| **OpenCode (`opencode`)** | Multi-model freedom, zero token cost (local/free tiers) | High-volume boilerplate, documentation, unit test scaffolding | Worker / Implementer |
| **OpenAI Codex (`codex`)** | Fast code synthesis, Python/JS scripting | Algorithmic implementations, script generation, focused functions | Worker / Implementer |

### Model Specification & Token Conservation

When dispatching tasks to worker harnesses, **always specify the intended model directly in the task prompt or ensure it is pre-configured in the harness environment**:
* **The Token Cost of Omission:** If the worker agent is invoked without an explicit model prompt, it often spawns exploratory subagents or queries installed providers and local configuration files (`~/.config/...`, environment variables) to choose an LLM engine. This exploratory discovery can consume 2,000–5,000 tokens before actual task execution begins.
* **Best Practice:** Prompt with clear model targets:
  - *Claude:* `"Refactor auth module using claude-3-7-sonnet..."`
  - *Antigravity:* `"Run model benchmark using gemini-2.5-pro..."`
  - *OpenCode:* `"Scaffold endpoints using deepseek-chat..."`
  - *Codex:* `"Optimize query planner using o3-mini..."`

### Automatic Default Model Fallbacks

If no model is explicitly specified in the prompt or `--model` parameter, each harness automatically applies its pre-configured default model:

| Harness | Default Fallback Model | Override Environment Variable |
|---|---|---|
| **Claude Code (`claude`)** | `claude-3-7-sonnet` | `CLAUDE_MODEL` |
| **Antigravity (`agy`)** | `gemini-3.8-flash-high` | `AGY_MODEL` |
| **OpenCode (`opencode`)** | `openrouter/openrouter/free` | `OPENCODE_MODEL` |
| **OpenAI Codex (`codex`)** | `o3-mini` | `CODEX_MODEL` |

---

## 2. Worktree & Workspace Isolation

The fundamental principle of Agent Coordinator is **Working-Tree Protection**: an autonomous background worker agent must never modify, lock, or corrupt the developer's or primary agent's active workspace.

### Git Worktree Mechanism
When a task is dispatched to any worker (`claude`, `agy`, `opencode`, or `codex`):

1. **Branch Creation:** A dedicated branch is created from the current HEAD:
   ```
   agent/<agent>/<task-id>
   ```
   Example: `agent/claude/20261110-104500-a1b2` or `agent/agy/20261110-104500-c3d4`

2. **Worktree Attachment:** A linked Git worktree is attached at:
   ```
   .coordinator/worktrees/<task-id>/
   ```
   This gives the worker agent a completely independent working directory, index, and `HEAD`, while sharing the underlying `.git` object storage.

3. **Autonomous Execution:** The worker agent runs headlessly inside this directory. Any file edits, unstaged modifications, test outputs, or intermediate commits happen entirely within the worktree.

4. **Lifecycle Teardown:** Upon completion and merge, the worktree is detached and cleaned up:
   ```bash
   git worktree remove --force .coordinator/worktrees/<task-id>
   ```

### Non-Git Mode (Snapshot Workspaces)
For non-git projects or folders where Git is undesirable:
- The `--non-git` / `--no-worktree` flag executes either directly in-place or copies files to `.coordinator/workspaces/<task-id>/`.
- Guarantees compatibility for single-script folders, notebooks, or external drives.

---

## 3. Token Economy Comparison

Conventional multi-agent frameworks pass entire conversation histories between agents or have a master agent read every file the worker touched. This causes token usage to skyrocket.

Agent Coordinator eliminates this waste:

| Interaction Step | Conventional Multi-Agent Framework | Agent Coordinator |
|---|---|---|
| **Dispatch** | Ingests entire conversation history & repo tree (~10k–50k tokens) | Concise task prompt + explicit `--files` list (~200 tokens) |
| **Worker Execution** | Master agent polls LLM repeatedly for status (~2k tokens/min) | Python deterministic process monitoring (**0 tokens**) |
| **GPU/Resource Wait** | LLM sleeps or checks output in loops | Python `nvidia-smi` parser (**0 tokens**) |
| **Status Check** | Full transcript transfer (~15k–30k tokens) | Compact `TASK_REPORT.json` (~100 tokens) |
| **Code Review** | Re-reads all project source files (~20k tokens) | Reviews only `git diff HEAD~1` (~500 tokens) |
| **Review Feedback** | Prompts master agent to rewrite code | One-line targeted guidance via `coord guide` (~50 tokens) |

**Total Token Savings: 85% – 95% per task.**

---

## 4. Headless Agent CLI Invocations

Agent Coordinator interfaces directly with native agent binaries:

### Claude Code (`claude`)
- Binary resolution: `which claude` or standard PATH
- Headless / non-interactive execution command:
  ```bash
  claude --non-interactive -p "<task_prompt>"
  ```
- Logs streamed to `.coordinator/logs/claude_<task_id>_<time>.log`.

### Antigravity (`agy`)
- Binary resolution: `which agy` or `~/.gemini/antigravity-cli/bin/agy`
- Headless execution command:
  ```bash
  agy --workspace <worktree_dir> -p "<task_prompt>"
  ```
- Logs streamed to `.coordinator/logs/agy_<task_id>_<time>.log`.

### OpenCode (`opencode`)
- Binary resolution: `which opencode` or `~/.opencode/bin/opencode` or `~/.local/bin/opencode`
- Headless execution command:
  ```bash
  opencode run --dir <worktree_dir> --auto "<task_prompt>"
  ```
- Logs streamed to `.coordinator/logs/opencode_<task_id>_<time>.log`.

### OpenAI Codex (`codex`)
- Binary resolution: `which codex` or `~/.local/bin/codex`
- Headless non-interactive execution command:
  ```bash
  codex exec -C <worktree_dir> --dangerously-bypass-approvals-and-sandbox --skip-git-repo-check "<task_prompt>"
  ```
- Logs streamed to `.coordinator/logs/codex_<task_id>_<time>.log`.

---

## 5. Guidance Loop Architecture

```
                 Master Reviews Task Report & Diff
                                 │
                                 ▼
                     Issues detected in code?
                                 │
                     ┌───────────┴───────────┐
                     ▼                       ▼
                    YES                      NO
                     │                       │
      `coord guide <task_id> "..."`          ▼
                     │                 `coord_merge` /
                     ▼                 Clean Worktree
         Harness re-enters worktree,
         applies targeted fix,
         re-runs test suite,
         regenerates report
```
