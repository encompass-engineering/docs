
# 11. Versions

## 11.1. Semantic Versioning

1. All packages and services MUST use Semantic Versioning: `MAJOR.MINOR.PATCH`
1. Version increment rules:

| Component | When to Increment |
|-----------|-------------------|
| `MAJOR` | Breaking changes requiring consumer action |
| `MINOR` | New backward-compatible features |
| `PATCH` | Backward-compatible bug fixes |

## 11.2. Breaking Change Definition

1. A breaking change is any modification that could cause existing consumers to fail or behave incorrectly without code changes
1. API/interface breaking changes:
   1. Endpoint removed, renamed, or path changed
   1. Required field added to request
   1. Field removed from response or type changed
   1. HTTP status code changed for existing scenario
   1. Query parameter made required or removed
1. Configuration breaking changes:
   1. Required configuration property added
   1. Configuration property removed or renamed
   1. Default value changed incompatibly
   1. Environment variable renamed or removed
1. Data/schema breaking changes:
   1. Incompatible database migration
   1. Data format changed
   1. File format version changed incompatibly
1. Behavioural breaking changes:
   1. Observable behaviour changed (sorting order, validation rules)
   1. Error handling changed (new exceptions thrown)
   1. Performance characteristics significantly degraded

## 11.3. CHANGELOG

1. Every package MUST maintain a `CHANGELOG.md` following the Keep a Changelog format (https://keepachangelog.com/)
1. Categories: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`
1. CHANGELOG MUST be updated in the same PR as the change
1. Each entry MUST include an issue or PR reference for traceability
1. Breaking changes MUST be prefixed with `(BREAKING)`
1. The `[Unreleased]` section MUST be maintained for in-progress changes

## 11.4. Pre-release Versions

1. Format: `X.Y.Z-<label>.<number>` (e.g., `1.0.0-alpha.1`, `1.0.0-beta.2`, `1.0.0-rc.1`)
1. Labels:
   1. `alpha` — early testing, unstable API
   1. `beta` — feature complete, stabilising
   1. `rc` — release candidate, final testing before production
1. Pre-release versions MUST be published to pre-release channels (npm `--tag beta`, Docker `-beta` suffix)

## 11.5. Version Sources

1. Each artefact type stores its version in a specific location:

| Artefact Type | Version Source |
|---------------|----------------|
| npm packages | `package.json` `version` field |
| Python packages | `pyproject.toml` `version` field |
| Rust crates | `Cargo.toml` `version` field |
| OCaml packages | `dune-project` `version` field |
| Helm charts | `Chart.yaml` `version` field |
| Container images | Git tag |

1. When a package directory contains multiple artefact types, versions MUST be kept aligned

## 11.6. Monorepo Versioning

1. Each package within the monorepo MUST be versioned independently
1. Package versions MUST NOT correlate to other package versions — they reflect the changes within that package only
1. A change to a shared package MUST trigger a version bump in that package, but MUST NOT force version bumps in consuming packages
1. Consuming packages update their dependency version at their own cadence via standard dependency update processes
