# 🐙 Git & GitHub Quick Reference Cheatsheet

## Setup
```bash
git config --global user.name "Name"
git config --global user.email "email"
git config --global init.defaultBranch main
git config --list
```

## Creating Repos
```bash
git init                          # New local repo
git clone <url>                   # Clone remote
git clone <url> <folder>          # Clone into specific folder
```

## Staging & Committing
```bash
git status                        # What's changed?
git add <file>                    # Stage file
git add .                         # Stage everything
git add -p                        # Stage interactively
git commit -m "feat: message"     # Commit
git commit -am "fix: message"     # Stage tracked + commit
git commit --amend                # Edit last commit (before push)
```

## Undoing
```bash
git restore <file>                # Discard working dir changes ⚠️
git restore --staged <file>       # Unstage file
git reset --soft HEAD~1           # Undo commit, keep staged
git reset --mixed HEAD~1          # Undo commit, keep unstaged
git reset --hard HEAD~1           # Undo commit, discard changes ⚠️
git revert <hash>                 # Safe undo on shared branches
```

## Branches
```bash
git branch                        # List local
git branch -a                     # List all
git switch -c <name>              # Create + switch
git switch <name>                 # Switch
git branch -d <name>              # Delete (safe)
git branch -D <name>              # Force delete
git push origin --delete <name>   # Delete remote branch
```

## Merging & Rebasing
```bash
git merge <branch>                # Merge into current
git merge --no-ff <branch>        # Force merge commit
git merge --abort                 # Abort in-progress merge
git rebase <branch>               # Rebase onto branch
git rebase -i HEAD~N              # Interactive rebase last N commits
git rebase --continue             # After fixing conflict
git rebase --abort                # Abort rebase
```

## Remote
```bash
git remote -v                     # View remotes
git remote add origin <url>       # Add remote
git push -u origin <branch>       # First push
git push                          # Push
git pull                          # Fetch + merge
git fetch origin                  # Download without merging
```

## History & Diff
```bash
git log --oneline --graph --all   # Visual history
git log --author="Name"           # Filter by author
git log --grep="keyword"          # Search messages
git diff                          # Unstaged changes
git diff --staged                 # Staged changes
git diff branch1..branch2         # Diff between branches
git blame <file>                  # Who changed what line
git show <hash>                   # Show commit details
```

## Stash
```bash
git stash                         # Save WIP
git stash push -m "label"         # Save with label
git stash list                    # List stashes
git stash pop                     # Apply latest + remove
git stash apply stash@{1}        # Apply specific stash
git stash drop stash@{0}         # Delete stash
```

## Tags
```bash
git tag -a v1.0.0 -m "msg"       # Create annotated tag
git tag                           # List tags
git push origin --tags            # Push all tags
git push origin v1.0.0            # Push one tag
git tag -d v1.0.0                 # Delete local tag
git push origin --delete v1.0.0   # Delete remote tag
```

## Advanced
```bash
git cherry-pick <hash>            # Apply single commit
git cherry-pick <a>..<b>          # Apply range of commits
git reflog                        # Recovery log
git bisect start                  # Start binary search
git bisect good <hash>            # Mark commit as good
git bisect bad                    # Mark current as bad
git bisect reset                  # End bisect
git worktree add <path> <branch>  # Multiple checkouts
```

## Conventional Commits
```
feat:     new feature
fix:      bug fix
docs:     documentation
style:    formatting
refactor: code restructure
test:     tests
chore:    build / deps
```

## Branch Naming
```
feature/short-description
fix/bug-description
hotfix/critical-fix
release/v1.2.0
chore/dependency-update
docs/update-readme
```

## Team Workflow (Daily)
```bash
# Start of day
git checkout dev
git pull origin dev
git checkout -b feature/your-task

# During work
git add . && git commit -m "feat: ..."

# Before PR
git fetch origin
git rebase origin/dev
git push -u origin feature/your-task

# Open PR → dev on GitHub
```
