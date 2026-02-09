# 1. General

1. A work item is Done only when ALL applicable criteria in this section are met
1. Every team member is responsible for verifying their work against these criteria before marking a work item as complete
1. Work items that do not meet the Definition of Done MUST be returned to in-progress with specific gaps documented

# 2. Code Quality

1. All automated tests pass (unit, integration, end-to-end as applicable)
1. Code coverage meets threshold (see Section 6.3)
1. No unresolved critical or high severity issues linked to the work item
1. Linter passes with zero errors
1. No `TODO`, `FIXME`, or `HACK` comments without a linked tracking issue
1. Dead code and debug statements removed
1. Code follows all standards defined in Section 2

# 3. Testing and Validation

1. All acceptance criteria fulfilled and verified
1. Manual testing completed in test environment (for UI and user-facing changes)
1. Non-functional requirements tested:
   1. Performance (no significant regressions)
   1. Security (no new vulnerabilities introduced)
   1. Accessibility (WCAG AA compliance for UI changes)
1. Edge cases and error scenarios tested
1. Regression testing completed (existing functionality still works)

# 4. Documentation

1. API documentation updated (auto-generated from schema — see Section 1.9.1)
1. Supplementary documentation updated within 5 business days (see Section 1.9.1)
1. CHANGELOG entry added (see Section 11.3)
1. ADR created if the work item involved an architectural decision (see Section 1.10.4)
1. Code comments added for complex logic (explaining why, not what)
1. Configuration changes documented

# 5. CI/CD and Deployment

1. CI pipeline passes all stages
1. Deployment tested in staging environment (for production-bound changes)
1. Rollback plan documented (for production changes)
1. Database migrations tested and reversible (if applicable)
1. Feature flags configured correctly (if applicable)
1. Monitoring and alerting configured (for new services or critical paths)

# 6. Process

1. Linked to tracking issue or ticket
1. PR checklist completed (both author and reviewer sections)
1. Required approvals obtained
1. Semantic version determined and documented (if applicable)
1. Branch is up to date with target branch
1. Clean commit history (no WIP or fix-lint commits)

# 7. Exceptions

## 7.1. Hotfixes

1. MAY skip staging deployment with tech lead approval
1. MAY reduce review requirement with tech lead approval
1. MUST still have tests and pass CI
1. MUST document why the hotfix was needed
1. MUST create follow-up issues for any skipped criteria

## 7.2. Documentation-Only Changes

1. MAY skip code coverage requirement
1. MAY skip deployment testing
1. MUST verify rendered documentation
1. MUST verify all links resolve

## 7.3. Prototype and Spike Work

1. MAY skip production-readiness criteria (monitoring, rollback plan)
1. MAY skip comprehensive testing
1. MUST be marked as non-production (feature flag or separate environment)
1. MUST document outcomes and learnings
1. MUST create follow-up issues for production-readiness work

## 7.4. Dependency Updates

1. MAY skip new documentation if the change is transparent
1. MAY skip architecture documentation for minor/patch updates
1. MUST test for breaking changes
1. MUST verify all tests pass
1. MUST check for security vulnerabilities in the new version
