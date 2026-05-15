# Project Instructions

## Overview

Multi-agent AI system for git lifecycle management via MCP, with Human-In-The-Loop controls and continuous learning from feedback.

## Architecture

```
├── .claude/agents/      # Chat agents (.agent.md — markdown-based AI personas)
├── .claude/skills/      # Skills (on-demand workflows for chat agents)
├── .claude/             # Support files (git-lessons-learned.md, git-report-template.md)
├── mcp/                 # MCP server implementations
├── tools/               # CLI tools and scripts
├── config/              # Environment configs (.env for tokens)
└── docs/                # Documentation
```

## Conventions

- Chat agents: one `.agent.md` per agent in `.claude/agents/`
- Skills: one folder per skill in `.claude/skills/<name>/SKILL.md`
- MCP servers expose tools via the Model Context Protocol standard
- All tokens and secrets go in `config/.env` (never committed)
- Lessons accumulate in `.claude/git-lessons-learned.md`

## Chat Agents

| Agent | Purpose |
|-------|----------|
| `@git-orchestrator` | Routes all git operations to specialist agents |
| `@git-repo-manager` | Repository creation, cloning, and releases (GitHub/GitLab) |
| `@git-sync-manager` | Pull, push, fetch with safety checks |
| `@git-merge-manager` | PRs, MRs, merge operations, conflict analysis |
| `@git-reviewer` | HITL review, reporting, feedback learning, doc sync |

## Skills

| Skill | Purpose |
|-------|---------|
| `/git-repo-create` | Create repositories on GitHub or GitLab |
| `/git-clone` | Clone repositories locally |
| `/git-pull` | Pull changes with safety checks |
| `/git-push` | Push changes with branch protection |
| `/git-pull-request` | Create GitHub pull requests |
| `/git-merge-request` | Create GitLab merge requests |
| `/git-conflict-analysis` | Analyze and classify merge conflicts |
| `/git-branch-protection` | Verify main branch integrity |
| `/git-action-report` | Generate structured action reports |
| `/git-feedback-learning` | Process feedback into lasting rules |
| `/git-release` | Create releases on GitHub or GitLab with version tags |
| `/git-sync-docs` | Regenerate documentation from current agents and skills |

## Core Rules

1. **Main branch is sacred** — always functional, feature branches adapt to main
2. **Never auto-approve** — all merge decisions require explicit human confirmation
3. **Always report** — every action produces a structured report
4. **Always learn** — feedback is captured and applied to future operations

## Agent Execution Rules

- All agents MUST read `.claude/git-lessons-learned.md` before performing any action
- When delegating to a chat agent, you MUST read and follow the full behavioral instructions in its `.agent.md` body
- This includes: constraints, approach steps, skill-checking procedures, and required output formats
- The `.agent.md` body defines strict runtime behavior, not just a persona label

## Agent Guidelines

- When adding a chat agent, create `.claude/agents/<name>.agent.md` with YAML frontmatter
- Use `tools: [agent]` and `agents: [name1, name2]` in frontmatter for multi-agent delegation
- MCP tools should have clear descriptions for agent discovery
- Prefer composing existing skills over creating new ones
- Test tool access independently before integrating with agents
