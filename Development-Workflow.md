# 1. Branch Strategy

> Deferred. To be determined by ADR.

1. The `main` branch MUST be protected
1. Direct commits to `main` are PROHIBITED
1. All changes MUST enter `main` via pull request
1. The `main` branch MUST always be in a deployable state

# 2. Commit Message Conventions

1. All commit messages MUST follow the Conventional Commits specification (https://www.conventionalcommits.org/)
1. Format: `<type>[optional scope]: <description>`
1. Permitted types:

| Type | Purpose |
|------|---------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no logic change |
| `refactor` | Code change that neither fixes nor adds |
| `perf` | Performance improvement |
| `test` | Adding or correcting tests |
| `build` | Build system or external dependencies |
| `ci` | CI/CD configuration |
| `chore` | Other changes that don't modify src or test |
| `revert` | Reverts a previous commit |

1. Scope MUST identify the affected package (e.g., `feat(ui-components): add button variant`)
1. Breaking changes MUST include `BREAKING CHANGE:` in the commit footer or `!` after the type/scope (e.g., `feat(api)!: remove deprecated endpoint`)
1. Commit messages MUST be written in imperative mood (`add`, not `added` or `adds`)
1. Subject line MUST NOT exceed 72 characters
1. Body (when present) MUST be separated from subject by a blank line
> Review: Identify Requiremt and issue

# 3. Continuous Integration / Continuous Deployment (CI/CD)
## 3.1. Tooling

1. All CI/CD pipelines MUST use Dagger for orchestration
1. Dagger modules MUST be written in Go and stored in the `infrastructure/ci-cd/` directory
1. Application repositories MUST reference shared Dagger modules — pipeline duplication across packages is PROHIBITED
1. All pipelines MUST be locally testable via `dagger call` before committing
1. A thin CI platform wrapper (GitHub Actions, GitLab CI, etc.) is the only platform-specific configuration permitted

## 3.2. Pipeline Stages

1. The standard pipeline consists of five stages:

| Stage | Purpose | Trigger |
|-------|---------|---------|
| Lint | Static analysis, formatting, security scanning | Every commit |
| Test | Unit tests, integration tests, contract tests | Every commit |
| Build | Compile, package, create container images | Merge to main, version tags |
| Publish | Push artefacts to registries | Merge to main, version tags |
| Deploy | Apply to target environment | Per-environment triggers |

1. Lint and Test MUST run on every commit (PR and main branch)
1. Build and Publish MUST run on merge to main and on version tags
1. Deploy MUST be triggered per environment according to the promotion model
1. Pipeline MUST fail fast — subsequent stages MUST NOT run if a prior stage fails

## 3.3. Environment Promotion

1. A 4-environment promotion model MUST be used:

| Environment | Purpose | Data | Trigger |
|-------------|---------|------|---------|
| `dev` | Developer testing | Synthetic | On demand / PR |
| `test` | Automated testing | Synthetic | Merge to main |
| `staging` | Pre-production validation | Anonymised production copy | Manual promotion |
| `prod` | Production | Real | Manual promotion after staging verification |

1. Promotion flow: `dev → test → staging → prod`
1. Staging MUST mirror production (same Kubernetes version, same infrastructure components, same configuration — reduced scale is acceptable)
1. No environment MAY be skipped in the promotion flow except under a documented hotfix process

## 3.4. Affected Package Detection

1. CI MUST detect which packages are affected by a given change
1. Only affected packages MUST be built and tested — full monorepo rebuilds on every commit are PROHIBITED
1. A change to a shared package MUST trigger builds and tests for all packages that depend on it

## 3.5. Secrets in CI/CD

1. Secrets MUST NOT exist in code, configuration files, or CI/CD configuration
1. CI/CD secrets MUST be stored in the CI platform's secret store and injected as environment variables at runtime
1. Runtime secrets MUST be stored in AWS Secrets Manager (as defined in Section 1.4.2)
1. Applications MUST reference secrets by name — embedding secret values is PROHIBITED

## 3.6. Local Testing

1. Developers MUST be able to run any pipeline stage locally using `dagger call`
1. Local execution MUST produce identical results to CI execution
1. Pipeline changes MUST be tested locally before committing

## 3.7. Caching

1. Dagger's layer caching MUST be enabled
1. Unchanged steps MUST be skipped (not re-run)
1. Cache keys MUST be shared between local and CI execution where possible
