---
description: Create a pull request for the current branch
argument-hint: "[target branch, e.g., main]"
---

Create a pull request for the current branch.

## Phase 1: Gather Context

1. Get current branch name: `git branch --show-current`
2. Check if branch has a remote: `git rev-parse --abbrev-ref @{upstream} 2>/dev/null`
3. If no upstream, push the branch: `git push -u origin [branch]`
4. Get the default branch if no target specified: `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`

## Phase 2: Analyze Changes

1. Compare with target branch: `git log [target]..HEAD --oneline`
2. Review the diff: `git diff [target]...HEAD --stat`
3. Understand what the PR accomplishes

## Phase 3: Create PR

Use GitHub CLI to create the PR:

```bash
gh pr create --base [target-branch] --title "title" --body "body"
```

### PR Title
- Use conventional commit style: `type(scope): description`
- Keep it concise but descriptive

### PR Body Format
```
## Summary
Brief description of what this PR does.

## Changes
- Key change 1
- Key change 2

## Testing
How the changes were tested.
```

## Phase 4: Report

After creating the PR:
1. Display the PR URL
2. Show PR number
3. Confirm the target branch
