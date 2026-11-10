# Agent Coordinator: Multi-Agent Parallel Orchestrator

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform: Linux | macOS](https://img.shields.io/badge/Platform-Linux%20%7C%20macOS-lightgrey.svg)](https://github.com/)
[![Harnesses: Claude Code | Antigravity | OpenCode | Codex](https://img.shields.io/badge/Harnesses-Claude%20%7C%20Agy%20%7C%20OpenCode%20%7C%20Codex-orange.svg)](https://github.com/)
[![Agensi: Verified Skill](https://img.shields.io/badge/Agensi-Verified%20Skill-brightgreen.svg)](https://agensi.io)

**Agent Coordinator** is a lightweight, deterministic multi-agent harness that connects **Claude Code**, Google **Antigravity (`agy`)**, **OpenCode (`opencode`)**, and **OpenAI Codex (`codex`)**.

It enables **parallel multi-agent development and background task offloading across any combination of agents with zero token waste**, strictly protecting active working directories from interference.

---

## 📖 Documentation Index

* 🚀 [Getting Started & Installation](docs/getting-started.md) — How to set up the CLI and connect via MCP (Claude Code, Antigravity, OpenCode, Cursor, Windsurf).
* 🏛 [Architecture & Token Economy](docs/architecture.md) — The Watcher-Implementer model, Git worktrees, and how deterministic orchestration saves 85–95% of inspection tokens.
* 💡 [Workflows & Prompting Guide](docs/workflows.md) — Real-world prompting examples for TDD, multi-agent parallel execution, GPU gating, and automated review guidance loops.
* 🔌 [MCP Tool Reference](docs/mcp-tools.md) — API specification for all 10 native `coord_*` MCP tools.
* 🛠 [CLI Reference](docs/cli-reference.md) — Complete cheat sheet for terminal commands and options.
* ❓ [Frequently Asked Questions (FAQ)](docs/faq.md) — Common questions regarding timeouts, non-git directories, GPU resource handling, and skill composition.
* 📋 [TASK_REPORT Schema](schemas/task-report-schema.json) — Standard JSON Schema for the worker completion report.

---

## 🏛 Core Philosophy: Dynamic Master-Worker Model

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
                       Compact TASK_REPORT.json
                                   │
                                   ▼
                   Diff Review & Clean Merge by Master
```

1. **Dynamic Master-Worker Roles:**
   * Any harness can be the **Master** (Architect/Reviewer) or **Worker** (Implementer/Reporter). Claude Code is the primary default, with instant offloading to Antigravity (for 1M+ context/GPU), OpenCode (for free/local models), or Codex (for OpenAI code synthesis).
2. **Working-Tree Protection:**
   * Autonomous worker agents execute in dedicated Git worktrees (`.coordinator/worktrees/<task-id>`) on isolated branches (`agent/<agent>/<task-id>`). Your active workspace, unstaged files, and Git index are **100% untouched**.
3. **Zero Token Waste:**
   * Worktree lifecycle management, process monitoring, GPU polling, and branch merging are handled deterministically via Python (**0 LLM tokens**).
   * Workers return a compact ~100-token `TASK_REPORT.json` rather than transferring large multi-turn conversation logs.

---

## ⚡ Quick Example: Parallel Task Dispatch

```bash
# Run multiple tasks concurrently across different agents:
coord parallel \
  --task "Build backend API:agy:server/api.py:pytest tests/test_api.py" \
  --task "Scaffold React Dashboard:opencode:ui/Dashboard.tsx:npm run build" \
  --task "Author integration tests:claude:tests/test_e2e.py:pytest tests/test_e2e.py" \
  --auto-merge
```

---

## 🔗 Community & Support

* **Marketplace Listing:** Available as an agent skill on [Agensi.io](https://agensi.io).
* **Issues & Questions:** Please open an issue in this repository for questions, feedback, or bug reports.
* **License:** [MIT License](LICENSE).
