# Vayva CI — Central CI Workflows

Reusable GitHub Actions workflows and composite actions for all Vayva services and frontend apps. Eliminates CI workflow duplication across 28+ repositories.

## Usage

### Backend Service

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: Vayva-Tech/vayva-ci/.github/workflows/backend-service.yml@main
    with:
      service-name: identity-service
      service-port: 4010
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
      REGISTRY_URL: ${{ secrets.REGISTRY_URL }}
      REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
      REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
```

### Frontend App

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: Vayva-Tech/vayva-ci/.github/workflows/frontend-app.yml@main
    with:
      app-name: vayva-merchant
      build-env: |
        NEXT_PUBLIC_APP_URL=http://localhost:3000
        NEXT_PUBLIC_IDENTITY_SERVICE_URL=http://localhost:4010
    secrets:
      CROSS_REPO_TOKEN: ${{ secrets.CROSS_REPO_TOKEN }}
      REGISTRY_URL: ${{ secrets.REGISTRY_URL }}
      REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
      REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
```

## What's Included

### Composite Actions
- **setup-workspace** — Clones 13 shared packages, configures npm registry, generates pnpm-workspace.yaml

### Reusable Workflows
- **backend-service.yml** — Full CI: install → lint → typecheck → test → build → Docker → smoke test → push
- **frontend-app.yml** — Full CI: quality checks → Next.js build → Docker → push

## Required GitHub Secrets (per repo)

### Backend services
| Secret | Purpose |
|--------|---------|
| `NPM_TOKEN` | GitHub PAT with read:packages for @vayva-tech scope |
| `REGISTRY_URL` | Docker registry URL (e.g. 13.140.159.213:5000) |
| `REGISTRY_USERNAME` | Registry username |
| `REGISTRY_PASSWORD` | Registry password |

### Frontend apps
| Secret | Purpose |
|--------|---------|
| `CROSS_REPO_TOKEN` | GitHub PAT with repo + read:packages (for git clone, npm auth, Docker build-arg) |
| `REGISTRY_URL` | Docker registry URL (e.g. 13.140.159.213:5000) |
| `REGISTRY_USERNAME` | Registry username |
| `REGISTRY_PASSWORD` | Registry password |

## Standards

- Node.js 22
- pnpm 11.25.0 (pinned via corepack)
- Docker build with BuildKit secret mounts (no token in image layers)
- Immutable image tags: `sha-{commit}` + `sha-latest`
- Registry push on main branch only

## Migrated Repos (29 total)

### Backend services (25) — `backend-service.yml`
identity-service, business-service, commerce-service, wallet-service, customer-service, storefront-service, messaging-service, marketing-service, support-service, analytics-service, billing-service, delivery-service, ai-service, connector-service, verification-service, notification-service, onboarding-service, storage-service, ops-service, industry-service, import-service, affiliate-service, workflow-service, personalization-service, semantic-layer-service

### Frontend apps (4) — `frontend-app.yml`
vayva-merchant, vayva-ops, vayva-marketing, vayva-storefront
