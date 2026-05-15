---
name: git-push
description: 'Push local commits to remote repository. Use when the user wants to push, upload, or send changes to remote.'
argument-hint: 'Optional remote and branch, e.g. "origin feature-branch"'
---

# Push Changes to Remote

## When to Use
- User wants to push commits to remote
- User says "push", "upload", "send to remote", "publish branch"

## Pre-Checks
1. Read `.claude/git-lessons-learned.md` for applicable rules
2. Check current state:
   ```bash
   git status
   git branch --show-current
   git log --oneline @{upstream}..HEAD 2>/dev/null || git log --oneline -5
   ```
3. Run `/git-branch-protection` check

## Safety Gates

### Gate 1: Branch Check
- If current branch is `main` or `master`:
  - **STOP** — warn the user
  - Ask for explicit confirmation: "You are about to push directly to main. Are you sure?"
  - If user does not confirm, abort

### Gate 2: Unpushed Commits Review
- Show the user what will be pushed:
  ```bash
  git log --oneline @{upstream}..HEAD
  git diff --stat @{upstream}..HEAD
  ```
- Ask: "These commits will be pushed. Proceed?"

### Gate 3: Remote State
- Check if remote has new commits:
  ```bash
  git fetch origin
  git status
  ```
- If behind remote: suggest pull first before pushing

## Procedure

1. Record before-state:
   ```bash
   git rev-parse HEAD
   git log --oneline -1
   ```

2. Push to remote:
   ```bash
   git push <remote|origin> <branch|current-branch>
   ```
   - If push is rejected (non-fast-forward):
     - DO NOT force push
     - Report the rejection
     - Suggest: pull first, resolve conflicts, then push again

3. If new branch (no upstream):
   ```bash
   git push -u <remote|origin> <branch>
   ```

4. Verify push succeeded:
   ```bash
   git status
   git log --oneline origin/<branch> -3
   ```

## Report Output
After completion, provide:
- **Action**: Push completed
- **Branch**: [branch name]
- **Remote**: [remote name]
- **Commits Pushed**: [number and list]
- **Files Changed**: [number]
- **Push Result**: Success / Rejected (with reason)
- **Warnings**: [any concerns, e.g., "pushed to main"]
