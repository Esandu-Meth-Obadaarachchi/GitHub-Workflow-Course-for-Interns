# ⚡ GitHub Actions Cheatsheet

## Workflow Skeleton
```yaml
name: Workflow Name
on: [push, pull_request]

jobs:
  job-name:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "hello"
```

## Triggers
```yaml
on:
  push:
    branches: [main, dev]
    paths: ['src/**']
  pull_request:
    branches: [main]
    types: [opened, synchronize]
  schedule:
    - cron: '0 9 * * 1'          # Mon 9am UTC
  workflow_dispatch:              # Manual trigger
  release:
    types: [published]
```

## Common Actions
```yaml
- uses: actions/checkout@v4
- uses: actions/setup-node@v4
  with: { node-version: '20', cache: 'npm' }
- uses: actions/setup-python@v5
  with: { python-version: '3.12' }
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
- uses: actions/upload-artifact@v4
  with: { name: build, path: dist/ }
- uses: actions/download-artifact@v4
  with: { name: build }
```

## Variables & Secrets
```yaml
env:
  NODE_ENV: test
  API_URL: ${{ secrets.API_URL }}

# Context variables
${{ github.sha }}
${{ github.ref_name }}
${{ github.actor }}
${{ github.repository }}
${{ github.event_name }}
```

## Job Control
```yaml
jobs:
  build:
    needs: [lint, test]          # Wait for these
    if: github.ref == 'refs/heads/main'
    timeout-minutes: 15
    continue-on-error: false
    environment: production       # Use env secrets
```

## Matrix
```yaml
strategy:
  matrix:
    node: ['18', '20', '22']
    os: [ubuntu-latest, windows-latest]
  fail-fast: false
runs-on: ${{ matrix.os }}
```

## Services (Databases)
```yaml
services:
  postgres:
    image: postgres:15
    env: { POSTGRES_PASSWORD: test }
    ports: ['5432:5432']
    options: --health-cmd pg_isready
```

## Best Practices
```yaml
# Cancel redundant runs
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

# Least privilege
permissions:
  contents: read
  pull-requests: write

# Pin to SHA
uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
```

## Reusable Workflow
```yaml
# Define
on:
  workflow_call:
    inputs:
      env: { type: string, required: true }
    secrets:
      TOKEN: { required: true }

# Call
jobs:
  deploy:
    uses: ./.github/workflows/deploy.yml
    with: { env: staging }
    secrets: { TOKEN: ${{ secrets.TOKEN }} }
```
