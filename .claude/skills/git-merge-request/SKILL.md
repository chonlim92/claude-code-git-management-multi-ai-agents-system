---
name: git-merge-request
description: 'Create a merge request on GitLab. Use when the user wants to open an MR, submit changes for review on GitLab, or propose merging a branch.'
argument-hint: 'Source branch and target branch, e.g. "feature-branch main"'
---

# Create Merge Request (GitLab)

## When to Use
- User wants to create a merge request on GitLab
- User says "open MR", "create merge request", "submit for review on GitLab"
- User wants to propose merging a branch on a GitLab project

## Pre-Checks
1. Read `.claude/git-lessons-learned.md` for applicable rules
2. Confirm with user:
   - Source branch (the feature branch)
   - Target branch (default: main)
   - MR title
   - MR description (what and why)
   - GitLab project ID or path (e.g., `namespace/project`)
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

4. Create the merge request via GitLab API:
   ```bash
   curl --header "PRIVATE-TOKEN: $GITLAB_PAT" \
     --data "source_branch=<source>&target_branch=<target>&title=<title>&description=<description>" \
     "https://gitlab.com/api/v4/projects/<project-id>/merge_requests"
   ```

5. **DO NOT auto-merge** — present analysis for human review

## Human Review Package

Present to the human:

```
## Merge Request Created

### MR Details
- **URL**: [MR URL]
- **Title**: [title]
- **Source**: [source-branch] → **Target**: [target-branch]

### Changes Summary
[commits list and diff stats]

### Conflict Status
[output of conflict analysis — "Clean" or list of conflicts]

### Risk Assessment
[Low/Medium/High based on: number of files, lines changed, conflicts, critical files touched]

### Next Steps
This MR is now open for review. It will NOT be auto-merged.
Please review the changes and approve/reject when ready.
```

## Report Output
After completion, provide:
- **Action**: Merge request created
- **MR URL**: [link]
- **Source Branch**: [branch]
- **Target Branch**: [branch]
- **Commits**: [number]
- **Files Changed**: [number]
- **Conflicts**: None / [list]
- **Status**: Open — awaiting human review
