---
description: Push commits to remote repository
argument-hint: "[optional: remote branch]"
---

Push local commits to the remote repository.

## Phase 1: Pre-push Checks

1. Check current branch: `git branch --show-current`
2. Check if there are commits to push: `git log @{upstream}..HEAD --oneline 2>/dev/null || git log origin/$(git branch --show-current)..HEAD --oneline 2>/dev/null`
3. If no commits to push, inform the user and exit

## Phase 2: Verify Remote

1. Check if upstream is set: `git rev-parse --abbrev-ref @{upstream} 2>/dev/null`
2. If no upstream, set it: `git push -u origin [branch]`
3. If upstream exists, push normally: `git push`

## Phase 3: Handle Push Issues

### If push is rejected (non-fast-forward)
1. Fetch latest: `git fetch origin`
2. Show the user what's different: `git log HEAD..origin/[branch] --oneline`
3. Ask user how they want to proceed:
   - Rebase: `git pull --rebase && git push`
   - Merge: `git pull && git push`
   - Force push (with caution): `git push --force-with-lease`

### If push fails for other reasons
1. Show the error to the user
2. Suggest possible solutions

## Phase 4: Confirm

1. Confirm push was successful
2. Show what was pushed: `git log @{upstream}..HEAD --oneline` (should be empty after push)
3. Show the remote URL for reference

## Key Reminders

- **Never force push to main/master** without explicit user confirmation
- **Use `--force-with-lease`** instead of `--force` when force pushing is necessary
- **Always check** what will be pushed before pushing
