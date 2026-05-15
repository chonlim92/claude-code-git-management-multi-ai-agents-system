---
name: git-conflict-analysis
description: 'Analyze merge conflicts between two branches. Use when conflicts are detected, before merging, or when the user asks to check if branches are compatible.'
argument-hint: 'Source and target branches, e.g. "feature-branch main"'
---

# Conflict Analysis

## When to Use
- Conflicts detected during pull or merge
- Before creating a PR/MR to pre-check compatibility
- User asks "are there conflicts?", "can I merge?", "check compatibility"
- Called by other skills/agents as part of merge workflow

## Pre-Checks
1. Read `.claude/git-lessons-learned.md` for applicable rules
2. Identify the two branches to compare:
   - Source branch (feature branch being merged IN)
   - Target branch (destination, usually main)

## Procedure

1. Fetch latest state:
   ```bash
   git fetch origin
   ```

2. Find the merge base (common ancestor):
   ```bash
   git merge-base origin/<target> origin/<source>
   ```

3. Attempt a trial merge (without committing):
   ```bash
   git checkout -b temp-conflict-check origin/<target>
   git merge --no-commit --no-ff origin/<source>
   ```

4. Check for conflicts:
   ```bash
   git diff --name-only --diff-filter=U
   ```

5. For each conflicted file, extract conflict details:
   ```bash
   git diff --check
   cat <conflicted-file>  # to see conflict markers
   ```

6. Abort the trial merge:
   ```bash
   git merge --abort
   git checkout -
   git branch -D temp-conflict-check
   ```

7. Also check for non-conflict risks:
   ```bash
   # Files modified in both branches (potential semantic conflicts)
   comm -12 <(git diff --name-only origin/<base>..origin/<target> | sort) <(git diff --name-only origin/<base>..origin/<source> | sort)
   ```

## Conflict Classification

For each conflict, classify as:

### Trivial (Auto-resolvable suggestion)
- Whitespace-only differences
- Import ordering differences
- Non-overlapping changes in same file (different sections)
- Suggestion: Auto-resolve favoring main's patterns

### Logic (Requires human decision)
- Same function/method modified differently in both branches
- Same variable/config changed to different values
- Overlapping line changes with different intent
- Suggestion: Present both sides, recommend, but REQUIRE human decision

### Structural (Always escalate)
- File renamed on one branch, modified on other
- File deleted on one branch, modified on other
- Directory structure conflicts
- Suggestion: Always escalate to human with full context

### Dependency (Recommend main's version)
- Package version conflicts (package.json, requirements.txt, etc.)
- Lock file conflicts
- Suggestion: Use main's version unless feature has critical security fix

## Golden Rule Application

> Feature branch MUST adapt to main. Never the other way around.

For every conflict, the resolution strategy must:
- Keep main's behavior intact
- Modify the feature branch to work WITH main's current state
- If feature cannot work with main's state, flag as "Requires feature branch rework"

## Report Output

```
## Conflict Analysis Report

### Branches
- **Source (merging in)**: [source-branch]
- **Target (destination)**: [target-branch]
- **Merge Base**: [commit hash]

### Summary
- Total conflicted files: [N]
- Trivial conflicts: [N] (auto-resolvable)
- Logic conflicts: [N] (requires human decision)
- Structural conflicts: [N] (escalated)
- Dependency conflicts: [N]

### Detailed Conflicts

#### [File path] — [Classification]
- **What**: [description of the conflict]
- **Main's version**: [what main has]
- **Feature's version**: [what feature branch has]
- **Suggested resolution**: [recommendation]
- **Confidence**: High/Medium/Low

[Repeat for each conflict]

### Risk Assessment
- **Overall Risk**: Low/Medium/High
- **Main Branch Impact**: [will main break? what's at risk?]
- **Recommendation**: Proceed with suggested resolutions / Rework feature branch / Reject merge

### Required Actions
[List what the feature branch must do before it can be merged]
```
