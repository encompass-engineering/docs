# 1. Release Steps

1. Determine version bump by reviewing changes since last release against SemVer rules (Section 11.1)
1. Update version in the package's manifest file
1. Update CHANGELOG (move `[Unreleased]` entries to the new version section with date)
1. Create release PR with version changes
1. Obtain required approvals
1. Merge release PR
1. Create and push annotated git tag (`git tag -a vX.Y.Z -m "Release version X.Y.Z"`)
1. CI automatically publishes artefacts on tag push

# 2. Artefact Publishing

1. CI MUST automatically publish artefacts when a version tag is pushed
1. Publishing per artefact type:
   1. npm packages → private npm registry
   1. Python packages → private PyPI server
   1. Rust crates → crates.io or private registry
   1. Container images → OCI-compliant registry
   1. Helm charts → OCI chart registry
1. Published artefacts MUST be verified (smoke test, registry confirmation)

# 3. Hotfix Process

1. Create hotfix branch from the latest release tag
1. Fix the bug and add tests
1. Bump PATCH version (hotfixes are always PATCH unless they introduce breaking changes)
1. Fast-track review with tech lead approval
1. Follow standard release steps (12.1, steps 5-8)
1. Backport to main if the fix is not already present

# 4. Deployment

## 4.1. Internal Deployment

1. Direct deployment to internal infrastructure via CI/CD
1. Continuous delivery: every merge to main deploys to test environment
1. Manual promotion to staging and production

## 4.2. External / On-Premise Deployment

1. Container images pushed to OCI-compliant registry
1. Helm charts packaged and versioned separately
1. Clients pull images and Helm charts from the registry
1. Clients deploy using Helm with custom `values.yaml`
1. Helm charts MUST NOT contain secret values — only references to Kubernetes Secrets
1. Clients are responsible for creating and rotating their own secrets

## 4.3. Helm Chart Standards

1. Chart versioning: Semantic Versioning for chart changes
1. App versioning: tracks application code releases
1. Charts distributed via OCI registry
1. Client-overridable values:
   1. Image registry and tag
   1. Resource limits (CPU, memory)
   1. Environment-specific configuration (database endpoints, log levels)
   1. Secret references (never secret values)

# 5. Rollback

1. Every production deployment MUST have a documented rollback plan
1. Rollback MUST be executable within 15 minutes
   > Review: confirm rollback time target
1. Database migrations MUST be reversible
1. Rollback criteria MUST be defined before deployment (what conditions trigger a rollback)

# 6. Post-Release

1. Published artefacts MUST be verified in the target environment
1. Monitoring MUST confirm no anomalies after deployment
1. Stakeholders MUST be notified of the release (changelog excerpt, impacted systems)
