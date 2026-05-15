---
name: git-pull-request
description: 'Create a pull request on GitHub. Use when the user wants to open a PR, submit changes for review, or propose merging a branch into another.'
argument-hint: 'Source branch and target branch, e.g. "feature-branch main"'
---

# Create Pull Request (GitHub)

## When to Use
- User wants to create a pull request on GitHub
- User says "open PR", "create pull request", "submit for review"
- User wants to propose merging a branch

## Pre-Checks
1. Read `.claude/git-lessons-learned.md` for applicable rules
2. Confirm with user:
   - Source branch (the feature branch)
   - Target branch (default: main)
   - PR title
   - PR description (what and why)
3. Run `/git-branch-protection` to verify target branch state
4. Run `/git-conflict-analysis` between source and target

## Procedure

1. Verify source branch is pushed to remote:
   ```bash
   git branch --show-current
   git log --oneline origin/<source-branch>..HEAD
   ```
   - If unpushed commits exist, push first (with user confirmation)

2. Check for conflicts with target:
   ```bash
   git fetch origin
   git merge-tree $(git merge-base origin/<target> origin/<source>) origin/<target> origin/<source>
   ```

3. Generate diff summary:
   ```bash
   git log --oneline origin/<target>..origin/<source>
   git diff --stat origin/<target>..origin/<source>
   ```

4. Create the pull request using GitHub MCP tool `create_pull_request`:
   - owner: `<repo-owner>`
   - repo: `<repo-name>`
   - title: `<PR title>`
   - body: `<PR description with conflict summary>`
   - head: `<source-branch>`
   - base: `<target-branch>`

5. **DO NOT auto-merge** — present analysis for human review

## Human Review Package

Present to the human:

```
## Pull Request Created

### PR Details
- **URL**: [PR URL]
- **Title**: [title]
- **Source**: [source-branch] → **Target**: [target-branch]

### Changes Summary
[commits list and diff stats]

### Conflict Status
[output of conflict analysis — "Clean" or list of conflicts]

### Risk Assessment
[Low/Medium/High based on: number of files, lines changed, conflicts, critical files touched]

### Next Steps
This PR is now open for review. It will NOT be auto-merged.
Please review the changes and approve/reject when ready.
```

## Report Output
After completion, provide:
- **Action**: Pull request created
- **PR URL**: [link]
- **Source Branch**: [branch]
- **Target Branch**: [branch]
- **Commits**: [number]
- **Files Changed**: [number]
- **Conflicts**: None / [list]
- **Status**: Open — awaiting human review
