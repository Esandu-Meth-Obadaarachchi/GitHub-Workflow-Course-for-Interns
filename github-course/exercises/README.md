# 🧪 Hands-On Exercises

Complete these in order. Each builds on the previous.

---

## Exercise 01 — Foundations

### Lab 1A: Your First Repo
```bash
# 1. Create a new folder and initialize a Git repo
mkdir my-first-repo && cd my-first-repo
git init

# 2. Configure your identity (if not done globally)
git config user.name "Your Name"
git config user.email "you@example.com"

# 3. Create a file and make your first commit
echo "# My First Repo" > README.md
git add README.md
git commit -m "docs: initial README"

# 4. Create an index.html with some basic HTML
# 5. Add a .gitignore that ignores node_modules/ and .env
# 6. Make separate commits for each file using proper Conventional Commits
# 7. Run: git log --oneline
#    Expected: 3 commits with clean messages
```

### Lab 1B: Push to GitHub
```bash
# 1. Create a new PUBLIC repo on GitHub (no README — empty)
# 2. Link your local repo to it
git remote add origin https://github.com/YOURNAME/my-first-repo.git
git branch -M main
git push -u origin main

# 3. Verify: go to your GitHub repo URL and confirm files appear
# 4. Edit README.md on GitHub (pencil icon), commit there
# 5. Pull the change back locally
git pull
git log --oneline   # You should see the GitHub commit
```

**✅ Done when:** You have a repo on GitHub with 4+ commits, visible online.

---

## Exercise 02 — Branching

### Lab 2A: Feature Branch Workflow
```bash
# Setup: use the repo from Exercise 01

# 1. Create and switch to a feature branch
git switch -c feature/about-page

# 2. Create about.html with some content
# 3. Commit it: "feat: add about page"

# 4. Meanwhile, simulate main moving forward
git switch main
echo "v1.0" > version.txt
git add version.txt
git commit -m "chore: add version file"

# 5. Merge feature branch into main
git switch main
git merge --no-ff feature/about-page -m "merge: add about page"

# 6. View the graph
git log --oneline --graph --all
# You should see a merge commit with two parents

# 7. Delete the feature branch
git branch -d feature/about-page
```

### Lab 2B: Resolve a Conflict
```bash
# 1. Create branch-a and branch-b, both from main

git switch -c branch-a
# Edit line 1 of README.md to say "# Hello from Branch A"
git add README.md && git commit -m "docs: update title in branch-a"

git switch main
git switch -c branch-b
# Edit the SAME line 1 of README.md to say "# Hello from Branch B"
git add README.md && git commit -m "docs: update title in branch-b"

# 2. Merge branch-a into main
git switch main
git merge branch-a

# 3. Now try merging branch-b — CONFLICT!
git merge branch-b
# Fix the conflict in README.md
# Choose the final text, remove conflict markers
git add README.md
git commit -m "merge: resolve title conflict between branches"

# 4. Verify: git log --oneline --graph
```

**✅ Done when:** You've resolved a real conflict and have a clean merge commit.

---

## Exercise 03 — Collaboration (Requires a Partner or Use Your Own 2nd Account)

### Lab 3A: Pull Request Workflow
```bash
# 1. One person: create a repo, push to GitHub, invite a collaborator
#    (Settings → Collaborators → Add people)

# 2. Both clone the repo

# 3. Person A: create feature/navigation branch
git switch -c feature/navigation
# Add a nav.html file
git add . && git commit -m "feat: add navigation component"
git push -u origin feature/navigation

# 4. Person A: open a PR on GitHub
#    - Title: "feat: add navigation component"
#    - Fill in the PR template (What / Why / How to test)
#    - Request Person B as reviewer

# 5. Person B: review the PR on GitHub
#    - Leave at least 1 [blocking] and 1 [nit] comment
#    - Click "Request Changes"

# 6. Person A: address the feedback, push new commits to same branch
git add . && git commit -m "fix: address review feedback"
git push

# 7. Person B: re-review, approve, and merge
# 8. Both: pull main locally
git switch main && git pull
```

### Lab 3B: Setting Up Branch Protection
```bash
# On GitHub (Settings → Branches):
# 1. Add rule for 'main':
#    ✅ Require PR before merging
#    ✅ Require 1 approval
#    ✅ Require status checks (if CI is set up)
#    ✅ Do not allow force push

# 2. Try to push directly to main:
git switch main
echo "direct push" >> README.md
git add . && git commit -m "test: try direct push"
git push
# Expected: REJECTED — protection works!

# 3. Undo the commit
git reset --soft HEAD~1
git restore --staged README.md
git restore README.md
```

**✅ Done when:** A PR was opened, reviewed with feedback, updated, approved, and merged.

---

## Exercise 04 — Advanced Git

### Lab 4A: Interactive Rebase (Squash)
```bash
# 1. Make 4 messy commits on a feature branch
git switch -c feature/messy-work
echo "step 1" >> work.txt && git add . && git commit -m "wip"
echo "step 2" >> work.txt && git add . && git commit -m "more wip"
echo "step 3" >> work.txt && git add . && git commit -m "fix typo"
echo "step 4" >> work.txt && git add . && git commit -m "done for now"

# 2. Squash all 4 into 1 clean commit
git rebase -i HEAD~4
# In the editor: keep first as 'pick', change rest to 'f' (fixup)
# Save and close

# 3. Verify
git log --oneline   # Should show 1 clean commit
```

### Lab 4B: Cherry-Pick
```bash
# 1. Create two branches from main
git switch -c feature/big-feature
echo "big feature code" >> big.txt && git add . && git commit -m "feat: big feature start"
echo "important fix" >> fix.txt && git add . && git commit -m "fix: critical security patch"
echo "more big feature" >> big.txt && git add . && git commit -m "feat: big feature continued"

# 2. The security fix needs to go to main NOW, but big-feature isn't ready
git log --oneline feature/big-feature
# Copy the hash of the "fix: critical security patch" commit

git switch main
git cherry-pick <HASH-OF-SECURITY-FIX>
git log --oneline   # Fix is now on main!
```

### Lab 4C: Reflog Recovery
```bash
# 1. Make a commit
echo "important work" >> recovery-test.txt
git add . && git commit -m "feat: very important work"

# 2. Accidentally hard reset (simulate a mistake)
git reset --hard HEAD~1
# OH NO — the commit is "gone"!

# 3. Find it in reflog
git reflog
# Look for the commit with message "very important work"

# 4. Recover it
git reset --hard HEAD@{1}   # or the specific hash
# Your commit is back!
git log --oneline
```

**✅ Done when:** You've squashed commits, cherry-picked across branches, and recovered lost work.

---

## Exercise 05 — Production Workflow Simulation

### Lab 5A: Full GitFlow Simulation
```bash
# Setup: new repo with main and dev branches
git init gitflow-practice && cd gitflow-practice
git commit --allow-empty -m "chore: initial commit"
git checkout -b dev

# === Sprint Work ===
# Feature 1
git checkout -b feature/user-login
echo "login code" > login.js
git add . && git commit -m "feat(auth): add user login"
git checkout dev
git merge --no-ff feature/user-login -m "merge: user login"
git branch -d feature/user-login

# Feature 2
git checkout -b feature/dashboard
echo "dashboard code" > dashboard.js
git add . && git commit -m "feat(ui): add main dashboard"
git checkout dev
git merge --no-ff feature/dashboard -m "merge: dashboard"
git branch -d feature/dashboard

# === Release ===
git checkout -b release/v1.0.0
echo "1.0.0" > VERSION
git add . && git commit -m "chore: bump version to 1.0.0"

# Merge to main + tag
git checkout main
git merge --no-ff release/v1.0.0 -m "release: v1.0.0"
git tag -a v1.0.0 -m "Release v1.0.0"

# Backmerge to dev
git checkout dev
git merge --no-ff release/v1.0.0 -m "chore: backmerge v1.0.0 release"
git branch -d release/v1.0.0

# === Hotfix ===
git checkout main
git checkout -b hotfix/login-crash
echo "hotfix code" >> login.js
git add . && git commit -m "fix(auth): handle null session on login"
git checkout main
git merge --no-ff hotfix/login-crash -m "hotfix: login crash"
git tag -a v1.0.1 -m "Hotfix v1.0.1"
git checkout dev
git merge --no-ff hotfix/login-crash -m "chore: backmerge hotfix v1.0.1"
git branch -d hotfix/login-crash

# View the full picture
git log --oneline --graph --all
```

**✅ Done when:** You see a clean GitFlow graph with features, a release, and a hotfix.

---

## Exercise 06 — GitHub Actions

### Lab 6A: Your First CI Workflow
```bash
# 1. Create a simple Node.js project
mkdir ci-practice && cd ci-practice
git init
npm init -y
npm install --save-dev jest

# 2. Create a function and test
mkdir src && cat > src/math.js << 'EOF'
function add(a, b) { return a + b; }
module.exports = { add };
EOF

mkdir tests && cat > tests/math.test.js << 'EOF'
const { add } = require('../src/math');
test('adds 1 + 2 = 3', () => { expect(add(1, 2)).toBe(3); });
EOF

# Update package.json test script to: "test": "jest"
npm test   # Confirm it passes locally

# 3. Add .gitignore
echo "node_modules/" > .gitignore

# 4. Create the workflow
mkdir -p .github/workflows
```

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main, dev]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm test
```

```bash
# 5. Push to GitHub, open a PR, watch the Actions tab
git add .
git commit -m "ci: add GitHub Actions test workflow"
git push -u origin main
```

### Lab 6B: Break the Build (on Purpose)
```bash
# 1. Create a branch
git switch -c feat/broken-math

# 2. Intentionally break the function
# Change add to: return a - b;  (wrong!)
git add . && git commit -m "feat: update math function"
git push -u origin feat/broken-math

# 3. Open a PR — the CI will FAIL
# 4. Branch protection prevents merging a failing PR
# 5. Fix the function, push, watch CI go green
```

**✅ Done when:** You have a working CI pipeline that blocks a failing PR and allows a passing one.

---

## 🏆 Final Challenge — Full Team Simulation

This combines everything. Gather 2–3 people (or use multiple accounts):

```
Goal: Build a simple multi-page website using the full team workflow.

Roles:
- Tech Lead: manages dev and main, reviews PRs
- Developer A: builds "Home" page
- Developer B: builds "Contact" page

Steps:
1. Lead: create repo, set up main + dev, branch protection on both
2. Lead: create CODEOWNERS, PR template in .github/
3. Lead: set up GitHub Actions CI workflow
4. Dev A: branch feature/home-page from dev, build, open PR → dev
5. Dev B: branch feature/contact-page from dev, build, open PR → dev
6. Lead: review both PRs, request changes, approve and merge
7. Lead: open release/v1.0.0 from dev, merge to main, tag v1.0.0
8. Someone discovers a "bug" — create hotfix/typo-fix from main
9. Merge hotfix to main (v1.0.1) AND backmerge to dev
10. Review the full git log --oneline --graph --all
```

**✅ Done when:** You have a tagged v1.0.1 on main, a clean graph, and everyone practiced every role.
