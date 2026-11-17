# Vayva CI — Central CI Workflows

Reusable GitHub Actions workflow templates for all Vayva services and frontend apps. Eliminates CI workflow duplication across 28+ repositories. This repository contains no package.json — it is a workflow-template repository only.

## Architecture

Reusable workflows live in `.github/workflows/`:

- `.github/workflows/backend-service.yml` — Backend service CI (typecheck, test, build, publish)
- `.github/workflows/frontend-app.yml` — Frontend app CI (typecheck, test, build, publish)
- `.github/workflows/library-package.yml` — Shared package CI (typecheck, test, build, publish)
- `.github/workflows/staging-deploy.yml` — Staging deployment
- `.github/workflows/production-deploy.yml` — Production deployment

Example caller templates live in `templates/` — copy these into consuming repositories' `.github/workflows/` and customize. Caller templates reference the reusable workflows cross-repo via `Vayva-Tech/vayva-ci/.github/workflows/<name>.yml@v1` — the `vayva-ci` repository must carry a `v1` tag/branch for these references to resolve:

- `templates/backend-service-ci.yml`
- `templates/frontend-app-ci.yml`
- `templates/shared-package-ci.yml`
- `templates/staging-deploy-ci.yml`
- `templates/production-deploy-ci.yml`

## Image Publishing

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
    uses: ./.github/workflows/backend-service-ci.yml
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
    uses: ./.github/workflows/frontend-app-ci.yml
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
    uses: ./.github/workflows/shared-package-ci.yml
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## Workflow Templates

| Template | Used By | Features |
|----------|---------|----------|
| `backend-service-ci.yml` | 25 backend services | install, lint, typecheck, test, build, Docker build, **smoke test**, GHCR push, deploy |
| `frontend-app-ci.yml` | 4 frontend apps | quality job (lint/typecheck/test), shared package build, Next.js build, Docker build, GHCR push, deploy |
| `shared-package-ci.yml` | 16 shared packages | install, lint, typecheck, test, build, package verification |
| `staging-deploy-ci.yml` | repos with staging | staging deployment workflow |
| `production-deploy-ci.yml` | repos with deploy: true | production deployment workflow |

## Required Secrets

### All repos
| Secret | Scope | Purpose |
|--------|-------|---------|
| `NPM_TOKEN` | Org or per-repo | GitHub PAT with `read:packages` for @vayva-tech scope |

`NPM_TOKEN` is required by `backend-service.yml` and `frontend-app.yml` (declared `required: true`). `library-package.yml` declares it optional — when unset, private-registry auth is skipped (only works for packages whose dependencies resolve from public npm).

### Repos with deployment (`deploy: true`)
| Secret | Scope | Purpose |
|--------|-------|---------|
| `PROD_SSH_KEY` | Org or per-repo | SSH private key for production VPS |
| `PROD_SSH_HOST` | Org or per-repo | Production VPS hostname |
| `PROD_SSH_USER` | Org or per-repo | SSH user (e.g. `root`) |

**GHCR push** uses `github.token` (automatic) — no PAT needed.

## Inputs

### backend-service-ci.yml
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `service-name` | yes | — | Service name (e.g. `identity-service`) |
| `service-port` | yes | — | Port for container smoke test |
| `smoke-extra-env` | no | `''` | Additional env vars for smoke test |
| `deploy` | no | `false` | Deploy to production after push |

### frontend-app-ci.yml
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

## Verified Status (CHAT-1 Campaign 2026-01-15)

**Repository type:** Workflow templates only (no executable code)

**Files verified:**
- 5 reusable workflows in `.github/workflows/`
- 1 self-test workflow in `.github/workflows/` (`ci.yml` — validates this repository's own workflow files)
- 5 caller templates in `templates/`
- 1 composite action in `.github/actions/setup-workspace/`

**Structure:**
- backend-service.yml — Backend service CI (typecheck, test, build, publish)
- frontend-app.yml — Frontend app CI (typecheck, test, build, publish)
- library-package.yml — Shared package CI (typecheck, test, build, publish)
- staging-deploy.yml — Staging deployment
- production-deploy.yml — Production deployment
- ci.yml — Self-test workflow for this repository

**REPO-READY:** vayva-ci
