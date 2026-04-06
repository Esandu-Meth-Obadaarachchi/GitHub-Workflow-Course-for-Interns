# Module 06 — DevOps & CI/CD with GitHub Actions
> 🔴 **Level:** Advanced | ⏱️ **Time:** 5–6 hours

---

## 🎯 Learning Objectives
- Understand CI/CD fundamentals
- Write GitHub Actions workflows from scratch
- Set up automated testing, linting, and building
- Deploy to production automatically
- Manage secrets and environments securely
- Use reusable workflows and composite actions

---

## 1. CI/CD Fundamentals

**CI (Continuous Integration):** Automatically test and build code when pushed.  
**CD (Continuous Delivery/Deployment):** Automatically deliver/deploy tested code.

```
Push code → CI triggers → Build → Test → Lint → (pass?) → Deploy
                                                 (fail?) → Notify + Block merge
```

### Why CI/CD?
- Catch bugs before they reach production
- Remove human error from deployments
- Ship faster with more confidence
- Every PR is automatically validated

---

## 2. GitHub Actions — Core Concepts

### Key terms

| Term | Definition |
|------|-----------|
| **Workflow** | An automated process defined in a YAML file |
| **Job** | A set of steps that run on the same runner |
| **Step** | A single task (run a command or use an action) |
| **Action** | A reusable unit of code (`uses: actions/checkout@v4`) |
| **Runner** | The machine that runs your jobs (GitHub-hosted or self-hosted) |
| **Event** | What triggers the workflow (push, PR, schedule, etc.) |

### File location
All workflows go in: `.github/workflows/*.yml`

---

## 3. Your First Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:                          # What triggers this workflow
  push:
    branches: [main, dev]
  pull_request:
    branches: [main, dev]

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest   # Runner type

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build
```

### Trigger events

```yaml
on:
  push:                       # On every push
    branches: [main]
    paths: ['src/**']         # Only if src/ files changed

  pull_request:               # On PR open/update
    branches: [dev, main]
    types: [opened, synchronize, reopened]

  schedule:                   # Cron job
    - cron: '0 9 * * 1'       # Every Monday at 9am UTC

  workflow_dispatch:          # Manual trigger from GitHub UI
    inputs:
      environment:
        description: 'Deploy to which environment?'
        required: true
        default: 'staging'

  release:                    # On GitHub release
    types: [published]
```

---

## 4. Secrets & Environment Variables

### Adding secrets to GitHub
**Settings → Secrets and variables → Actions → New repository secret**

```yaml
# Using secrets in workflows
steps:
  - name: Deploy to server
    env:
      API_KEY: ${{ secrets.API_KEY }}
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
    run: ./deploy.sh
```

### Environment-scoped secrets
```yaml
jobs:
  deploy:
    environment: production    # Uses secrets from "production" environment
    steps:
      - run: echo ${{ secrets.PROD_API_KEY }}
```

### Built-in variables
```yaml
${{ github.sha }}              # Commit SHA
${{ github.ref }}              # Branch ref (refs/heads/main)
${{ github.ref_name }}         # Branch name (main)
${{ github.actor }}            # Username who triggered
${{ github.repository }}       # owner/repo
${{ github.event_name }}       # push, pull_request, etc.
```

---

## 5. Real-World Workflow: Full CI Pipeline

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  pull_request:
    branches: [dev, main]

jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check

  test:
    name: Test
    runs-on: ubuntu-latest
    needs: lint                  # Only run after lint passes

    services:                    # Spin up a database for testing
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        ports: ['5432:5432']
        options: --health-cmd pg_isready --health-timeout 5s --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - name: Run tests with coverage
        run: npm test -- --coverage
        env:
          DATABASE_URL: postgresql://postgres:testpass@localhost:5432/testdb
      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  security:
    name: Security Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run npm audit
        run: npm audit --audit-level=high
      - name: Scan for secrets
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
```

---

## 6. Deployment Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches:
      - dev      # Deploy to staging when dev is updated
      - main     # Deploy to production when main is updated

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/dev'
    name: Deploy to Staging
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.yourapp.com

    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: |
          docker build -t yourapp:${{ github.sha }} .
          docker tag yourapp:${{ github.sha }} yourregistry/yourapp:staging

      - name: Push to registry
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push yourregistry/yourapp:staging

      - name: Deploy to server
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            docker pull yourregistry/yourapp:staging
            docker-compose -f /app/docker-compose.yml up -d --no-deps app

  deploy-production:
    if: github.ref == 'refs/heads/main'
    name: Deploy to Production
    runs-on: ubuntu-latest
    environment:
      name: production         # Requires approval from reviewers
      url: https://yourapp.com

    steps:
      - uses: actions/checkout@v4
      - name: Deploy to production
        run: ./scripts/deploy-prod.sh
        env:
          PROD_API_KEY: ${{ secrets.PROD_API_KEY }}
```

---

## 7. Matrix Builds — Test Multiple Versions

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: ['18', '20', '22']
      fail-fast: false          # Don't cancel all if one fails

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci
      - run: npm test
```

This runs 9 jobs (3 OS × 3 Node versions) in parallel.

---

## 8. Caching for Speed

```yaml
steps:
  - uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: 'npm'              # Built-in npm cache

  # Manual cache (for anything else)
  - uses: actions/cache@v4
    with:
      path: ~/.cache/pip
      key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
      restore-keys: |
        ${{ runner.os }}-pip-
```

Caching `node_modules` can reduce install time from 60s → 5s.

---

## 9. Reusable Workflows

```yaml
# .github/workflows/reusable-test.yml
name: Reusable Test Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: '20'
    secrets:
      CODECOV_TOKEN:
        required: true

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci && npm test
```

```yaml
# .github/workflows/ci.yml — call the reusable workflow
jobs:
  run-tests:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '20'
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

---

## 10. PR Automation

```yaml
# .github/workflows/pr-automation.yml
name: PR Automation

on:
  pull_request:
    types: [opened, ready_for_review]

jobs:
  auto-assign:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/auto-assign-action@v1
        with:
          configuration-path: .github/auto_assign.yml

  label-size:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/labeler@v5
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          configuration-path: .github/labeler.yml

  welcome-first-contribution:
    runs-on: ubuntu-latest
    if: github.event.action == 'opened'
    steps:
      - uses: actions/first-interaction@v1
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          pr-message: "🎉 Thanks for your first PR! A reviewer will look at this soon."
```

---

## 11. GitHub Actions Best Practices

```yaml
# ✅ Pin action versions to a full SHA (not just a tag)
uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4.2.2

# ✅ Use least-privilege permissions
permissions:
  contents: read
  pull-requests: write

# ✅ Timeout your jobs (prevent runaway billing)
jobs:
  test:
    timeout-minutes: 15

# ✅ Use concurrency to cancel redundant runs
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

# ✅ Never print secrets
run: |
  echo "Deploying..."           # ✅ fine
  echo "${{ secrets.KEY }}"     # ❌ NEVER — masks in logs but bad practice
```

---

## ✅ Module 06 Checklist
- [ ] Created a CI workflow that lints, tests, and builds on PRs
- [ ] Added a database service to a test job
- [ ] Created a deploy workflow targeting staging and production environments
- [ ] Used GitHub Secrets for sensitive values
- [ ] Set up a matrix build for multiple OS/versions
- [ ] Configured caching to speed up workflows
- [ ] Created a reusable workflow and called it from another workflow
- [ ] Added concurrency cancellation and job timeouts
- [ ] Set up a protected production environment with required reviewers

---

## 🎓 Course Complete!

You've covered:
- ✅ Git foundations and configuration
- ✅ Branching, merging, and conflict resolution
- ✅ Pull requests, code review, and open-source collaboration
- ✅ Advanced Git: rebase, cherry-pick, hooks, reflog
- ✅ Production branching strategies and release management
- ✅ CI/CD with GitHub Actions

**Suggested next steps:**
- Study GitHub's [Security Best Practices](https://docs.github.com/en/code-security)
- Explore [OpenSSF Scorecard](https://securityscorecards.dev/) for repo security
- Look into [GitHub Copilot](https://github.com/features/copilot) for AI pair programming
- Contribute to an open-source project using the fork workflow from Module 03

---

📋 [Cheatsheets](../cheatsheets/) | 🧪 [Exercises](../exercises/)
