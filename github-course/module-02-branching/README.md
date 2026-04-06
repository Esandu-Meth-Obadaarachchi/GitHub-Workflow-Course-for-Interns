# Module 02 — Branching & Merging
> 🟡 **Level:** Intermediate | ⏱️ **Time:** 3–4 hours

---

## 🎯 Learning Objectives
- Create, switch, and delete branches
- Understand fast-forward vs 3-way merges
- Resolve merge conflicts confidently
- Know when to merge vs rebase
- Use branch naming conventions used in real teams

---

## 1. Why Branches?

Branches let you work on features, fixes, and experiments **in isolation** — without breaking the main codebase.

```
main:      A ── B ── C ─────────────────── G   (stable)
                      \                   /
feature:               D ── E ── F ──────     (your work)
```

Every branch is just a **pointer** to a commit. Creating a branch is instant and free in Git.

---

## 2. Branch Commands

### Create & switch
```bash
git branch feature/login          # Create branch
git switch feature/login          # Switch to it
git switch -c feature/login       # Create AND switch (shortcut)
git checkout -b feature/login     # Older syntax (same result)
```

### View branches
```bash
git branch                  # Local branches
git branch -r               # Remote branches
git branch -a               # All branches (local + remote)
git branch -v               # Show last commit on each branch
```

### Delete branches
```bash
git branch -d feature/login       # Delete (safe — blocks if unmerged)
git branch -D feature/login       # Force delete
git push origin --delete feature/login   # Delete remote branch
```

---

## 3. Branch Naming Conventions

Use a consistent naming pattern. Recommended:

```
<type>/<ticket-or-description>

feature/user-authentication
fix/null-pointer-on-checkout
chore/upgrade-node-18
docs/api-reference-update
release/v2.3.0
hotfix/payment-gateway-timeout
```

> 💡 Using `/` creates visual grouping in Git GUIs and GitHub.

---

## 4. Merging

### Fast-Forward Merge (no divergence)
When the target branch has no new commits since the branch was created:

```
Before:   main: A ── B
                       \
               feat:   C ── D

After FF: main: A ── B ── C ── D   (clean, linear history)
```

```bash
git checkout main
git merge feature/login           # Git does FF automatically when possible
```

### 3-Way Merge (diverged branches)
When both branches have moved forward:

```
Before:   main: A ── B ── E
                       \
               feat:   C ── D

After:    main: A ── B ── E ── M  (M = merge commit)
                       \       /
               feat:   C ── D
```

```bash
git checkout main
git merge feature/login           # Creates a merge commit
git merge --no-ff feature/login   # Force merge commit even if FF is possible
```

### When to use `--no-ff`
Use `--no-ff` (no fast-forward) on important merges (like merging to `main` or `dev`) to preserve the history of when a feature was integrated.

---

## 5. Resolving Merge Conflicts

A conflict happens when two branches changed the **same lines** of the same file.

### What a conflict looks like
```
<<<<<<< HEAD (your current branch)
  const greeting = "Hello, world!";
=======
  const greeting = "Hi there!";
>>>>>>> feature/new-greeting
```

### Resolution steps
```bash
# 1. Trigger the conflict
git merge feature/new-greeting
# OUTPUT: CONFLICT in src/app.js

# 2. See which files conflict
git status

# 3. Open the file and edit it to your desired final state
#    Remove all the <<<, ===, >>> markers

# 4. Stage the resolved file
git add src/app.js

# 5. Complete the merge
git commit -m "merge: resolve greeting conflict"

# To abort if you're not ready
git merge --abort
```

### Using a merge tool
```bash
git mergetool                # Opens configured visual merge tool
git config --global merge.tool vscode
```

---

## 6. Merge vs Rebase (When to Use Each)

| | Merge | Rebase |
|--|-------|--------|
| History | Preserves full history with merge commits | Creates linear history |
| Safe to use on shared branches? | ✅ Yes | ❌ No (rewrites history) |
| Good for | `dev` → `main`, PR merges | Keeping feature branch up-to-date |
| Conflict resolution | Once, in merge commit | Once per commit being replayed |

### Rebase (preview — covered fully in Module 04)
```bash
# Update your feature branch with latest dev
git checkout feature/login
git rebase dev        # Replays your commits on top of dev

# Golden rule: NEVER rebase branches others are working on
```

---

## 7. Viewing Branch Differences

```bash
# What commits are in feature but not in main?
git log main..feature/login --oneline

# What files changed between branches?
git diff main..feature/login

# What's been merged into main already?
git branch --merged main

# What hasn't been merged yet?
git branch --no-merged main
```

---

## 8. Stashing Work-in-Progress

When you need to switch branches but have uncommitted work:

```bash
git stash                         # Save WIP to stash
git stash push -m "half-done login form"   # With a label

git stash list                    # See all stashes
git stash pop                     # Apply latest stash & remove it
git stash apply stash@{1}        # Apply a specific stash (keep it)
git stash drop stash@{0}         # Delete a stash
git stash clear                   # Delete all stashes
```

---

## 9. Team Branch Workflow Example

```bash
# Start of your day
git checkout dev
git pull origin dev               # Always sync dev first!

# Create your feature branch
git checkout -b feature/invoice-export

# ... do your work, commit often ...
git add .
git commit -m "feat(invoices): add CSV export button"

# Before opening PR, sync with latest dev
git fetch origin
git rebase origin/dev             # Rebase your commits on top of updated dev

# Push your branch
git push -u origin feature/invoice-export

# Open a Pull Request on GitHub: feature/invoice-export → dev
```

---

## ✅ Module 02 Checklist
- [ ] Created and switched between 3+ branches
- [ ] Performed a fast-forward merge
- [ ] Performed a 3-way merge (with merge commit)
- [ ] Intentionally created and resolved a merge conflict
- [ ] Used `git stash` to save and restore WIP
- [ ] Viewed branch differences with `git log branch1..branch2`
- [ ] Deleted a local and remote branch

---

➡️ **Next:** [Module 03 — Collaboration on GitHub](../module-03-collaboration/README.md)
