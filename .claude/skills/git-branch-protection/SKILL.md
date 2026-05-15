---
name: git-branch-protection
description: 'Verify main branch integrity and enforce branch protection rules. Use before any merge, push to main, or operation that could affect the main branch.'
argument-hint: 'Optional target branch to check, defaults to main'
---

# Branch Protection

## When to Use
- Before any merge into main/master
- Before any direct push to main/master
- After any merge to verify main is still functional
- When user asks "is main safe?", "check main branch"
- Called by other skills/agents as a safety gate

## Core Principle

> **The main branch must ALWAYS remain functional.**
> Feature branches adapt to main. Main NEVER adapts to feature branches.

## Pre-Merge Checks

### Check 1: Main Branch State
```bash
git fetch origin
git checkout main
git status
git log --oneline -5
```
- Is main clean? (no uncommitted changes)
- Is main up-to-date with remote?

### Check 2: Feature Branch Compatibility
```bash
git log --oneline main..<feature-branch>
git merge-base main <feature-branch>
```
- Is feature branch based on current main? (not stale)
- If stale: feature branch must rebase/merge main first

### Check 3: Conflict Status
```bash
git merge --no-commit --no-ff <feature-branch> 2>&1
git merge --abort
```
- Any conflicts? If yes, feature branch must resolve them FIRST

### Check 4: CI/Test Status (if available)
```bash
# Check GitHub CI status via MCP
# Or check local tests:
# npm test / pytest / make test (depending on project)
```
- Are tests passing on the feature branch?
- Will the merge break any existing tests?

### Check 5: Breaking Change Detection
```bash
git diff --stat main..<feature-branch>
git diff main..<feature-branch> -- "*.lock" "package.json" "requirements.txt" "Cargo.toml"
```
- Are dependency files changed?
- Are critical config files modified?
- Are public APIs changed (function signatures, endpoints)?

## Post-Merge Verification

After a merge is completed:
```bash
git log --oneline -3
git status
# Run tests if available
```
- Verify HEAD is a merge commit (or fast-forward as expected)
- Verify no unexpected files are staged/modified
- Run tests to confirm nothing broke

## Protection Rules

1. **No direct commits to main** — all changes via PR/MR
2. **No force pushes to main** — ever
3. **Feature branches must be up-to-date** — rebase or merge main into feature before merging feature into main
4. **All conflicts resolved in feature** — never resolve conflicts by modifying main
5. **Tests must pass** — if tests exist, they must pass before merge

## Report Output

```markdown
## Branch Protection Check

### Target Branch: [main/master]
| Check | Status | Details |
|-------|--------|---------|
| Branch state | PASS/FAIL | [clean/dirty, up-to-date/behind] |
| Feature compatibility | PASS/FAIL | [based on current main / stale] |
| Conflict status | PASS/FAIL | [clean / N conflicts found] |
| CI/Tests | PASS/FAIL/SKIP | [status or "no tests configured"] |
| Breaking changes | PASS/WARN | [none / list of concerns] |

### Overall Verdict: SAFE TO MERGE / BLOCKED
[If blocked: list what must be fixed before merge can proceed]
```

## Blocking Behavior

If ANY check fails:
- **DO NOT proceed with the merge**
- Report all failures clearly
- List specific actions needed to unblock
- Return control to the requesting agent/user
