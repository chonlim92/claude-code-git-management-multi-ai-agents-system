# Git Multi-Agent System Documentation

**Author: Chong Kiat Lim**

## Overview

This document describes the architecture, flow, and relationships of the git multi-agent management system. The system uses 5 markdown-based AI agents and 10 skills to manage the full git lifecycle with Human-In-The-Loop (HITL) controls and continuous learning.

## System Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              USER REQUEST                                        │
└─────────────────────────────────┬───────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          @git-orchestrator                                       │
│                                                                                 │
│  1. Reads git-lessons-learned.md                                                │
│  2. Analyzes request type                                                       │
│  3. Routes to specialist agent                                                  │
│  4. Ensures report is generated                                                 │
└───────┬──────────────────────┬──────────────────────┬───────────────────────────┘
        │                      │                      │
        ▼                      ▼                      ▼
┌───────────────┐  ┌───────────────────┐  ┌──────────────────────┐
│@git-repo-     │  │@git-sync-         │  │@git-merge-           │
│  manager      │  │  manager           │  │  manager             │
│               │  │                    │  │                      │
│ Skills:       │  │ Skills:            │  │ Skills:              │
│ /git-repo-    │  │ /git-pull          │  │ /git-pull-request    │
│   create      │  │ /git-push          │  │ /git-merge-request   │
│ /git-clone    │  │ /git-branch-       │  │ /git-conflict-       │
│               │  │   protection       │  │   analysis           │
│               │  │                    │  │ /git-branch-         │
│               │  │                    │  │   protection         │
└───────┬───────┘  └────────┬───────────┘  └──────────┬───────────┘
        │                   │                          │
        │                   │                          │
        └───────────────────┼──────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            @git-reviewer                                         │
│                                                                                 │
│  Skills: /git-action-report, /git-feedback-learning                             │
│                                                                                 │
│  1. Generates structured report for every action                                │
│  2. Presents HITL decisions to human (APPROVE / REJECT / REVISE)                │
│  3. Processes feedback into rules                                               │
└───────────────────────────────┬─────────────────────────────────────────────────┘
                                │
                ┌───────────────┼───────────────┐
                ▼               ▼               ▼
        ┌──────────┐    ┌─────────────┐   ┌──────────────┐
        │  REPORT  │    │   HUMAN     │   │  FEEDBACK    │
        │  OUTPUT  │    │  DECISION   │   │  LEARNING    │
        │          │    │             │   │              │
        │ Shown to │    │ APPROVE /   │   │ Writes to:   │
        │ user     │    │ REJECT /    │   │ • lessons-   │
        │          │    │ REVISE      │   │   learned.md │
        └──────────┘    └──────┬──────┘   │ • Claude     │
                               │          │   memory     │
                               │          └──────────────┘
                               ▼
                ┌──────────────────────────────┐
                │     git-lessons-learned.md   │
                │                              │
                │  Read by ALL agents before   │
                │  every operation             │
                │                              │
                │  Rules accumulate over time  │
                │  from human feedback         │
                └──────────────────────────────┘
```

## Agent Relationships

```
┌──────────────────────────────────────────────────────────────┐
│                    AGENT DELEGATION MAP                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  @git-orchestrator ──┬──► @git-repo-manager                  │
│                      ├──► @git-sync-manager                  │
│                      ├──► @git-merge-manager ──► @git-reviewer│
│                      └──► @git-reviewer                      │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│                    SKILL OWNERSHIP MAP                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  @git-repo-manager:                                          │
│    ├── /git-repo-create                                      │
│    ├── /git-clone                                            │
│    └── /git-release                                          │
│                                                              │
│  @git-sync-manager:                                          │
│    ├── /git-pull                                             │
│    ├── /git-push                                             │
│    └── /git-branch-protection                                │
│                                                              │
│  @git-merge-manager:                                         │
│    ├── /git-pull-request                                     │
│    ├── /git-merge-request                                    │
│    ├── /git-conflict-analysis                                │
│    └── /git-branch-protection (shared)                       │
│                                                              │
│  @git-reviewer:                                              │
│    ├── /git-action-report                                    │
│    ├── /git-feedback-learning                                │
│    └── /git-sync-docs                                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Operation Flows

### Flow 1: Repository Creation

```
User: "Create a repo"
  │
  ▼
@git-orchestrator
  │ routes to
  ▼
@git-repo-manager
  │ runs /git-repo-create
  │   → Asks: name, platform, visibility
  │   → Creates repo via MCP (GitHub) or API (GitLab)
  │
  ▼
@git-reviewer
  │ runs /git-action-report
  │   → Generates report: repo URL, settings, status
  │
  ▼
Report shown to user
```

### Flow 2: Pull with Conflict Detection

```
User: "Pull latest from main"
  │
  ▼
@git-orchestrator
  │ routes to
  ▼
@git-sync-manager
  │ runs /git-pull
  │   → Checks for uncommitted changes (stash if needed)
  │   → Fetches remote
  │   → Detects diverged history?
  │       ├── NO → pulls cleanly
  │       └── YES → STOPS
  │                   │
  │                   ▼
  │           runs /git-conflict-analysis
  │                   │
  │                   ▼
  │           @git-reviewer presents conflicts to human
  │                   │
  │                   ▼
  │           Human decides how to resolve
  │
  ▼
@git-reviewer
  │ runs /git-action-report
  ▼
Report shown to user
```

### Flow 3: Pull Request with HITL

```
User: "Create PR from feature to main"
  │
  ▼
@git-orchestrator
  │ routes to
  ▼
@git-merge-manager
  │
  ├── runs /git-branch-protection
  │     → Checks main is clean and up-to-date
  │     → Checks feature branch is based on current main
  │
  ├── runs /git-conflict-analysis
  │     → Detects conflicts between feature and main
  │     → Classifies: trivial / logic / structural / dependency
  │     → Suggests resolutions
  │
  ├── runs /git-pull-request
  │     → Creates PR via GitHub MCP
  │     → Does NOT auto-merge
  │
  └── Compiles full analysis
        │
        ▼
@git-reviewer
  │
  ├── runs /git-action-report (PR creation report)
  │
  └── Presents HITL decision:
        ┌─────────────────────────────────┐
        │   HUMAN DECISION REQUIRED       │
        │                                 │
        │   Changes: [summary]            │
        │   Conflicts: [list or "None"]   │
        │   Risk: [Low/Medium/High]       │
        │   Recommendation: [action]      │
        │                                 │
        │   → APPROVE                     │
        │   → REJECT (with comments)      │
        │   → REVISE (request changes)    │
        └────────────────┬────────────────┘
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
        APPROVE       REJECT       REVISE
            │            │            │
            │            ▼            │
            │   /git-feedback-        │
            │     learning            │
            │       │                 │
            │       ├── Parse feedback│
            │       ├── Write rule    │
            │       ├── Save memory   │
            │       └── Apply now     │
            │                         │
            ▼                         ▼
        Merge proceeds         Agent adapts and
                               re-submits
```

### Flow 4: Feedback Learning Cycle

```
Human rejects or comments
  │
  ▼
@git-reviewer runs /git-feedback-learning
  │
  ├── Step 1: PARSE
  │     What was wrong? What should change?
  │
  ├── Step 2: FORMULATE
  │     WHEN: [trigger condition]
  │     DO/DO NOT: [required behavior]
  │     BECAUSE: [reason from feedback]
  │
  ├── Step 3: STORE (shared)
  │     Append rule to .claude/git-lessons-learned.md
  │
  ├── Step 4: STORE (personal)
  │     Save to Claude memory system
  │
  ├── Step 5: APPLY
  │     Fix current task using the new rule
  │
  └── Step 6: REPORT
        What was learned, rule created, impact
          │
          ▼
    ┌─────────────────────────────────────┐
    │   Future operations:                │
    │   ALL agents read lessons-learned   │
    │   BEFORE every action               │
    │   → Rule is applied automatically   │
    └─────────────────────────────────────┘
```

## Shared Resources

| Resource | Path | Purpose |
|----------|------|---------|
| Lessons Learned | `.claude/git-lessons-learned.md` | Accumulated rules from feedback, read by all agents |
| Report Template | `.claude/git-report-template.md` | Structured template for action reports |
| Environment Config | `config/.env` | Tokens (GITHUB_PAT, GITLAB_PAT) |

## Design Decisions

### Why Markdown Agents (Not Code)?

- **Declarative**: Behavior defined as instructions, not imperative code
- **Transparent**: Anyone can read and understand agent behavior
- **Composable**: Skills are reusable across agents without imports
- **Evolvable**: Learning system adds rules without code changes

### Why Human-In-The-Loop?

- **Safety**: Prevents accidental merges that break main
- **Learning**: Every rejection teaches the system something new
- **Trust**: Human retains full control over irreversible actions
- **Audit**: Every decision is documented with reasoning

### Why Dual Storage for Learning?

- **Project file** (`.claude/git-lessons-learned.md`): Shared with team, version-controlled, survives context resets
- **Claude memory**: Personal retention across conversations, survives session boundaries
