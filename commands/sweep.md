---
description: Remove every git worktree whose work is finished (on a pushed branch with a PR, or merged). Asks before touching anything with uncommitted changes.
allowed-tools: Bash(git worktree:*), Bash(git status:*), Bash(git branch:*), Bash(git log:*), Bash(git rev-parse:*), Bash(git fetch:*), Bash(gh pr list:*), Bash(gh pr view:*)
---

Clean up finished worktrees in this repository.

1. Run `git fetch --prune` and `git worktree list --porcelain`. Ignore the main worktree.
2. For each other worktree, find its branch and decide:
   - **Done:** the branch has a PR (`gh pr list --head <branch> --state all --json number,state`) and `git -C <path> status --porcelain` is empty and no local commits are missing from the remote branch; or the branch is fully merged into the default branch (`git branch --merged`).
   - **Not done:** anything else.
3. Remove each done worktree with `git worktree remove <path>`, then run `git worktree prune`. Never pass `--force`.
4. For a worktree that is done except for uncommitted changes or unpushed commits, don't touch it: list it and ask the user what to do.
5. Report a short table: worktree, branch, PR, action taken.
