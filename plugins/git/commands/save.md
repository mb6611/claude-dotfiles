---
description: Commit all changes in logical commits, then push to main without asking
argument-hint: "[optional: commit scope]"
---

Save my work: commit everything, then push. Running this command is the go-ahead to push, so don't ask.

1. Invoke the `git:commit` skill (pass along any scope: $ARGUMENTS) to create logical conventional commits. Skip its final step of asking whether to push.
2. If the current branch isn't `main`, stop and ask before pushing anything.
3. Invoke the `git:push` skill to push to `main`. If the push is rejected, follow its handling and ask before rebasing, merging, or force-pushing.
4. Reply with the commits pushed, one line each.
