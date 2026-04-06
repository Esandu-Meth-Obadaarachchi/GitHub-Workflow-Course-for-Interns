# 🐙 GitHub: Zero to Production
### A Complete Course for Developers & Interns

> **Author:** Claude (Anthropic)  
> **Branch:** `claude/full-github-course` → `dev` → `main`  
> **Level:** Beginner to Advanced Production Workflows

---

## 📚 Course Modules

| # | Module | Level | Topics |
|---|--------|-------|--------|
| 01 | [Foundations](./module-01-foundations/README.md) | 🟢 Beginner | Git basics, repos, commits, config |
| 02 | [Branching & Merging](./module-02-branching/README.md) | 🟡 Intermediate | Branches, merges, conflicts, strategies |
| 03 | [Collaboration](./module-03-collaboration/README.md) | 🟡 Intermediate | PRs, code review, forks, issues |
| 04 | [Advanced Git](./module-04-advanced/README.md) | 🔴 Advanced | Rebase, cherry-pick, hooks, reflog |
| 05 | [Production Workflows](./module-05-production/README.md) | 🔴 Advanced | GitFlow, trunk-based, release strategies |
| 06 | [DevOps & CI/CD](./module-06-devops/README.md) | 🔴 Advanced | GitHub Actions, environments, secrets |

---

## 🗂️ Resources

- [`/exercises`](./exercises/) — Hands-on labs per module
- [`/cheatsheets`](./cheatsheets/) — Quick reference cards

---

## 🔀 Team Branching Convention (used in this course)

```
main          ← production, always stable
  └── dev     ← integration branch, tested before main
        └── claude/full-github-course   ← this course branch
        └── feature/your-feature
        └── fix/bug-name
```

### Workflow Rule
1. Always `git pull origin dev` before starting work
2. Create your branch off `dev`
3. Open PR: `your-branch` → `dev`
4. Lead reviews & merges to `dev`
5. When tested, lead merges `dev` → `main`

---

## ⚡ Quick Start

```bash
# Clone and set up
git clone <repo-url>
cd github-course

# Always sync with dev first!
git checkout dev
git pull origin dev

# Create your branch
git checkout -b yourname/feature-name

# After your work is done
git push origin yourname/feature-name
# Then open a Pull Request → dev
```
