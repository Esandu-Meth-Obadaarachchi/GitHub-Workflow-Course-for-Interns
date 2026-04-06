# Module 05 — Production Workflows & Branching Strategies
> 🔴 **Level:** Advanced | ⏱️ **Time:** 4–5 hours

---

## 🎯 Learning Objectives
- Understand and implement GitFlow, GitHub Flow, and Trunk-Based Development
- Manage releases and hotfixes professionally
- Implement semantic versioning
- Use environment branches (dev, staging, production)
- Handle database migrations safely with Git
- Manage mono-repos vs multi-repos

---

## 1. Branching Strategies Compared

### GitFlow
Best for: projects with scheduled releases (mobile apps, versioned software)

```
main         ─────────────────────────────────────────  (production)
hotfix/       \──── fix ────/
release/             \─── prep ───/
dev          ──────────────────────────────────────────  (integration)
feature/       \─ A ─/  \─ B ─/  \─── C ───/
```

**Branches:**
| Branch | Purpose | Who merges here |
|--------|---------|-----------------|
| `main` | Production code, tagged releases | Release manager |
| `dev` | Integration, next release | PRs from features |
| `feature/*` | New features | Devs (PR to dev) |
| `release/*` | Pre-release stabilization | From dev, to main+dev |
| `hotfix/*` | Urgent production fixes | From main, back to main+dev |

### GitHub Flow
Best for: web apps with continuous deployment

```
main      ──────────────────────────────────────────── (always deployable)
feature/    \── A ──/  \── B ──/  \── C ──/
```

**Rules:**
1. `main` is always deployable
2. Branch from `main` for everything
3. Open a PR when ready to discuss/review
4. Deploy from your branch to test (optional)
5. Merge to `main` when reviewed and CI passes
6. Deploy immediately after merging

### Trunk-Based Development
Best for: high-velocity teams with strong CI/CD

```
main (trunk)  ── commit ── commit ── commit ── commit ──►
short-lived:        \─ fix (< 2 days) ─/
```

**Rules:**
- Everyone commits to `main` (or very short-lived branches < 2 days)
- Feature flags used to hide unfinished features
- Requires robust CI/CD and high test coverage
- Used by Google, Facebook, and top-tier engineering teams

### Which to choose?

| Factor | GitFlow | GitHub Flow | Trunk-Based |
|--------|---------|-------------|-------------|
| Release cadence | Scheduled | Continuous | Continuous |
| Team size | Medium-large | Any | Medium-large |
| CI/CD maturity | Low-medium | Medium | High |
| Complexity | High | Low | Low-medium |

**This course uses:** A simplified GitFlow (`main` → `dev` → `feature/*`) which works well for most teams.

---

## 2. Release Management

### Release branch workflow (GitFlow)
```bash
# Branch off dev when ready for release prep
git checkout dev
git pull origin dev
git checkout -b release/v2.3.0

# Bug fixes ONLY on release branch
git commit -m "fix: remove debug logging before release"

# When ready — merge to main AND back to dev
git checkout main
git merge --no-ff release/v2.3.0
git tag -a v2.3.0 -m "Release 2.3.0: invoice export + bug fixes"
git push origin main --tags

git checkout dev
git merge --no-ff release/v2.3.0
git push origin dev

git branch -d release/v2.3.0
git push origin --delete release/v2.3.0
```

### Hotfix workflow
```bash
# Critical bug in production!
git checkout main
git pull origin main
git checkout -b hotfix/payment-gateway-crash

# Fix it
git commit -m "fix(payments): handle Stripe API timeout with retry"

# Merge to main
git checkout main
git merge --no-ff hotfix/payment-gateway-crash
git tag -a v2.3.1 -m "Hotfix 2.3.1: payment gateway timeout"
git push origin main --tags

# ALSO merge to dev (don't lose the fix!)
git checkout dev
git merge --no-ff hotfix/payment-gateway-crash
git push origin dev

git branch -d hotfix/payment-gateway-crash
```

---

## 3. Semantic Versioning

Format: `MAJOR.MINOR.PATCH` (e.g., `v2.3.1`)

| Part | Increment when… | Example |
|------|-----------------|---------|
| MAJOR | Breaking change (API incompatible) | `v2.0.0` → `v3.0.0` |
| MINOR | New feature, backwards compatible | `v2.3.0` → `v2.4.0` |
| PATCH | Bug fix, backwards compatible | `v2.3.0` → `v2.3.1` |

Pre-release: `v2.4.0-beta.1`, `v2.4.0-rc.1`

### Automate with standard-version
```bash
npm install --save-dev standard-version

# In package.json scripts:
"release": "standard-version"

# Usage:
npm run release             # Auto-bump based on commits
npm run release -- --minor  # Force minor bump
npm run release -- --dry-run # Preview without committing
```

---

## 4. Environment Branch Strategy

Map branches to deployment environments:

```
main     → Production   (prod.yourapp.com)
staging  → Staging      (staging.yourapp.com)
dev      → Development  (dev.yourapp.com)
```

### Promotion flow
```
feature/* → dev → staging → main (production)
```

Each merge triggers a deploy to that environment (via CI/CD).

### Protecting environments
On GitHub: **Settings → Environments → New Environment**
- Add required reviewers for production
- Set deployment wait time (e.g., 10 min to notice issues)
- Add environment secrets (different API keys per environment)

---

## 5. Monorepos

A monorepo stores multiple projects/packages in one Git repository.

```
repo/
├── packages/
│   ├── api/              (backend)
│   ├── web/              (frontend)
│   ├── mobile/           (React Native app)
│   └── shared/           (shared utilities)
├── infra/
└── docs/
```

### Advantages
- Single source of truth
- Atomic cross-package changes in one commit
- Easier to share code and tooling

### Tools for monorepos
```bash
# Turborepo (recommended for JS/TS)
npx create-turbo@latest

# Nx
npx create-nx-workspace

# Lerna (older, still used)
npx lerna init
```

### Scoped commits in a monorepo
```bash
git commit -m "feat(api): add invoice endpoint"
git commit -m "fix(web): fix invoice table sorting"
git commit -m "chore(shared): upgrade lodash to 4.17.21"
```

---

## 6. Git Submodules (and When NOT to Use Them)

Submodules embed one repo inside another.

```bash
# Add a submodule
git submodule add https://github.com/org/lib.git libs/mylib

# Clone a repo WITH its submodules
git clone --recurse-submodules https://github.com/org/main-repo.git

# Update all submodules
git submodule update --init --recursive

# Pull latest of all submodules
git submodule update --remote
```

### ⚠️ Submodule gotchas
- They pin to a specific commit — you must manually update
- New team members often forget `--recurse-submodules`
- Complex to work with in CI

**Recommendation:** For most teams, use a package manager (npm, pip, etc.) or monorepo instead of submodules. Use submodules only when you need to embed a repo you don't own.

---

## 7. Handling Secrets in Git

### If you accidentally committed a secret

```bash
# Option 1: If not pushed yet
git reset --soft HEAD~1     # Undo commit
# Remove the secret, recommit

# Option 2: If already pushed — use BFG Repo Cleaner
# 1. Download bfg.jar from https://rtyley.github.io/bfg-repo-cleaner/
java -jar bfg.jar --replace-text secrets.txt my-repo.git
cd my-repo.git
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force

# ALWAYS rotate the exposed secret immediately!
```

### Prevention
```bash
# Use git-secrets to scan for credentials
brew install git-secrets
git secrets --install        # Install hooks in current repo
git secrets --register-aws   # Add AWS patterns
git secrets --scan           # Scan history
```

---

## 8. Code Freeze & Feature Flags

### Code freeze for releases
1. Create `release/vX.Y.Z` branch from `dev`
2. Announce freeze: no new features, bug fixes only
3. All fixes go to `release` branch, then backport to `dev`

### Feature flags (better than code freeze)
```javascript
// config/features.js
module.exports = {
  NEW_DASHBOARD: process.env.FEATURE_NEW_DASHBOARD === 'true',
  INVOICE_EXPORT: process.env.FEATURE_INVOICE_EXPORT === 'true',
};

// In your code
if (features.NEW_DASHBOARD) {
  return <NewDashboard />;
}
return <OldDashboard />;
```

Benefits:
- Merge incomplete features to `main` safely
- Enable/disable per environment
- Gradual rollout (% of users)
- Instant rollback without deploying

---

## ✅ Module 05 Checklist
- [ ] Set up a GitFlow structure (main, dev, feature branches)
- [ ] Created a release branch, tagged it, merged to main + dev
- [ ] Simulated a hotfix workflow
- [ ] Applied semantic versioning to releases
- [ ] Set up environment branches mapping to deploy environments
- [ ] Understood monorepo structure and tooling
- [ ] Scanned a repo for accidentally committed secrets

---

➡️ **Next:** [Module 06 — DevOps & CI/CD with GitHub Actions](../module-06-devops/README.md)
