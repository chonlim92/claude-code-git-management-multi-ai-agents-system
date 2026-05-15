---
name: git-release
description: 'Create a release on GitHub or GitLab with version tag, release notes, and optional assets. Use when the user wants to publish a release, tag a version, or create a release with notes.'
argument-hint: 'Version tag and optional platform, e.g. "v1.2.0 github" or "v2.0.0 gitlab"'
---

# Create Release

## When to Use
- User wants to create a release on GitHub or GitLab
- User says "release", "publish version", "tag a release", "create release"
- User wants to tag a version and publish release notes

## Pre-Checks
1. Read `.claude/git-lessons-learned.md` for applicable rules
2. Confirm with user:
   - Version tag (e.g., `v1.2.0`) — must follow semantic versioning
   - Platform (GitHub or GitLab)
   - Target branch (default: main)
   - Release title (default: same as version tag)
   - Release notes (auto-generate from commits or user-provided)
   - Pre-release? (default: no)
   - Assets to attach (optional)
3. Verify current branch state:
   ```bash
   git status
   git branch --show-current
   ```

## Procedure

### Step 1: Validate Version Tag

```bash
# Check if tag already exists
git tag -l "<version>"

# Get latest tag for reference
git describe --tags --abbrev=0 2>/dev/null || echo "No existing tags"
```

- If tag already exists: STOP, inform user, ask for a different version
- Validate format follows semver (vMAJOR.MINOR.PATCH)

### Step 2: Generate Release Notes (if not provided)

```bash
# Get commits since last tag
git log $(git describe --tags --abbrev=0 2>/dev/null)..HEAD --oneline --no-merges
```

Format release notes as:
```markdown
## What's Changed
- [commit message] ([short hash])
- ...

## Full Changelog
[previous-tag]...[new-tag]
```

### Step 3: Create Git Tag

```bash
git tag -a <version> -m "<Release title>"
git push origin <version>
```

### Step 4: Create Release

#### GitHub (via MCP)

Use GitHub MCP tool `create_release` with parameters:
- owner: `<repo-owner>`
- repo: `<repo-name>`
- tag_name: `<version>`
- name: `<release title>`
- body: `<release notes>`
- target_commitish: `<target branch>`
- draft: `false`
- prerelease: `<true/false>`

#### GitLab (via API)

```bash
curl --header "PRIVATE-TOKEN: $GITLAB_PAT" \
  --data "name=<release-title>&tag_name=<version>&description=<release-notes>&ref=<target-branch>" \
  "https://gitlab.com/api/v4/projects/<project-id>/releases"
```

### Step 5: Upload Assets (if any)

#### GitHub
Use GitHub MCP tool to upload release assets to the created release.

#### GitLab
```bash
curl --header "PRIVATE-TOKEN: $GITLAB_PAT" \
  --form "file=@<filepath>" \
  "https://gitlab.com/api/v4/projects/<project-id>/uploads"
```

Then link the uploaded file to the release.

## Human Review

Before creating the release, present summary for confirmation:

```
## Release Summary

| Field | Value |
|-------|-------|
| Version | [tag] |
| Platform | GitHub / GitLab |
| Target Branch | [branch] |
| Title | [title] |
| Pre-release | Yes / No |
| Assets | [list or "None"] |

### Release Notes Preview
[generated or provided notes]

Proceed with creating this release? (YES / NO)
```

## Report Output
After completion, provide:
- **Action**: Release created
- **Platform**: GitHub / GitLab
- **Version**: [tag]
- **Release URL**: [link to release page]
- **Target Branch**: [branch]
- **Release Notes**: [included/linked]
- **Assets**: [list or "None"]
- **Previous Version**: [previous tag or "First release"]
