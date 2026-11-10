# Agent Coordinator: Multi-Agent Parallel Orchestrator

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform: Linux | macOS](https://img.shields.io/badge/Platform-Linux%20%7C%20macOS-lightgrey.svg)](https://github.com/)
[![Harnesses: Claude Code | Antigravity | OpenCode](https://img.shields.io/badge/Harnesses-Claude%20%7C%20Agy%20%7C%20OpenCode-orange.svg)](https://github.com/)
[![Agensi: Verified Skill](https://img.shields.io/badge/Agensi-Verified%20Skill-brightgreen.svg)](https://agensi.io)

**Agent Coordinator** is a lightweight, deterministic multi-agent harness that connects **Claude Code**, Google **Antigravity (`agy`)**, and **OpenCode (`opencode`)**.

It enables **parallel multi-agent development and background task offloading across any combination of agents with zero token waste**, strictly protecting active working directories from interference.

---

## 📖 Documentation Index

* 🚀 [Getting Started & Installation](docs/getting-started.md) — How to set up the CLI and connect via MCP (Claude Code, Antigravity, OpenCode, Cursor, Windsurf).
* 🏛 [Architecture & Token Economy](docs/architecture.md) — The Watcher-Implementer model, Git worktrees, and how deterministic orchestration saves 85–95% of inspection tokens.
* 💡 [Workflows & Prompting Guide](docs/workflows.md) — Real-world prompting examples for TDD, multi-agent parallel execution, GPU gating, and automated review guidance loops.
* 🔌 [MCP Tool Reference](docs/mcp-tools.md) — API specification for all 9 native `coord_*` MCP tools.
* 🛠 [CLI Reference](docs/cli-reference.md) — Complete cheat sheet for terminal commands and options.
* ❓ [Frequently Asked Questions (FAQ)](docs/faq.md) — Common questions regarding timeouts, non-git directories, GPU resource handling, and skill composition.
* 📋 [TASK_REPORT Schema](schemas/task-report-schema.json) — Standard JSON Schema for the worker completion report.

---

## 🏛 Core Philosophy: The Watcher-Implementer Model

```
                    DEVELOPER / WATCHER (Claude Code)
                     Architect · Reviewer · Orchestrator
                                      │
                 ┌────────────────────┴────────────────────┐
                 ▼                                         ▼
         ANTIGRAVITY (`agy`)                      OPENCODE (`opencode`)
      (Gemini 3.8 Flash / Pro)                  (Free / Custom Models)
                 │                                         │
        Worktree 1 (Isolated)                     Worktree 2 (Isolated)
       `agent/agy/<task-id>`                    `agent/opencode/<task-id>`
                 │                                         │
                 └────────────────────┬────────────────────┘
                                      ▼
                          Compact TASK_REPORT.json
                                      │
                                      ▼
                      Diff Review & Clean Merge by Watcher
```

1. **Role Specialization:**
   * **Claude Code = Watcher, Architect, and Guide:** Maintains macro architectural context, reviews diffs, and guides background agents.
   * **Antigravity & OpenCode = Autonomous Implementers:** Handle heavy code generation, test suites, builds, and GPU workloads in isolated workspaces.
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
