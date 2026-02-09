# 1. Architecture

1. All projects MUST use a monorepo architecture
1. Each service, package, and library within the monorepo MUST be independently versioned
1. Each independently versioned unit MUST have its own build, test, and publish pipeline
1. Dependency direction MUST flow downward: applications → shared packages → infrastructure utilities
1. Bidirectional dependencies between packages are PROHIBITED (as defined in Section 1.3.3)

# 2. Directory Layout

1. The monorepo MUST have a clear top-level organisation that separates concerns (e.g., applications from shared packages from infrastructure)
1. The specific top-level directory structure is a project-level decision — it MUST be documented in the project's root README
1. Each independently versioned unit MUST reside in its own subdirectory
1. Directory names MUST use kebab-case
1. Each package directory MUST contain its own language-appropriate manifest file (`package.json`, `Cargo.toml`, `pyproject.toml`, `dune-project`, `.cabal`)

# 3. Naming Conventions

1. Directory names: kebab-case
1. Package naming MUST follow the language-idiomatic convention:

| Language | Pattern | Example |
|----------|---------|---------|
| npm (TypeScript) | `@{org}/{package-name}` | `@acme/ui-components` |
| Python | `{org}_{package_name}` | `acme_data_processing` |
| Rust | `{org}_{crate_name}` | `acme_math_core` |
| OCaml | `{org}-{package-name}` | `acme-parser` |

1. The organisation prefix (`{org}`) is a project-level decision — it MUST be consistent across all packages within the monorepo

# 4. Package Independence

1. Each package MUST declare its own dependencies explicitly — implicit resolution through the monorepo root is PROHIBITED
1. Each package MUST be buildable and testable in isolation
1. Shared packages MUST be consumed via the package manager (npm, cargo, pip), not via relative filesystem imports across package boundaries
1. A change to package A MUST NOT require rebuilding package B unless B declares a dependency on A

# 5. Package Creation Checklist

1. When adding a new package or application to the monorepo, the following MUST be completed:
   1. Directory created under the appropriate location per the project's documented layout
   1. Language-appropriate manifest file with name and version
   1. Filepath comment convention applied to all source files
   1. CODEOWNERS entry added for the new directory
   1. CI/CD pipeline configured (referencing shared Dagger modules)
   1. README with purpose, quick start, and dependency information
   1. CHANGELOG initialised
   1. Package registered in the monorepo workspace configuration
