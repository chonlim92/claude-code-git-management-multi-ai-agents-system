# Git Multi-Agent System Guidelines

## Overview

Multi-agent AI system for git lifecycle management via MCP, with Human-In-The-Loop controls, conflict management, and continuous learning from feedback.

## Architecture

```
.claude/agents/      → Chat agents (.agent.md — markdown-based AI personas)
.claude/skills/      → Skills (on-demand workflows invoked by agents)
.claude/             → Support files (lessons-learned, report template)
mcp/                 → MCP server implementations (Model Context Protocol)
tools/               → CLI tools and scripts
config/              → Environment configs (.env for tokens)
docs/                → Documentation (installation, MCP setup)
```

## Agents

| Agent | Role | Delegates To |
|-------|------|--------------|
| `@git-orchestrator` | Routes all git requests to specialists | All other agents |
| `@git-repo-manager` | Repository creation, cloning, and releases | — |
| `@git-sync-manager` | Pull, push, fetch with safety checks | — |
| `@git-merge-manager` | PRs, MRs, merges, conflict analysis | @git-reviewer |
| `@git-reviewer` | HITL review, reporting, feedback processing, doc sync | — |

## Skills

| Skill | Used By | Purpose |
|-------|---------|---------|
| `/git-repo-create` | @git-repo-manager | Create repos on GitHub/GitLab |
| `/git-clone` | @git-repo-manager | Clone repos locally |
| `/git-pull` | @git-sync-manager | Pull with safety checks |
| `/git-push` | @git-sync-manager | Push with branch protection |
| `/git-pull-request` | @git-merge-manager | Create GitHub PRs |
| `/git-merge-request` | @git-merge-manager | Create GitLab MRs |
| `/git-conflict-analysis` | @git-merge-manager | Detect and classify conflicts |
| `/git-branch-protection` | @git-sync-manager, @git-merge-manager | Verify main branch safety |
| `/git-action-report` | @git-reviewer | Generate structured reports |
| `/git-feedback-learning` | @git-reviewer | Process feedback into rules |
| `/git-release` | @git-repo-manager | Create releases with version tags |
| `/git-sync-docs` | @git-reviewer | Regenerate docs from current agents/skills |

## Core Rules

1. **Main branch is sacred** — always functional, never modified to accommodate feature branches
2. **Never auto-approve** — all merge/PR/MR decisions require explicit human confirmation
3. **Always report** — every action produces a structured before/after report
4. **Always learn** — feedback is captured as rules and applied to future operations
5. **Feature adapts to main** — conflicts are resolved in the feature branch, never in main

## Agent Execution Rules

- All agents MUST read `.claude/git-lessons-learned.md` before performing any action
- All agents MUST apply relevant rules from lessons-learned to the current operation
- All agents MUST produce a structured report after completing any action
- The `.agent.md` body defines strict runtime behavior — not just routing descriptions
- When delegating to an agent, follow its full behavioral instructions (constraints, approach, skill-checking)

## Conventions

- Chat agents: one `.agent.md` per agent in `.claude/agents/`
- Skills: one folder per skill in `.claude/skills/<name>/SKILL.md`
- MCP servers expose tools via the Model Context Protocol standard
- All tokens and secrets go in `config/.env` (never committed)
- Lessons accumulate in `.claude/git-lessons-learned.md` (committed, shared with team)

## Adding New Components

| Component | Steps |
|-----------|-------|
| **Chat Agent** | Create `.claude/agents/<name>.agent.md` with YAML frontmatter |
| **Skill** | Create `.claude/skills/<name>/SKILL.md` with frontmatter |
| **MCP Server** | Implement in `mcp/`, register via `claude mcp add` |
| **CLI Tool** | Create script in `tools/`, make it independently testable |
| **Lesson** | Via `/git-feedback-learning` skill or manually append to `git-lessons-learned.md` |
