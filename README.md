# Agentihub

Private agent identities for the agenti* ecosystem.

## Three-Layer Architecture

```
agenticore   = execution engine (K8s pod — runs Claude Code, manages jobs)
agentihooks  = hook system + MCP tools (open-source — guardrails, integrations)
agentihub    = agent identities (private — CLAUDE.md, prompts, evaluation)
```

Agenticore provisions agents **directly** from this repo. On startup, agenticore's `agent_mode/initializer.py` clones agentihub, finds the requested agent, and copies its `package/` directory to `/app/package/`.

## Directory Structure

```
agents/
└── <name>/
    ├── agent.yml              # Agent metadata (model, turns, timeout, MCP categories)
    ├── package/               # Production config — copied to /app/package/
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
AGENTIHUB_URL + AGENTIHUB_AGENT env vars
          │
          ▼
    agenticore agent_mode/initializer.py
    git clone --depth 1 $AGENTIHUB_URL → /tmp/agentihub-clone/
          │
          ▼
    Copy agents/<name>/package/ → /app/package/
    Copy agents/<name>/evaluation/ → /app/evaluation/
          │
          ▼
    Claude Code runs with /app/package/ as config root
```

1. Agenticore reads `AGENTIHUB_URL` and `AGENTIHUB_AGENT` env vars
2. Shallow-clones this repo to `/tmp/agentihub-clone/`
3. Copies `agents/{name}/package/` → `/app/package/` and `evaluation/` → `/app/evaluation/`
4. Claude Code loads CLAUDE.md, settings, prompts, and MCP config from `/app/package/`

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

5. Set `AGENTIHUB_URL` and `AGENTIHUB_AGENT=myagent` in the agenticore pod environment.

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
