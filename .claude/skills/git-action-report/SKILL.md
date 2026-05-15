---
name: git-action-report
description: 'Generate a structured report for any git action performed. Use after every git operation to document what was done.'
argument-hint: 'Action type and details, e.g. "pull origin/main" or "push feature-branch"'
---

# Git Action Report

## When to Use
- After ANY git action is completed (pull, push, clone, PR, MR, merge, etc.)
- When user asks "what happened?" or "show me what was done"
- Called by other agents/skills to document their operations

## Procedure

1. Gather the action context:
   - What action was performed
   - When it was performed (timestamp)
   - Who requested it
   - Which branch/repo was involved

2. Capture before and after state:
   ```bash
   git status
   git log --oneline -5
   git branch -vv
   ```

3. Format using the report template below

4. Present the report to the user

## Report Template

```markdown
# Git Action Report

| Field | Value |
|-------|-------|
| **Timestamp** | [YYYY-MM-DD HH:MM:SS] |
| **Action** | [action type: pull/push/clone/PR/MR/merge/create-repo] |
| **Repository** | [repo name or URL] |
| **Branch** | [branch name] |
| **Remote** | [remote name and URL] |

## Before State
- **Branch**: [branch before action]
- **HEAD**: [commit hash before]
- **Working Tree**: [clean/dirty — uncommitted changes?]
- **Upstream Status**: [ahead/behind/up-to-date]

## After State
- **Branch**: [branch after action]
- **HEAD**: [commit hash after]
- **Working Tree**: [clean/dirty]
- **Upstream Status**: [ahead/behind/up-to-date]

## Changes Summary
- **Commits**: [number of commits involved]
- **Files Changed**: [number]
- **Insertions**: [+lines]
- **Deletions**: [-lines]
- **Key Files**: [list of important files that changed]

## Warnings
[Any concerns, risks, or follow-up actions needed — or "None"]

## Lessons Applied
[List any rules from git-lessons-learned.md that were applied during this action — or "None"]
```

## Additional Notes
- Keep reports concise but complete
- Highlight anything unusual or unexpected
- If the action failed, report the failure clearly with error details
- If warnings exist, make them prominent
