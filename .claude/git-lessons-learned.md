# Git Lessons Learned

This file contains accumulated rules and lessons from human feedback on git operations. All git agents MUST read this file before performing any action and apply all relevant rules.

---

## Rules

<!-- New rules are appended below. Do not remove existing rules unless they are explicitly superseded. -->

### RULE 1: Main Branch is Sacred
- **WHEN**: Any operation that could modify the main/master branch
- **DO**: Always protect main. Feature branches adapt to main, never the reverse.
- **BECAUSE**: Main must always remain functional and deployable.
- **SEVERITY**: Critical
- **ADDED**: 2026-05-15
- **CATEGORY**: [SAFETY]

### RULE 2: Never Auto-Approve
- **WHEN**: Any merge, PR, or MR requires approval
- **DO NOT**: Automatically approve or merge without explicit human confirmation
- **BECAUSE**: Human-in-the-loop is mandatory for all merge decisions. The agent provides analysis and recommendations, but the human makes the final call.
- **SEVERITY**: Critical
- **ADDED**: 2026-05-15
- **CATEGORY**: [PROCESS]

### RULE 3: Always Report
- **WHEN**: Any git action is completed (success or failure)
- **DO**: Generate a structured report documenting what was done, the before/after state, and any warnings
- **BECAUSE**: Transparency and auditability are required for all operations.
- **SEVERITY**: Important
- **ADDED**: 2026-05-15
- **CATEGORY**: [PROCESS]

---

## How to Add New Rules

When human feedback is received, the `/git-feedback-learning` skill will:
1. Parse the feedback into a structured rule
2. Assign the next rule number
3. Append the rule in the format above
4. Categorize with one of: [CODE-QUALITY], [PROCESS], [SAFETY], [PREFERENCE], [CONFLICT], [REVIEW], [BRANCH]
