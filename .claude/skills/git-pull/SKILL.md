---
name: git-pull
description: 'Pull changes from remote repository. Use when the user wants to pull, update, sync from remote, or get latest changes.'
argument-hint: 'Optional remote and branch, e.g. "origin main"'
---

# Pull Changes from Remote

## When to Use
- User wants to pull changes from remote
- User says "pull", "update", "get latest", "sync from remote"
- User wants to fetch and merge remote changes

## Pre-Checks
1. Read `.claude/git-lessons-learned.md` for applicable rules
2. Check current state:
   ```bash
   git status
   git branch --show-current
   git stash list
   ```
3. If uncommitted changes exist:
   - Warn the user
   - Offer to stash changes first:
     ```bash
     git stash push -m "Auto-stash before pull [timestamp]"
     ```

## Procedure

1. Record before-state:
   ```bash
   git log --oneline -1
   git rev-parse HEAD
   ```

2. Fetch first to check what's coming:
   ```bash
   git fetch <remote|origin>
   git log --oneline HEAD..<remote>/<branch> | head -20
   ```

3. If conflicts are likely (diverged history), STOP and report:
   - Show incoming changes
   - Show local changes that may conflict
   - Recommend running `/git-conflict-analysis` before proceeding
   - Ask user if they want to continue

4. Pull changes:
   ```bash
   git pull <remote|origin> <branch|current-branch>
   ```

5. If conflicts occur during pull:
   - DO NOT auto-resolve
   - List all conflicted files:
     ```bash
     git diff --name-only --diff-filter=U
     ```
   - Invoke `/git-conflict-analysis` for detailed analysis
   - Present to human for decision

6. Verify post-pull state:
   ```bash
   git status
   git log --oneline -5
   ```

7. If changes were stashed, remind user:
   - "You have stashed changes. Run `git stash pop` to restore them."

## Report Output
After completion, provide:
- **Action**: Pull completed
- **Remote/Branch**: [remote]/[branch]
- **Before**: [commit hash before pull]
- **After**: [commit hash after pull]
- **Commits Pulled**: [number and list]
- **Files Changed**: [number and list of key files]
- **Conflicts**: None / [list of conflicted files]
- **Stashed Changes**: Yes (reminder to pop) / No
