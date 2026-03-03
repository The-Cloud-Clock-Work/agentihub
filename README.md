# Agentihub

Private agent identities for the agenti* ecosystem.

## Three-Layer Architecture

```
agenticore   = execution engine (K8s pod — runs Claude Code, manages jobs)
agentihooks  = hook system + MCP tools (open-source — guardrails, integrations)
agentihub    = agent identities (private — CLAUDE.md, prompts, evaluation)
```

Agenticore provisions agents **directly** from this repo. Each agent is a self-contained package that gets synced to `/app/package/` inside the agenticore container via S3.

## Directory Structure

```
agents/
└── <name>/
    ├── agent.yml              # Agent metadata (model, turns, timeout, MCP categories)
    ├── package/               # Production config — synced to /app/package/
    │   ├── CLAUDE.md          # Agent system prompt
    │   ├── system.md          # Extended system instructions
    │   ├── .claude/
    │   │   └── settings.json  # Claude Code settings (hooks, permissions)
    │   ├── .mcp.json          # MCP server config for the agent
    │   ├── prompts/           # Command prompts (optional)
    │   │   ├── commands.json
    │   │   └── default.md
    │   └── runners/           # Startup scripts (optional, sorted by prefix)
    │       └── 001_setup.sh
    └── evaluation/            # Eval harness
        ├── eval.yml
        └── cases/
```

## Provisioning Flow

```
agentihub/agents/<name>/package/
          │
          ▼
    Upload to S3 (STORAGE_URL)
          │
          ▼
    agenticore initializer.py
    syncs S3 → /app/package/
          │
          ▼
    Claude Code runs with
    /app/package/ as config root
```

1. Agent packages are uploaded to S3 (per-agent bucket path)
2. Agenticore's `initializer.py` syncs `{STORAGE_URL}/package/` → `/app/package/` on startup
3. Claude Code loads CLAUDE.md, settings, prompts, and MCP config from `/app/package/`

In dev mode (`STORAGE_SYNC=false`), the package directory is mounted directly via volume.

## Adding a New Agent

1. Create the agent directory:
   ```bash
   mkdir -p agents/myagent/package/.claude
   mkdir -p agents/myagent/evaluation/cases
   ```

2. Write `agent.yml`:
   ```yaml
   name: myagent
   description: "What this agent does"

   claude:
     model: sonnet
     max_turns: 50
     permission_mode: bypassPermissions
     timeout: 1800

   mcp_categories: all
   ```

3. Write `package/CLAUDE.md` with the agent's system prompt and workflow instructions.

4. Copy or create `package/.claude/settings.json` with hook wiring and permissions.

5. Upload the package to S3 and configure agenticore to point at it via `STORAGE_URL`.

## Current Agents

| Agent | Description |
|-------|-------------|
| `publishing` | Content publishing — drafts, reviews, publishes to LinkedIn/Medium |

## Related Projects

| Project | Description |
|---------|-------------|
| [agenticore](https://github.com/The-Cloud-Clock-Work/agenticore) | Claude Code runner and orchestrator |
| [agentihooks](https://github.com/The-Cloud-Clock-Work/agentihooks) | Hook system and MCP tools |
| [agentibridge](https://github.com/The-Cloud-Clock-Work/agentibridge) | Session persistence and remote control |
