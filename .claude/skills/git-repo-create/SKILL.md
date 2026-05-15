---
name: git-repo-create
description: 'Create a new git repository on GitHub or GitLab. Use when the user wants to create a new repo, initialize a project, or set up a new remote repository.'
argument-hint: 'Repository name and optional platform (github/gitlab), e.g. "my-project github"'
---

# Create Git Repository

## When to Use
- User wants to create a new repository
- User wants to initialize a new project on GitHub or GitLab
- User says "create repo", "new repository", "init project"

## Pre-Checks
1. Read `.claude/git-lessons-learned.md` for applicable rules
2. Confirm with user:
   - Repository name
   - Platform (GitHub or GitLab)
   - Visibility (public or private)
   - Description (optional)
   - Initialize with README? (default: yes)
   - Default branch name (default: main)

## Procedure

### GitHub (via MCP)
1. Use GitHub MCP tool `create_repository` with parameters:
   - name: `<repo-name>`
   - private: `<true/false>`
   - description: `<description>`
   - auto_init: `<true/false>`
2. Verify creation was successful
3. Return the repository URL

### GitLab (via CLI)
1. Run:
   ```bash
   curl --header "PRIVATE-TOKEN: $GITLAB_PAT" \
     --data "name=<repo-name>&visibility=<public/private>&description=<description>&initialize_with_readme=<true/false>" \
     "https://gitlab.com/api/v4/projects"
   ```
2. Verify creation was successful
3. Return the repository URL

## Report Output
After completion, provide:
- **Action**: Repository created
- **Platform**: GitHub / GitLab
- **Repository URL**: [the URL]
- **Visibility**: Public / Private
- **Default Branch**: main
- **Initialized**: Yes (with README) / No (empty)
