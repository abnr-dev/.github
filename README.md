# abnr-dev/.github — Shared CI/CD workflows

Central repository for ABNR DEV's reusable GitHub Actions workflows and org-wide
workflow templates. Owned by DevOps (Bruno).

## Reusable workflows (`.github/workflows/`)

### `node-ci.yml` — Node.js / TypeScript CI

Runs **install → prisma generate (if present) → lint → typecheck → test → build**
in a single job on `ubuntu-latest`.

Key behaviors:

- **Package manager auto-detection** from the lockfile: `pnpm-lock.yaml` → pnpm,
  `yarn.lock` → yarn, `package-lock.json` → `npm ci`. No lockfile → `npm install`
  (works, but commit a lockfile to get reproducible installs and dependency caching).
- **Missing scripts are skipped, not failed**: every step uses `npm run <script> --if-present`,
  so a repo without a `test` script still goes green.
- **Prisma support**: if `prisma/schema.prisma` exists, `npx prisma generate` runs
  before typecheck/build.
- **Dependency caching** via `actions/setup-node` whenever a lockfile is committed.

#### Usage

Create `.github/workflows/ci.yml` in your repo:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    uses: abnr-dev/.github/.github/workflows/node-ci.yml@main
```

Or pick "ABNR Node CI" from the **Actions → New workflow** page — the starter
template in [`workflow-templates/`](workflow-templates/) generates this file for you.

#### Inputs

| Input | Type | Default | Description |
|---|---|---|---|
| `node-version` | string | `"20"` | Node.js version |
| `working-directory` | string | `"."` | Directory containing `package.json` (monorepos/subfolders) |
| `run-lint` | boolean | `true` | Run `npm run lint` |
| `run-typecheck` | boolean | `true` | Run `npm run typecheck` |
| `run-tests` | boolean | `true` | Run `npm run test` |
| `run-build` | boolean | `true` | Run `npm run build` |
| `copy-env-example` | boolean | `false` | Copy `.env.example` → `.env` before the build (for builds that read env at build time) |

Example with overrides:

```yaml
jobs:
  ci:
    uses: abnr-dev/.github/.github/workflows/node-ci.yml@main
    with:
      node-version: "22"
      run-build: false          # build needs real secrets; deploy workflow covers it
      copy-env-example: true
```

#### Versioning

`@main` is the moving target every repo should normally track. If a breaking
change to the template is ever needed, it will be tagged (e.g. `@v1`) and
announced before merging to `main`.

## Conventions

- CI (this template) is a **gate**: it runs on every push and PR and must be green
  before deploy.
- Deploys stay in each repo's own `deploy.yml` (they need repo-specific secrets);
  CI is shared here.
- New language templates (PHP/Laravel, etc.) will be added here as
  `<stack>-ci.yml` following the same pattern.

## Pilot

First adopter: [`abnr-dev/effect-new`](https://github.com/abnr-dev/effect-new)
(`.github/workflows/ci.yml`).
