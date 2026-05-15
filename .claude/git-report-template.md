# Git Action Report Template

Use this template for all git operation reports. Copy and fill in for each action.

---

```markdown
# Git Action Report

| Field | Value |
|-------|-------|
| **Timestamp** | [YYYY-MM-DD HH:MM:SS] |
| **Action** | [pull / push / clone / PR / MR / merge / create-repo / conflict-resolution] |
| **Repository** | [repo name or URL] |
| **Branch** | [branch name] |
| **Remote** | [remote name and URL] |
| **Platform** | [GitHub / GitLab / Local] |

## Before State
- **Branch**: [branch before action]
- **HEAD**: [commit hash]
- **Working Tree**: [clean / dirty]
- **Upstream Status**: [ahead N / behind N / up-to-date / no upstream]

## After State
- **Branch**: [branch after action]
- **HEAD**: [commit hash]
- **Working Tree**: [clean / dirty]
- **Upstream Status**: [ahead N / behind N / up-to-date]

## Changes Summary
- **Commits**: [N commits — list short hashes and messages]
- **Files Changed**: [N files]
- **Insertions**: [+N lines]
- **Deletions**: [-N lines]
- **Key Files**:
  - [file1.ext] — [brief description of change]
  - [file2.ext] — [brief description of change]

## Conflicts
[None — or list each conflict with classification and resolution status]

## Warnings
[None — or list any concerns, risks, or follow-up actions needed]

## Lessons Applied
[None — or list rules from git-lessons-learned.md that were applied]

## Human Action Required
[None — or describe what the human needs to decide/do]
```
