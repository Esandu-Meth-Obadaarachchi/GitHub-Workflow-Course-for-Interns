# Module 03 — Collaboration on GitHub
> 🟡 **Level:** Intermediate | ⏱️ **Time:** 4–5 hours

---

## 🎯 Learning Objectives
- Open, review, and merge Pull Requests
- Write useful code reviews
- Use Issues, Labels, and Milestones
- Fork and contribute to open-source projects
- Protect branches with rules
- Use GitHub Discussions and Project boards

---

## 1. Pull Requests (PRs)

A Pull Request is a proposal to merge code from one branch into another. It's also a conversation and code review tool.

### PR Lifecycle
```
1. Push your branch → 2. Open PR → 3. Review & discussion
→ 4. CI/tests pass → 5. Approval → 6. Merge → 7. Delete branch
```

### Opening a PR on GitHub
1. Push your branch: `git push -u origin feature/my-feature`
2. GitHub shows a banner: **"Compare & pull request"** — click it
3. Fill in:
   - **Title** — use Conventional Commits style: `feat: add invoice export`
   - **Description** — explain what and why (see template below)
   - **Reviewers** — tag teammates
   - **Labels** — `feature`, `bug`, `needs-review`, etc.
   - **Linked Issue** — `Closes #42`
4. Click **Create Pull Request**

### PR Description Template
```markdown
## What does this PR do?
Brief explanation of the change.

## Why is it needed?
Context or problem being solved.

## How to test?
1. Step one
2. Step two
3. Expected result

## Screenshots (if UI change)
<!-- Paste before/after screenshots -->

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No secrets or credentials in code
- [ ] Reviewed my own diff before requesting review

Closes #<issue-number>
```

---

## 2. Code Review Best Practices

### As the Reviewer
- **Review the diff, not the author** — keep it professional and kind
- **Be specific** — don't just say "bad"; explain what and why
- **Ask questions** — `"Could we use a Map here for O(1) lookup?"`
- **Distinguish blocking vs non-blocking comments:**
  - 🔴 `[blocking]` — must fix before merge
  - 🟡 `[suggestion]` — consider it, but optional
  - 🟢 `[nit]` — tiny style thing, totally optional
- **Approve or Request Changes** — don't leave PRs in limbo

### Review Comment Examples
```
# ❌ Unhelpful
"This is wrong."

# ✅ Helpful
"[blocking] This loop runs O(n²) because Array.includes() is O(n). 
 Consider using a Set for O(1) membership checks:
 const validIds = new Set(items.map(i => i.id));
 if (validIds.has(userId)) { ... }"

# ✅ Suggesting (non-blocking)
"[nit] Minor: the variable name `d` could be `dueDate` for readability."
```

### As the Author
- Keep PRs **small** — aim for < 400 lines changed
- Self-review your own diff before requesting review
- Respond to every comment (even if just "Done ✅" or "Disagree because…")
- Don't take feedback personally

---

## 3. GitHub Issues

Issues track bugs, features, tasks, and questions.

### Creating a good Issue

```markdown
## Bug Report

**Describe the bug**
The login button does nothing when the email field is empty.

**To Reproduce**
1. Go to /login
2. Enter password, leave email blank
3. Click "Log In"
4. Expected: validation error. Actual: nothing happens.

**Environment**
- OS: macOS 14
- Browser: Chrome 122
- App version: 2.4.1

**Screenshots**
[Attach here]
```

### Labels to use
| Label | Purpose |
|-------|---------|
| `bug` | Something broken |
| `feature` | New functionality |
| `enhancement` | Improvement to existing feature |
| `documentation` | Docs needed |
| `good first issue` | Suitable for new contributors |
| `blocked` | Waiting on something |
| `priority: high` | Urgent |

### Link Issues to PRs
```bash
# In a PR description or commit message:
Closes #42       # Closes issue when PR merges
Fixes #15        # Same
Resolves #8      # Same
Relates to #20   # Reference without auto-closing
```

---

## 4. Forking & Open-Source Contribution

**Fork** = your personal copy of someone else's repo on GitHub.

### Fork workflow
```bash
# 1. Fork on GitHub (click Fork button)

# 2. Clone YOUR fork
git clone https://github.com/YOURNAME/repo.git
cd repo

# 3. Add the original repo as "upstream"
git remote add upstream https://github.com/ORIGINAL/repo.git

# 4. Verify remotes
git remote -v
# origin    https://github.com/YOURNAME/repo.git (your fork)
# upstream  https://github.com/ORIGINAL/repo.git (original)

# 5. Keep your fork up to date
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# 6. Create feature branch, do work, push to YOUR fork
git checkout -b fix/typo-readme
git push origin fix/typo-readme

# 7. Open PR from your fork to the original repo on GitHub
```

---

## 5. Branch Protection Rules

Protect your `main` and `dev` branches from accidental pushes.

### Set up on GitHub:
**Settings → Branches → Add branch protection rule**

Recommended rules for `main`:
- ✅ **Require pull request reviews before merging** (min. 1 reviewer)
- ✅ **Require status checks to pass before merging** (CI must pass)
- ✅ **Require branches to be up to date before merging**
- ✅ **Do not allow bypassing the above settings**
- ✅ **Restrict who can push to matching branches** (only team leads)

Recommended rules for `dev`:
- ✅ Require pull request reviews (1 reviewer)
- ✅ Require CI to pass
- ❌ Don't restrict push as strictly as `main`

---

## 6. GitHub Project Boards & Milestones

### Milestones
Group issues and PRs into a deliverable (sprint, release, etc.):
- Go to **Issues → Milestones → New Milestone**
- Set a due date (e.g., "Sprint 12 — June 30")
- Assign issues to it

### Project Boards (GitHub Projects)
Kanban-style board for your team:

```
Backlog → In Progress → In Review → Done
```

- Create at **Projects → New Project**
- Use automation: auto-move issues when PR opens/closes

---

## 7. CODEOWNERS File

Automatically assign reviewers based on file paths:

```
# .github/CODEOWNERS

# Global fallback reviewer
*                @team-lead

# Frontend files
/src/frontend/   @alice @bob

# Backend
/src/api/        @charlie

# Infrastructure
/infra/          @devops-team

# Documentation
/docs/           @tech-writer
```

Place at `.github/CODEOWNERS`. GitHub automatically requests reviews from these people when their files change.

---

## 8. GitHub Notifications & Communication

```bash
# Use @mentions in comments to notify people
"@alice can you take a look at the auth logic here?"

# Reference issues/PRs across repos
"This is blocked by org/other-repo#42"

# Mark as draft PR when not ready for review
# On GitHub: Create Pull Request → dropdown → "Create Draft Pull Request"
```

---

## ✅ Module 03 Checklist
- [ ] Opened a PR using the description template
- [ ] Left a constructive code review (blocking + non-blocking comments)
- [ ] Linked a PR to an Issue (Closes #N)
- [ ] Forked a repo and submitted a PR to the original
- [ ] Set up branch protection rules on `main`
- [ ] Created a CODEOWNERS file
- [ ] Used a Project board to organize issues

---

➡️ **Next:** [Module 04 — Advanced Git](../module-04-advanced/README.md)
