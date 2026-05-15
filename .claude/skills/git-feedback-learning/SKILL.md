---
name: git-feedback-learning
description: 'Process human feedback from declined PRs, MR comments, or corrections into lasting rules. Use when human provides feedback, rejects a proposal, or corrects agent behavior.'
argument-hint: 'The feedback or comment to process'
---

# Feedback Learning

## When to Use
- Human declines/rejects a PR or MR
- Human leaves comments or corrections on a proposal
- Human says "don't do that", "that's wrong", "next time do X"
- Human provides any feedback on git operations
- After a failed merge attempt that was caused by agent error
- When human approves something with notes ("good, but next time also...")

## Procedure

### Step 1: Parse the Feedback

Identify from the human's feedback:
- **What was wrong**: The specific issue or mistake
- **What should change**: The correct behavior going forward
- **Context**: When/where this applies (specific repos, file types, scenarios)
- **Severity**: Critical (must never happen) / Important (should avoid) / Preference (nice to have)

### Step 2: Formulate the Rule

Write a clear, actionable rule:
- Start with WHEN (the trigger condition)
- Then DO or DO NOT (the required behavior)
- Then BECAUSE (the reason — helps judge edge cases)

Format:
```
RULE [N]: [Short title]
- WHEN: [specific scenario or condition]
- DO/DO NOT: [required behavior]
- BECAUSE: [reason from human feedback]
- SEVERITY: Critical / Important / Preference
- ADDED: [date]
- CONTEXT: [original feedback quote]
```

### Step 3: Write to Shared Project File

Append the new rule to `.claude/git-lessons-learned.md`:

```bash
# Read current file to get the next rule number
cat .claude/git-lessons-learned.md
```

Add the new rule at the end of the rules section.

### Step 4: Save to Claude Memory

Save the lesson as a feedback-type memory in the Claude memory system with:
- The rule itself
- Why it matters (the human's reasoning)
- How to apply it (when this kicks in)

### Step 5: Apply Immediately

If the feedback relates to the current task:
1. Identify what needs to change in the current work
2. Apply the lesson immediately
3. Re-run the failed operation with the correction
4. Present the corrected result to the human

### Step 6: Report

```markdown
## Feedback Processed

### What Was Learned
[One-line summary of the lesson]

### Rule Created
[The full rule as written to git-lessons-learned.md]

### Immediate Application
[What was changed in the current task based on this feedback — or "N/A, feedback is for future operations"]

### Stored In
- [x] `.claude/git-lessons-learned.md` (shared team rules)
- [x] Claude memory system (personal retention)

### Impact
This rule will be automatically checked before all future [relevant operation type] operations.
```

## Categories of Lessons

When categorizing, use these tags in the lessons-learned file:

- `[CODE-QUALITY]` — Code formatting, structure, naming, patterns
- `[PROCESS]` — Workflow, timing, sequencing, when to do what
- `[SAFETY]` — What to avoid, what breaks things, danger zones
- `[PREFERENCE]` — How the human/team prefers things done
- `[CONFLICT]` — How to handle specific types of merge conflicts
- `[REVIEW]` — What to check during review, what reviewers care about
- `[BRANCH]` — Branching strategy, naming, lifecycle rules

## Continuous Improvement

After writing a rule:
1. Check if it contradicts or supersedes any existing rule
2. If yes, update or remove the old rule
3. Check if related rules can be consolidated
4. Keep the lessons-learned file organized and free of duplicates
