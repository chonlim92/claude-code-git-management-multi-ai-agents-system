---
name: git-clone
description: 'Clone a git repository to local filesystem. Use when the user wants to clone, download, or copy a repository locally.'
argument-hint: 'Repository URL or owner/repo, e.g. "https://github.com/user/repo" or "user/repo"'
---

# Clone Git Repository

## When to Use
- User wants to clone a repository
- User wants to download/copy a repo locally
- User says "clone", "download repo", "get the code"

## Pre-Checks
1. Read `.claude/git-lessons-learned.md` for applicable rules
2. Confirm with user:
   - Repository URL or owner/repo identifier
   - Target directory (default: current directory)
   - Specific branch to clone (default: default branch)
   - Shallow clone? (default: no)

## Procedure

1. Determine the clone URL:
   - If full URL provided, use as-is
   - If `owner/repo` format on GitHub: `https://github.com/owner/repo.git`
   - If `owner/repo` format on GitLab: `https://gitlab.com/owner/repo.git`

2. Run the clone:
   ```bash
   git clone <url> [target-directory] [--branch <branch>] [--depth 1 if shallow]
   ```

3. Verify clone integrity:
   ```bash
   cd <cloned-directory>
   git status
   git log --oneline -5
   ```

4. Report directory size:
   ```bash
   du -sh <cloned-directory>
   ```

## Report Output
After completion, provide:
- **Action**: Repository cloned
- **Source URL**: [the URL]
- **Local Path**: [absolute path]
- **Branch**: [branch name]
- **Latest Commit**: [short hash + message]
- **Size**: [directory size]
