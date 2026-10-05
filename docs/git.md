# Git Cheat Sheet

## Inspect

```bash
git status
git branch --show-current
git remote -v
git log --oneline --decorate --graph -20
git diff
git diff --staged
```

## Fetch / pull

```bash
git fetch
git fetch --all --prune
git pull --ff-only
```

`--ff-only` is a useful default when you do not want `pull` to create surprise merge commits.

## Branches

```bash
git branch
git branch -a
git switch <branch>
git switch -c <new-branch>
git branch -d <branch>
git branch -D <branch>          # DANGER
```

## Stage / commit

```bash
git add <file>
git add -p
git commit -m "message"
git commit --amend
```

## Restore

Discard unstaged changes to a file:

```bash
git restore <file>
```

Unstage:

```bash
git restore --staged <file>
```

Restore from another commit:

```bash
git restore --source=<commit> -- <file>
```

## Stash

```bash
git stash
git stash push -m "WIP"
git stash list
git stash show -p stash@{0}
git stash pop
git stash apply stash@{0}
```

Include untracked:

```bash
git stash -u
```

## Rebase

```bash
git fetch origin
git rebase origin/main
```

Interactive:

```bash
git rebase -i HEAD~5
```

Abort:

```bash
git rebase --abort
```

Continue:

```bash
git rebase --continue
```

## Cherry-pick

```bash
git cherry-pick <commit>
git cherry-pick --abort
```

## Reflog: recovery superpower

```bash
git reflog
```

Recover a lost commit/branch:

```bash
git switch -c recovery <reflog-sha>
```

## Reset

Move HEAD but keep changes staged:

```bash
git reset --soft HEAD~1
```

Keep changes unstaged:

```bash
git reset HEAD~1
```

**DANGER — discard tracked working-tree changes:**

```bash
git reset --hard <commit>
```

Check `git status` and `git reflog` first.

## Worktrees

Useful for working on two branches without constantly switching:

```bash
git worktree list
git worktree add ../repo-feature <branch>
git worktree remove ../repo-feature
```

## Find who changed something

```bash
git blame <file>
git log -S '<text>' --oneline
git log -G '<regex>' --oneline
git log -- <file>
```

## Bisect a regression

```bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
```

Test each checkout and mark:

```bash
git bisect good
git bisect bad
```

Finish:

```bash
git bisect reset
```

## Clean

Preview untracked files that would be removed:

```bash
git clean -nd
```

**DANGER:**

```bash
git clean -fd
```

## Useful config

```bash
git config --global pull.ff only
git config --global fetch.prune true
git config --global rerere.enabled true
```

`rerere` can remember previous conflict resolutions, useful during repeated rebases.
