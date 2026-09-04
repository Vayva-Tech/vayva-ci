# Vayva CI — Central CI Workflows

Reusable GitHub Actions workflows for all Vayva services and frontend apps. Eliminates CI workflow duplication across 28+ repositories.

## Architecture

All images push to **GHCR** (`ghcr.io/vayva-tech/`) using `github.token` for authentication (no PAT needed for push). Immutable tags: `sha-<commit>` + `sha-latest`.

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
      service-name: identity-service
      service-port: 4010
      deploy: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
    secrets: inherit
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
      app-name: vayva-merchant
      build-env: |
        NEXT_PUBLIC_APP_URL=http://localhost:3000
      deploy: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
    secrets: inherit
```

### Shared Package

```yaml
# .github/workflows/ci.yml
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

## Reusable Workflows

| Workflow | Used By | Features |
|----------|---------|----------|
| `backend-service.yml` | 25 backend services | install, lint, typecheck, test, build, Docker build, **smoke test**, GHCR push, deploy |
| `frontend-app.yml` | 4 frontend apps | quality job (lint/typecheck/test), shared package build, Next.js build, Docker build, GHCR push, deploy |
| `library-package.yml` | 16 shared packages | install, lint, typecheck, test, build, package verification |

## Required Secrets

### All repos
| Secret | Scope | Purpose |
|--------|-------|---------|
| `NPM_TOKEN` | Org or per-repo | GitHub PAT with `read:packages` for @vayva-tech scope |

### Repos with deployment (`deploy: true`)
| Secret | Scope | Purpose |
|--------|-------|---------|
| `PROD_SSH_KEY` | Org or per-repo | SSH private key for production VPS |
| `PROD_SSH_HOST` | Org or per-repo | Production VPS hostname |
| `PROD_SSH_USER` | Org or per-repo | SSH user (e.g. `root`) |

**GHCR push** uses `github.token` (automatic) — no PAT needed.

## Inputs

### backend-service.yml
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `service-name` | yes | — | Service name (e.g. `identity-service`) |
| `service-port` | yes | — | Port for container smoke test |
| `smoke-extra-env` | no | `''` | Additional env vars for smoke test |
| `deploy` | no | `false` | Deploy to production after push |

### frontend-app.yml
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `app-name` | yes | — | App directory name (e.g. `vayva-merchant`) |
| `build-env` | no | `''` | Build-time env vars (KEY=VALUE per line) |
| `deploy` | no | `false` | Deploy to production after push |

## Standards

- Node.js 22, pnpm 11.25.0 (pinned via corepack)
- Docker build with BuildKit secret mounts (no token in image layers)
- Immutable image tags: `sha-<40-char-commit>` + `sha-latest`
- GHCR registry: `ghcr.io/vayva-tech/<name>`
- Registry push on main branch push only
- Container smoke test validates boot health before push
- Runner disk guard prunes images when disk >80%
