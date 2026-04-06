# Module 04 — Advanced Git
> 🔴 **Level:** Advanced | ⏱️ **Time:** 4–6 hours

---

## 🎯 Learning Objectives
- Rebase, squash, and clean up commit history
- Cherry-pick specific commits across branches
- Use interactive rebase to rewrite history
- Recover lost work with reflog
- Use tags for releases
- Write Git hooks for automation

---

## 1. Rebase

Rebase **replays** your commits on top of another branch, creating a linear history.

```
Before rebase:
  main:    A ── B ── C
                \
  feature:       D ── E

After: git rebase main (from feature)
  main:    A ── B ── C
                      \
  feature:             D' ── E'   (new commits, same changes)
```

### Basic rebase
```bash
git checkout feature/login
git rebase main           # Replay feature commits on top of main

# If conflicts occur:
# 1. Fix the conflict in the file
git add fixed-file.txt
git rebase --continue     # Move to next commit

# Abort and return to original state
git rebase --abort
```

### ⚠️ The Golden Rule
**NEVER rebase commits that exist on a shared/remote branch.**  
Rebasing rewrites history (new commit hashes). If others have those commits, their history will diverge and cause chaos.

✅ Safe: rebase your local feature branch before pushing  
❌ Dangerous: rebase `dev` or `main` after others have pulled

---

## 2. Interactive Rebase — Rewriting History

Interactive rebase lets you edit, squash, reorder, or drop commits.

```bash
# Rewrite the last 4 commits
git rebase -i HEAD~4
```

This opens an editor:
```
pick a1b2c3d feat: add login form
pick e4f5g6h fix: login form validation
pick i7j8k9l fix: typo in login
pick m1n2o3p test: add login tests

# Commands:
# p, pick   = use commit as-is
# r, reword = use commit, but edit message
# e, edit   = pause and amend this commit
# s, squash = combine into previous commit (keep message)
# f, fixup  = combine into previous commit (discard message)
# d, drop   = remove commit entirely
```

### Squash example — clean up messy commits before PR

```
# Before (messy)
pick a1b2c3d feat: add login form
f    e4f5g6h fix: validation      ← squash into feat
f    i7j8k9l fix: typo            ← squash into feat
pick m1n2o3p test: add login tests

# Result: 2 clean commits
# feat: add login form
# test: add login tests
```

---

## 3. Cherry-Pick

Apply a specific commit from one branch to another:

```bash
# Get the commit hash from git log
git log --oneline feature/payments
# abc1234 feat: add stripe checkout
# def5678 fix: error handling

# Apply just the fix to main
git checkout main
git cherry-pick def5678

# Cherry-pick multiple commits
git cherry-pick abc1234 def5678

# Cherry-pick a range
git cherry-pick abc1234..def5678

# Cherry-pick without committing (just apply changes)
git cherry-pick --no-commit def5678
```

### When to use cherry-pick
- Hotfix: cherry-pick a fix from dev onto main
- You committed to the wrong branch — cherry-pick to correct branch, then drop the original
- Backport: apply a new feature to an older release branch

---

## 4. Git Reflog — Your Safety Net

`reflog` records every movement of HEAD. It's how you recover "lost" work.

```bash
git reflog                  # Show all recent HEAD positions

# Output example:
# abc1234 HEAD@{0}: commit: feat: new feature
# def5678 HEAD@{1}: rebase: finish
# ghi9012 HEAD@{2}: checkout: moving to feature/login
# jkl3456 HEAD@{3}: commit: fix: button color

# Recover a dropped commit
git checkout HEAD@{3}
# or create a branch from it:
git branch recovered-work HEAD@{3}

# Recover after accidental git reset --hard
git reset --hard HEAD@{2}
```

> 💡 Reflog entries expire after 90 days by default. Use it quickly after a mistake.

---

## 5. Reset vs Revert

Both undo things — but they work very differently.

### `git reset` — moves the branch pointer back
```bash
git reset --soft HEAD~1    # Undo commit, keep changes STAGED
git reset --mixed HEAD~1   # Undo commit, keep changes UNSTAGED (default)
git reset --hard HEAD~1    # Undo commit, DISCARD all changes ⚠️

# Only use reset on LOCAL, unpushed commits
# Never reset commits others have pulled
```

### `git revert` — creates a new "undo" commit
```bash
git revert abc1234         # Adds a new commit that undoes abc1234
git revert HEAD~3..HEAD    # Revert last 3 commits

# This is SAFE to use on shared branches
# It doesn't rewrite history — it adds to it
```

| | `reset` | `revert` |
|--|---------|---------|
| Rewrites history? | ✅ Yes | ❌ No |
| Safe on shared branches? | ❌ No | ✅ Yes |
| Commit added? | No | Yes |
| Use when | Local cleanup | Undoing pushed changes |

---

## 6. Git Tags

Tags mark specific commits — typically used for **releases**.

```bash
# Lightweight tag (just a pointer)
git tag v1.2.0

# Annotated tag (recommended — includes message, author, date)
git tag -a v1.2.0 -m "Release version 1.2.0"

# Tag a specific commit
git tag -a v1.1.0 abc1234 -m "Hotfix release"

# List all tags
git tag
git tag -l "v1.*"          # Filter by pattern

# Push tags to remote (they don't push automatically!)
git push origin v1.2.0     # Push one tag
git push origin --tags     # Push all tags

# Delete a tag
git tag -d v1.2.0
git push origin --delete v1.2.0
```

---

## 7. Git Hooks

Hooks are scripts that run automatically at Git events. They live in `.git/hooks/`.

### Common hooks

| Hook | When it runs | Common use |
|------|-------------|------------|
| `pre-commit` | Before commit | Lint code, run tests |
| `commit-msg` | After commit message written | Validate message format |
| `pre-push` | Before push | Run tests, security scan |
| `post-merge` | After a merge | Rebuild deps, notify |

### Example: pre-commit hook that runs ESLint

```bash
# .git/hooks/pre-commit
#!/bin/sh

echo "Running linter..."
npx eslint src/
if [ $? -ne 0 ]; then
  echo "❌ Lint errors found. Commit blocked."
  exit 1
fi
echo "✅ Lint passed."
```

```bash
chmod +x .git/hooks/pre-commit   # Make it executable
```

### Sharing hooks with the team (hooks aren't committed)
Use **Husky** (for Node.js projects):

```bash
npm install --save-dev husky
npx husky init
# Creates .husky/ folder — this IS committed to the repo
```

```bash
# .husky/pre-commit
npx lint-staged
```

---

## 8. Bisect — Find the Commit That Broke Everything

```bash
# Start bisect
git bisect start
git bisect bad                    # Current commit is broken
git bisect good v1.2.0            # This tag was working

# Git checks out a midpoint commit
# Test your app, then tell Git:
git bisect good                   # This commit is fine
git bisect bad                    # This commit is broken

# Git narrows it down until it finds the first bad commit
# When done:
git bisect reset                  # Return to original HEAD
```

---

## 9. Worktrees — Multiple Branches Checked Out Simultaneously

```bash
# Check out dev branch in a separate directory (without switching)
git worktree add ../project-dev dev

# List worktrees
git worktree list

# Remove a worktree
git worktree remove ../project-dev
```

Useful when: reviewing a PR while still working on your feature.

---

## ✅ Module 04 Checklist
- [ ] Rebased a feature branch onto `dev`
- [ ] Used interactive rebase to squash 3 commits into 1
- [ ] Cherry-picked a single commit to another branch
- [ ] Used `git reflog` to recover a "lost" commit
- [ ] Used `git revert` to undo a pushed commit safely
- [ ] Created and pushed an annotated tag
- [ ] Written a `pre-commit` hook that runs a check
- [ ] Used `git bisect` to find a broken commit

---

➡️ **Next:** [Module 05 — Production Workflows](../module-05-production/README.md)
