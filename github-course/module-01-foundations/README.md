# Module 01 — Git & GitHub Foundations
> 🟢 **Level:** Beginner | ⏱️ **Time:** 3–4 hours

---

## 🎯 Learning Objectives
By the end of this module you will:
- Understand the difference between Git and GitHub
- Configure Git on your machine
- Create and clone repositories
- Stage, commit, and push changes confidently
- Write a proper `.gitignore`

---

## 1. Git vs GitHub

| | Git | GitHub |
|--|-----|--------|
| What | Version control system (local tool) | Cloud hosting + collaboration platform |
| Where | Runs on your computer | Website / remote server |
| Who made it | Linus Torvalds (2005) | GitHub Inc. (now Microsoft) |
| Works offline? | ✅ Yes | ❌ No |

> **Analogy:** Git is like "Track Changes" in Word. GitHub is Google Drive where you share that document with your team.

---

## 2. Installation & First-Time Configuration

### Install Git
```bash
# macOS
brew install git

# Ubuntu/Debian
sudo apt update && sudo apt install git

# Windows — download from https://git-scm.com
```

### Configure your identity (do this ONCE per machine)
```bash
git config --global user.name "Your Name"
git config --global user.email "you@yourcompany.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"   # or vim, nano, etc.

# Verify
git config --list
```

> ⚠️ Your name and email appear on **every commit forever**. Use your real work email.

---

## 3. The Three States of Git

```
┌────────────────────┐   git add    ┌─────────────┐   git commit  ┌─────────────┐
│  Working Directory │ ──────────► │ Staging Area│ ────────────► │ Repository  │
│  (what you edit)   │             │  (snapshot  │               │  (.git dir) │
│                    │ ◄────────── │   preview)  │               │             │
└────────────────────┘ git restore └─────────────┘               └─────────────┘
```

- **Working Directory** — Files on your disk you can edit
- **Staging Area** — Files "queued" for the next commit
- **Repository** — The permanent history stored in `.git/`

---

## 4. Creating a Repository

### Start from scratch
```bash
mkdir my-project && cd my-project
git init
```

### Clone an existing repo from GitHub
```bash
git clone https://github.com/user/repo-name.git
cd repo-name
```

### Create on GitHub first, then clone (recommended for teams)
1. GitHub → **New Repository** → fill in name, README, license
2. Copy the HTTPS or SSH URL
3. `git clone <url>`

---

## 5. Core Daily Commands

### See what's happening
```bash
git status                        # What files changed?
git log --oneline --graph --all   # Visual history of all branches
git diff                          # Changes not yet staged
git diff --staged                 # Changes staged but not committed
```

### Stage & Commit
```bash
git add filename.txt              # Stage one file
git add .                         # Stage everything
git add -p                        # Interactively choose chunks to stage

git commit -m "feat: add login"   # Commit staged changes
git commit -am "fix: typo"        # Stage ALL tracked files + commit
```

### Undo things safely
```bash
git restore --staged file.txt     # Unstage (keep changes)
git restore file.txt              # Discard working dir changes ⚠️ irreversible
git commit --amend -m "new msg"   # Fix last commit message (BEFORE pushing)
```

---

## 6. Working with GitHub (Remotes)

```bash
git remote -v                           # View remotes
git remote add origin <url>             # Link to GitHub repo

git push -u origin main                 # First push (sets upstream)
git push                                # Every push after that

git pull                                # Fetch + merge latest
git fetch origin                        # Download changes, don't merge yet
```

---

## 7. Writing Good Commit Messages

### Conventional Commits format
```
<type>(<scope>): <short description>

[optional longer explanation]
[optional footer: BREAKING CHANGE, Closes #123]
```

### Commit types
| Type | Use when… |
|------|-----------|
| `feat` | Adding a new feature |
| `fix` | Fixing a bug |
| `docs` | Documentation only |
| `style` | Formatting, whitespace |
| `refactor` | Code restructure, no behavior change |
| `test` | Adding or updating tests |
| `chore` | Build scripts, dependencies |

### Examples
```bash
# ✅ Good
git commit -m "feat(auth): add OAuth2 Google login"
git commit -m "fix(api): handle null response from payment gateway"
git commit -m "docs: add architecture diagram to README"

# ❌ Bad
git commit -m "update stuff"
git commit -m "WIP"
git commit -m "fix"
```

---

## 8. The .gitignore File

```gitignore
# OS junk
.DS_Store
Thumbs.db

# Dependencies (never commit these)
node_modules/
vendor/
__pycache__/

# Secrets & environment (NEVER commit these)
.env
.env.local
*.pem
*.key

# Build artifacts
dist/
build/
*.o
*.pyc
*.class

# IDE settings
.vscode/
.idea/
*.swp
```

> 💡 Use [gitignore.io](https://www.toptal.com/developers/gitignore) to generate a `.gitignore` for your stack.

---

## 9. Reading History

```bash
git log                    # Full history
git log --oneline          # One line per commit
git log -p                 # Show diffs per commit
git log --author="Alice"   # Filter by author
git log --grep="login"     # Search commit messages
git show abc1234           # Show a specific commit
git blame filename.txt     # See who wrote each line
```

---

## ✅ Module 01 Checklist
- [ ] Git installed and configured (name, email, defaultBranch)
- [ ] Created a local repo with `git init`
- [ ] Cloned a repo from GitHub
- [ ] Made 3+ commits using Conventional Commits format
- [ ] Created and committed a `.gitignore`
- [ ] Pushed to GitHub and confirmed it appears online
- [ ] Used `git log --oneline --graph --all`

---

➡️ **Next:** [Module 02 — Branching & Merging](../module-02-branching/README.md)
