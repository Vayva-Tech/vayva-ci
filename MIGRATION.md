# Migration Guide

How to migrate each repo type from inline CI to the central reusable workflows.

## What Changes

- **Before:** 100-150 lines of CI logic duplicated in every repo
- **After:** ~25-line wrapper that calls the central workflow
- **Registry:** Self-hosted (`13.140.159.213:5000`) → GHCR (`ghcr.io/vayva-tech/`)
- **Auth:** `REGISTRY_PASSWORD` → `github.token` (automatic for GHCR push)
- **Deploy:** Added optional SSH deploy step (controlled by `deploy` input)

## Backend Services (25 repos)

Replace `.github/workflows/ci.yml` with:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read
  packages: write

jobs:
  ci:
    uses: Vayva-Tech/vayva-ci/.github/workflows/backend-service.yml@main
    with:
      service-name: __SERVICE_NAME__
      service-port: __PORT__
      deploy: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
    secrets: inherit
```

Replace `__SERVICE_NAME__` and `__PORT__` per service:

| Service | Port |
|---------|------|
| identity-service | 4010 |
| commerce-service | 4011 |
| billing-service | 4012 |
| wallet-service | 4013 |
| customer-service | 4014 |
| analytics-service | 4015 |
| ai-service | 4016 |
| delivery-service | 4017 |
| search-service | 4018 |
| storage-service | 4019 |
| notification-service | 4020 |
| messaging-service | 4021 |
| onboarding-service | 4022 |
| ops-service | 4023 |
| storefront-service | 4024 |
| support-service | 4025 |
| verification-service | 4026 |
| worker-service | 4027 |
| workflow-service | 4028 |
| marketing-service | 4029 |
| connector-service | 4030 |
| import-service | 4031 |
| industry-service | 4032 |
| business-service | 4033 |
| mcp-service | 4034 |

## Frontend Apps (4 repos)

Replace `.github/workflows/ci.yml` with:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read
  packages: write

jobs:
  ci:
    uses: Vayva-Tech/vayva-ci/.github/workflows/frontend-app.yml@main
    with:
      app-name: __APP_DIR__
      build-env: |
        NEXT_PUBLIC_APP_URL=http://localhost:3000
      deploy: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
    secrets: inherit
```

| App | `app-name` |
|-----|------------|
| vayva-merchant | vayva-merchant |
| vayva-ops | vayva-ops |
| vayva-marketing | vayva-marketing |
| vayva-storefront | vayva-storefront |

## Shared Packages (16 repos)

Replace `.github/workflows/ci.yml` with:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    uses: Vayva-Tech/vayva-ci/.github/workflows/library-package.yml@main
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## Required Secrets

After migration, each repo needs these secrets (or set them at org level):

| Secret | Purpose | Required By |
|--------|---------|-------------|
| `NPM_TOKEN` | PAT with `read:packages` | All repos |
| `PROD_SSH_KEY` | SSH key for production VPS | Repos with `deploy: true` |
| `PROD_SSH_HOST` | Production VPS hostname | Repos with `deploy: true` |
| `PROD_SSH_USER` | SSH user | Repos with `deploy: true` |

**Removed secrets** (no longer needed):
- `REGISTRY_URL` — GHCR is hardcoded
- `REGISTRY_USERNAME` — `github.actor` used instead
- `REGISTRY_PASSWORD` — `github.token` used instead
- `CROSS_REPO_TOKEN` — `NPM_TOKEN` used for both npm and git clone

## Verification

After migration, verify:
1. PR triggers CI run (lint, typecheck, test, build)
2. Merge to main triggers Docker build + smoke test + GHCR push
3. Image appears at `ghcr.io/vayva-tech/<name>:sha-<commit>`
4. If `deploy: true`, SSH deploy runs after push

## Rollback

To revert, restore the previous `ci.yml` from git history:
```bash
git log --oneline .github/workflows/ci.yml
git checkout <commit-before-migration> -- .github/workflows/ci.yml
```
