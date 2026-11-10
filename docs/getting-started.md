# Getting Started & Setup

This guide walks you through installing Agent Coordinator and registering it as a Model Context Protocol (MCP) server across different AI agent environments.

---

## 1. Prerequisites

* **Operating System:** Linux or macOS
* **Python:** Python 3.9 or newer (`python3 --version`)
* **Git:** Version 2.20+ (optional; non-git directories are supported via snapshot workspaces)
* **Agent CLIs (at least one):**
  * [Claude Code](https://claude.ai/code) (`claude`)
  * Google Antigravity CLI (`agy`)
  * [OpenCode](https://opencode.ai) (`opencode`)

---

## 2. Installation & Setup

### Acquisition via Agensi Marketplace
Agent Coordinator is distributed as a verified skill package on **[Agensi.io](https://agensi.io)**.

1. Download the verified `agent-coordinator.zip` skill package from your Agensi dashboard.
2. Unzip into your AI agent's skill directory (e.g. `~/.claude/skills/` or `~/.gemini/antigravity-cli/skills/`):
   ```bash
   unzip agent-coordinator.zip -d ~/.claude/skills/
   ```
3. Run the automated local installer script inside the extracted skill folder:
   ```bash
   bash ~/.claude/skills/agent-coordinator/scripts/install-coordinator.sh
   ```
   This automatically creates and links `coord`, `agent-coordinator`, and `coord-mcp` into your `~/.local/bin/`.

4. Verify that the CLI is accessible:
   ```bash
   coord status
   ```

---

## 3. Registering the MCP Server

The coordinator includes a built-in JSON-RPC Model Context Protocol (MCP) server that exposes 9 coordination tools to your AI agent.

### A. Claude Code Setup
Add the `coordinator` configuration to `~/.claude.json`:

```json
{
  "mcpServers": {
    "coordinator": {
      "command": "coord-mcp",
      "args": []
    }
  }
}
```

*Note: If `coord-mcp` is not in your global `PATH`, specify the explicit python command:*
```json
{
  "mcpServers": {
    "coordinator": {
      "command": "python3",
      "args": ["-m", "coordinator.mcp_server"]
    }
  }
}
```

---

### B. Google Antigravity Setup
Add the coordinator to your Antigravity configuration at `~/.gemini/config/mcp_config.json`:

```json
{
  "mcpServers": {
    "coordinator": {
      "command": "coord-mcp",
      "args": []
    }
  }
}
```

---

### C. OpenCode Setup
Add the coordinator entry to `~/.config/opencode/opencode.jsonc`:

```jsonc
{
  "mcp": {
    "coordinator": {
      "type": "local",
      "command": [
        "coord-mcp"
      ],
      "enabled": true
    }
  }
}
```

---

### D. Cursor & Windsurf Setup
In your IDE settings:
1. Open **Features → MCP Servers**.
2. Click **Add New MCP Server**.
3. Set **Type** to `command`.
4. Set **Name** to `coordinator`.
5. Set **Command** to `coord-mcp`.

---

## 4. Verifying Connectivity

Once registered, ask your agent:
> *"Check coordinator status and GPU health."*

The agent will call `coord_status` and confirm that all harnesses and isolated workspaces are operational.
