# Agentihub

Private agent identities for the agenti* ecosystem.

## Architecture

```
agenticore   = execution engine (runs Claude, manages jobs)
agentihooks  = build system (open-source — profiles, hooks, guardrails)
agentihub    = agent identities (private — CLAUDE.md, workflows, evaluation)
```

## Usage

Agent Hop connector in agentihooks builds these agents into deployable profiles:

```bash
cd /path/to/agentihooks
python scripts/agent_hop.py /path/to/agentihub
```

## Adding an Agent

```
agents/<name>/
├── agent.yml                  # Profile config (same schema as agentihooks profile.yml)
├── settings.overrides.json    # Env overrides
├── __init__.py                # Package marker
├── .claude/
│   └── CLAUDE.md              # Agent personality + workflow
└── evaluation/
    ├── eval.yml               # Criteria, weights, thresholds
    └── cases/                 # Test cases
```
