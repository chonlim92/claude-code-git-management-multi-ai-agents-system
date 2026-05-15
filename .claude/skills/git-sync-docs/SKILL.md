---
name: git-sync-docs
description: 'Regenerate project documentation (README.md, AGENTS.md, CLAUDE.md) from current agents and skills. Use after adding, modifying, or deleting agents or skills to keep docs in sync.'
---

# Sync Documentation

## When to Use
- After adding a new agent or skill
- After modifying an existing agent or skill
- After deleting an agent or skill
- When user says "sync docs", "update docs", "refresh documentation"
- Triggered automatically by the post-edit hook on `.claude/agents/` or `.claude/skills/` changes

## Procedure

### Step 1: Scan Current Agents

Read all `.agent.md` files in `.claude/agents/`:
```bash
ls .claude/agents/*.agent.md
```

For each agent, extract from YAML frontmatter and body:
- **Name**: filename without `.agent.md`
- **Description**: from `description:` in frontmatter
- **Purpose**: first sentence of the agent body (after "You are...")
- **Skills**: list of `/skill-name` references in the body
- **Delegates to**: from `agents:` in frontmatter

### Step 2: Scan Current Skills

Read all `SKILL.md` files in `.claude/skills/*/`:
```bash
ls .claude/skills/*/SKILL.md
```

For each skill, extract from YAML frontmatter:
- **Name**: from `name:` field
- **Description**: from `description:` field
- **Used by**: which agent(s) reference this skill

### Step 3: Update CLAUDE.md

Regenerate the following sections in `CLAUDE.md`, preserving all other content:

**Chat Agents table:**
```markdown
## Chat Agents

| Agent | Purpose |
|-------|----------|
| `@<name>` | <description> |
...
```

**Skills table:**
```markdown
## Skills

| Skill | Purpose |
|-------|---------|
| `/<name>` | <description> |
...
```

### Step 4: Update AGENTS.md

Regenerate the following sections in `AGENTS.md`, preserving all other content:

**Agents table:**
```markdown
## Agents

| Agent | Role | Delegates To |
|-------|------|--------------|
| `@<name>` | <description> | <delegates-to or "—"> |
...
```

**Skills table:**
```markdown
## Skills

| Skill | Used By | Purpose |
|-------|---------|---------|
| `/<name>` | @<agent> | <description> |
...
```

### Step 5: Update README.md

Regenerate the following sections in `README.md`, preserving all other content:

**Agents table:**
```markdown
## Agents

| Agent | Purpose |
|-------|---------|
| `@<name>` | <description> |
...
```

**Skills table:**
```markdown
## Skills

| Skill | Description |
|-------|-------------|
| `/<name>` | <description> |
...
```

**Documentation section** — add link if new doc exists:
```markdown
## Documentation

- [Git Agent System Architecture & Flows](docs/git-agent-system.md)
- ...
```

### Step 6: Update docs/git-agent-system.md

Regenerate the **Skill Ownership Map** and **Agent Relationships** sections to reflect current state.

### Step 7: Report

```markdown
## Documentation Sync Complete

### Changes Detected
- Agents: [N total] ([added/removed/modified])
- Skills: [N total] ([added/removed/modified])

### Files Updated
- [x] CLAUDE.md
- [x] AGENTS.md
- [x] README.md
- [x] docs/git-agent-system.md

### Current Inventory
**Agents**: [list]
**Skills**: [list]
```

## Important Rules
- NEVER remove manually-written sections (Key Principles, Configuration, Tech Stack, etc.)
- ONLY regenerate the agent/skill tables and relationship diagrams
- Preserve the "Author: Chong Kiat Lim" line in all files
- Preserve the Tech Stack section in README.md
- If a new skill is added to the Tech Stack category, add it to the Tech Stack table too
