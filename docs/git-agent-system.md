# Git Multi-Agent System Documentation

**Author: Chong Kiat Lim**

## Overview

This document describes the architecture, flow, and relationships of the git multi-agent management system. The system uses 5 markdown-based AI agents and 10 skills to manage the full git lifecycle with Human-In-The-Loop (HITL) controls and continuous learning.

## System Flow Diagram

```mermaid
flowchart TD
    USER(["🧑‍💻 USER REQUEST"])
    USER --> ORCH

    subgraph ORCH_BOX ["@git-orchestrator"]
        ORCH["1. Reads git-lessons-learned.md\n2. Analyzes request type\n3. Routes to specialist agent\n4. Ensures report is generated"]
    end

    ORCH --> REPO
    ORCH --> SYNC
    ORCH --> MERGE

    subgraph REPO_BOX ["@git-repo-manager"]
        REPO["Skills:\n/git-repo-create\n/git-clone"]
    end

    subgraph SYNC_BOX ["@git-sync-manager"]
        SYNC["Skills:\n/git-pull\n/git-push\n/git-branch-protection"]
    end

    subgraph MERGE_BOX ["@git-merge-manager"]
        MERGE["Skills:\n/git-pull-request\n/git-merge-request\n/git-conflict-analysis\n/git-branch-protection"]
    end

    REPO --> REVIEWER
    SYNC --> REVIEWER
    MERGE --> REVIEWER

    subgraph REVIEW_BOX ["@git-reviewer"]
        REVIEWER["Skills: /git-action-report, /git-feedback-learning\n\n1. Generates structured report for every action\n2. Presents HITL decisions to human\n3. Processes feedback into rules"]
    end

    REVIEWER --> REPORT["📄 REPORT OUTPUT\nShown to user"]
    REVIEWER --> DECISION["🧑‍⚖️ HUMAN DECISION\nAPPROVE / REJECT / REVISE"]
    REVIEWER --> FEEDBACK["🧠 FEEDBACK LEARNING\nWrites to lessons-learned.md\n& Claude memory"]

    DECISION --> LESSONS[("📚 git-lessons-learned.md\n\nRead by ALL agents\nbefore every operation\n\nRules accumulate over time\nfrom human feedback")]
    FEEDBACK --> LESSONS
    LESSONS -.->|"feeds back into"| ORCH

    style USER fill:#4A90D9,stroke:#2C5F8A,color:#fff
    style ORCH_BOX fill:#2D2D2D,stroke:#4A90D9,color:#fff
    style REPO_BOX fill:#2D2D2D,stroke:#27AE60,color:#fff
    style SYNC_BOX fill:#2D2D2D,stroke:#27AE60,color:#fff
    style MERGE_BOX fill:#2D2D2D,stroke:#27AE60,color:#fff
    style REVIEW_BOX fill:#2D2D2D,stroke:#E67E22,color:#fff
    style REPORT fill:#1A1A2E,stroke:#16A085,color:#fff
    style DECISION fill:#1A1A2E,stroke:#E74C3C,color:#fff
    style FEEDBACK fill:#1A1A2E,stroke:#9B59B6,color:#fff
    style LESSONS fill:#1A1A2E,stroke:#F39C12,color:#fff
```

## Agent Relationships

```mermaid
flowchart LR
    subgraph DELEGATION ["Agent Delegation Map"]
        direction TB
        ORCH["@git-orchestrator"] --> REPO_MGR["@git-repo-manager"]
        ORCH --> SYNC_MGR["@git-sync-manager"]
        ORCH --> MERGE_MGR["@git-merge-manager"]
        ORCH --> REVIEWER["@git-reviewer"]
        MERGE_MGR -.-> REVIEWER
    end

    style ORCH fill:#4A90D9,stroke:#2C5F8A,color:#fff
    style REPO_MGR fill:#27AE60,stroke:#1E8449,color:#fff
    style SYNC_MGR fill:#27AE60,stroke:#1E8449,color:#fff
    style MERGE_MGR fill:#27AE60,stroke:#1E8449,color:#fff
    style REVIEWER fill:#E67E22,stroke:#CA6F1E,color:#fff
```

```mermaid
flowchart LR
    subgraph SKILLS ["Skill Ownership Map"]
        direction TB

        subgraph REPO_S ["@git-repo-manager"]
            S1["/git-repo-create"]
            S2["/git-clone"]
            S3["/git-release"]
        end

        subgraph SYNC_S ["@git-sync-manager"]
            S4["/git-pull"]
            S5["/git-push"]
            S6["/git-branch-protection"]
        end

        subgraph MERGE_S ["@git-merge-manager"]
            S7["/git-pull-request"]
            S8["/git-merge-request"]
            S9["/git-conflict-analysis"]
            S10["/git-branch-protection \n(shared)"]
        end

        subgraph REVIEW_S ["@git-reviewer"]
            S11["/git-action-report"]
            S12["/git-feedback-learning"]
            S13["/git-sync-docs"]
        end
    end

    style REPO_S fill:#2D2D2D,stroke:#27AE60,color:#fff
    style SYNC_S fill:#2D2D2D,stroke:#27AE60,color:#fff
    style MERGE_S fill:#2D2D2D,stroke:#27AE60,color:#fff
    style REVIEW_S fill:#2D2D2D,stroke:#E67E22,color:#fff
```

## Operation Flows

### Flow 1: Repository Creation

```mermaid
flowchart TD
    A(["🧑‍💻 User: Create a repo"]) --> B

    subgraph B_BOX ["@git-orchestrator"]
        B["Routes to @git-repo-manager"]
    end

    B --> C

    subgraph C_BOX ["@git-repo-manager"]
        C["/git-repo-create\n→ Asks: name, platform, visibility\n→ Creates repo via MCP or API"]
    end

    C --> D

    subgraph D_BOX ["@git-reviewer"]
        D["/git-action-report\n→ Generates report: repo URL, settings, status"]
    end

    D --> E["📄 Report shown to user"]

    style A fill:#4A90D9,stroke:#2C5F8A,color:#fff
    style B_BOX fill:#2D2D2D,stroke:#4A90D9,color:#fff
    style C_BOX fill:#2D2D2D,stroke:#27AE60,color:#fff
    style D_BOX fill:#2D2D2D,stroke:#E67E22,color:#fff
    style E fill:#1A1A2E,stroke:#16A085,color:#fff
```

### Flow 2: Pull with Conflict Detection

```mermaid
flowchart TD
    A(["🧑‍💻 User: Pull latest from main"]) --> B

    subgraph B_BOX ["@git-orchestrator"]
        B["Routes to @git-sync-manager"]
    end

    B --> C

    subgraph C_BOX ["@git-sync-manager"]
        C["/git-pull\n→ Checks for uncommitted changes\n→ Stash if needed\n→ Fetches remote"]
    end

    C --> D{"Diverged history?"}
    D -->|"✅ NO"| E["Pulls cleanly"]
    D -->|"⚠️ YES"| F

    subgraph F_BOX ["Conflict Resolution"]
        F["/git-conflict-analysis"]
        F --> G["@git-reviewer presents\nconflicts to human"]
        G --> H{"🧑‍⚖️ Human decides\nhow to resolve"}
    end

    E --> I
    H --> I

    subgraph I_BOX ["@git-reviewer"]
        I["/git-action-report"]
    end

    I --> J["📄 Report shown to user"]

    style A fill:#4A90D9,stroke:#2C5F8A,color:#fff
    style B_BOX fill:#2D2D2D,stroke:#4A90D9,color:#fff
    style C_BOX fill:#2D2D2D,stroke:#27AE60,color:#fff
    style D fill:#1A1A2E,stroke:#F39C12,color:#fff
    style E fill:#1A1A2E,stroke:#27AE60,color:#fff
    style F_BOX fill:#2D2D2D,stroke:#E74C3C,color:#fff
    style H fill:#1A1A2E,stroke:#E74C3C,color:#fff
    style I_BOX fill:#2D2D2D,stroke:#E67E22,color:#fff
    style J fill:#1A1A2E,stroke:#16A085,color:#fff
```

### Flow 3: Pull Request with HITL

```mermaid
flowchart TD
    A(["🧑‍💻 User: Create PR from feature to main"]) --> B

    subgraph B_BOX ["@git-orchestrator"]
        B["Routes to @git-merge-manager"]
    end

    B --> C

    subgraph C_BOX ["@git-merge-manager"]
        C1["/git-branch-protection\nChecks main is clean & up-to-date"]
        C1 --> C2["/git-conflict-analysis\nDetects & classifies conflicts:\ntrivial / logic / structural / dependency"]
        C2 --> C3["/git-pull-request\nCreates PR via GitHub MCP\nDoes NOT auto-merge"]
        C3 --> C4["Compiles full analysis"]
    end

    C4 --> D

    subgraph D_BOX ["@git-reviewer"]
        D["/git-action-report\nPR creation report"]
        D --> E["Presents HITL Decision:\n\nChanges: summary\nConflicts: list or None\nRisk: Low/Medium/High\nRecommendation: action"]
    end

    E --> F{"🧑‍⚖️ HUMAN DECISION"}
    F -->|"✅ APPROVE"| G["Merge proceeds"]
    F -->|"❌ REJECT"| H
    F -->|"🔄 REVISE"| I["Agent adapts &\nre-submits"]

    subgraph H_BOX ["/git-feedback-learning"]
        H["Parse feedback\n→ Write rule\n→ Save to memory\n→ Apply now"]
    end

    I --> C1

    style A fill:#4A90D9,stroke:#2C5F8A,color:#fff
    style B_BOX fill:#2D2D2D,stroke:#4A90D9,color:#fff
    style C_BOX fill:#2D2D2D,stroke:#27AE60,color:#fff
    style D_BOX fill:#2D2D2D,stroke:#E67E22,color:#fff
    style F fill:#1A1A2E,stroke:#E74C3C,color:#fff
    style G fill:#1A1A2E,stroke:#27AE60,color:#fff
    style H_BOX fill:#2D2D2D,stroke:#9B59B6,color:#fff
    style I fill:#1A1A2E,stroke:#F39C12,color:#fff
```

### Flow 4: Feedback Learning Cycle

```mermaid
flowchart TD
    A(["❌ Human rejects or comments"]) --> B

    subgraph B_BOX ["@git-reviewer → /git-feedback-learning"]
        B["Step 1: PARSE\nWhat was wrong? What should change?"] --> C
        C["Step 2: FORMULATE\nWHEN: trigger condition\nDO/DO NOT: required behavior\nBECAUSE: reason from feedback"] --> D
        D["Step 3: STORE shared\nAppend rule to\n.claude/git-lessons-learned.md"] --> E
        E["Step 4: STORE personal\nSave to Claude memory system"] --> F
        F["Step 5: APPLY\nFix current task using the new rule"] --> G
        G["Step 6: REPORT\nWhat was learned, rule created, impact"]
    end

    G --> H

    subgraph H_BOX ["🔁 Future Operations"]
        H["ALL agents read lessons-learned\nBEFORE every action\n→ Rule is applied automatically"]
    end

    style A fill:#E74C3C,stroke:#C0392B,color:#fff
    style B_BOX fill:#2D2D2D,stroke:#9B59B6,color:#fff
    style H_BOX fill:#2D2D2D,stroke:#F39C12,color:#fff
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
