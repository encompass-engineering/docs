
# 1. Coding Standards

## 1.1. General Standards

### 1.1.1. Code Structure

1. Max nesting depth: 3
1. Max columns: 80
1. Max lines per function: 40
1. Cyclomatic complexity: 10
1. Indentation: 4 spaces — tabs are PROHIBITED
1. First line of every file MUST be a comment containing the file path relative to project root:
   1. TypeScript/JavaScript: `// src/components/HexagonGrid/HexagonGrid.tsx`
   1. Rust: `// src/services/auth/handler.rs`
   1. OCaml: `(* src/lib/parser/lexer.ml *)`
   1. Haskell: `-- src/Lib/Parser/Lexer.hs`
   1. Python: `# src/tests/test_auth.py`
   1. C#: `// src/Services/Auth/Handler.cs`

### 1.1.2. Code Style (Universal)

1. Constants: `SCREAMING_SNAKE_CASE` (unless the language idiom dictates otherwise — see per-language sections)
1. All other naming conventions defer to the language-idiomatic standard as defined in the per-language sections below
1. The name MUST convey what the entity is and how it behaves — if additional annotation or documentation is needed to understand a name, the name is wrong

### 1.1.3. Dead Code

1. Dead code is PROHIBITED on the main branch
1. Dead code MUST be resolved before any PR is merged
1. CI MUST check for dead code on all PRs
1. Dead code on feature branches is permitted during development

### 1.1.4. Error Handling

1. Exceptions as control flow are PROHIBITED where the language provides alternatives
1. Errors MUST be treated as values:
   1. Rust: `Result<T, E>` and `Option<T>`
   1. OCaml: `Result` and `Option` types
   1. Haskell: `Either` and `Maybe` types
   1. TypeScript: explicit error returns or discriminated unions — `try/catch` only at the boundary for truly exceptional cases (network failures, system errors)
   1. Python: relaxed (prototype-only language)
   1. C#: accepted as language convention — `try/catch` permitted but should be minimised
1. Validate at the boundary, reject invalid input — never sanitise and pass through
1. Overly broad catch blocks (catching `Exception`, `Error`, `_` without specific handling) are PROHIBITED

### 1.1.5. Composition

1. Functions, modules, and components MUST compose — accumulation of logic in a single unit is PROHIBITED
1. If a function exceeds the line limit, it MUST be decomposed into smaller composable functions
1. If a module accumulates unrelated responsibilities, it MUST be split

## 1.2. Language and Framework Specific Standards

### 1.2.1. TypeScript

#### 1.2.1.1. Naming

| Entity | Convention | Example |
|--------|-----------|---------|
| Interface | `I`-prefix + `PascalCase` | `IUserService` |
| Type alias | `T`-prefix + `PascalCase` | `TVariant` |
| Type parameter | `T`-prefix + `PascalCase` | `TData` |
| Constant | `SCREAMING_SNAKE_CASE` | `MAX_GRID_SIZE` |
| Variable | `snake_case` | `hex_angle` |
| Function | `camelCase` | `calculateAngle` |
| Component | `PascalCase` | `HexagonGrid` |

#### 1.2.1.2. Naming Edge Cases

1. `snake_case` variable exceptions (NOT violations):
   1. Destructured hook returns: `const [count, setCount] = useState(0)` — `camelCase` from React API
   1. Destructured props: `const { className, onClick } = props` — `camelCase` from interface definition
   1. React refs: `const inputRef = useRef(null)` — `camelCase` convention
   1. Event handler parameters: `(event) => ...` — `camelCase` from DOM API
   1. Loop iterators: `items.map((item) => ...)` — `camelCase` from external API
   1. React built-in types: `CSSProperties`, `ReactNode`, `Ref`, `HTMLAttributes` — do NOT add `I`-prefix to imported React types
1. `snake_case` applies to: locally declared utility variables, intermediate calculation results, configuration objects
1. `SCREAMING_SNAKE_CASE` enforcement scope: `.consts.ts` files. Constants declared inline in `.tsx` files follow function-scoped convention
1. Component vs function distinction: functions returning JSX use `PascalCase`, all other functions use `camelCase`

#### 1.2.1.3. Exports

1. Named exports are the default — `export default` is PROHIBITED except when required for lazy loading (`React.lazy` requires a default export)
1. Components intended for lazy loading MUST use `export default` alongside the named export:
   ```typescript
   export { HexagonGrid };
   export default HexagonGrid;
   export type { IHexagonGridProps };
   ```
1. Export block MUST be at the bottom of the file
1. Scattered exports throughout the file are PROHIBITED

#### 1.2.1.4. Semicolons

1. Semicolons: always

### 1.2.2. React

#### 1.2.2.1. Component Structure

1. Components MUST use `React.FC<IProps>` typing pattern
1. Component prop interfaces MUST use the convention `I<ComponentName>Props` (e.g., `IHexagonGridProps`)
1. Component returns MUST be composed components, not raw HTML element trees
1. JSX blocks exceeding 20 lines MUST be decomposed into sub-components
1. Raw HTML elements (`<div>`, `<span>`, `<p>`) are permitted only in leaf components — higher-level components MUST compose named components
1. Props MUST be spread for clean JSX:
   ```tsx
   return (<SomeComp {...props}><AnotherComp {...other_props} /></SomeComp>);
   ```

#### 1.2.2.2. Hooks

1. All hooks MUST be externalised — no inline stateful logic in components
1. Stateful logic exceeding 10 lines MUST be extracted to a custom hook
1. Hooks MUST compose other hooks — a hook with complex logic MUST be decomposed into smaller hooks

#### 1.2.2.3. State Management

1. UI state (theme, layout, sidebar, modals): `useContext<CTXType>(...)` / `createContext<CTXType>(...)`
1. Business logic state (user data, API responses, domain entities): Redux
1. Mixing UI state in Redux or business logic in Context is PROHIBITED

#### 1.2.2.4. CSS

1. Style objects MUST use `React.CSSProperties` type in `.styles.ts` files
1. For keyframes, pseudo-classes, and media queries: `<style>` tags in JSX
1. `.css` files are PROHIBITED
1. CSS modules are PROHIBITED
1. `styled-components` is PROHIBITED
1. Tailwind is PROHIBITED

#### 1.2.2.5. File Structure

| Suffix | Purpose | When to Create |
|--------|---------|---------------|
| `.tsx` | Component implementation | Always |
| `.types.ts` | Interfaces and type aliases | When component has props interface or exported types |
| `.consts.ts` | Constants (`SCREAMING_SNAKE_CASE`) | When component uses magic numbers, config values, or shared constants |
| `.styles.ts` | `CSSProperties` style objects | When component has non-trivial styles (>2-3 inline properties) |

1. Create ONLY the files the component needs
1. Directory names: `PascalCase` matching the component (`src/components/HexagonGrid/HexagonGrid.tsx`)
1. When both `.types.ts` and `.tsx` exist, `.types.ts` MUST be created first (types before implementation)

#### 1.2.2.6. Code Reuse

1. Code reuse is a first-class condition
1. Extraction triggers (when used by 2+ components in different directory subtrees):
   1. Types → shared `types/` directory
   1. Constants → shared `constants/` directory
   1. Style objects → shared `styles/` directory
   1. Utility functions → shared `utils/` directory
   1. Hooks → shared `hooks/` directory
1. Code shared within the same directory subtree MAY remain in a local shared file
1. Preference order:
   1. Import from shared directory (already exists)
   1. Extract to shared directory (create reusable artefact)
   1. Inline in component (only when truly component-specific)

### 1.2.3. Rust

#### 1.2.3.1. Naming

| Entity | Convention | Enforced By |
|--------|-----------|-------------|
| Structs / Enums / Traits | `PascalCase` | Compiler |
| Functions / Methods | `snake_case` | Compiler |
| Variables | `snake_case` | Compiler |
| Constants / Statics | `SCREAMING_SNAKE_CASE` | Compiler |
| Enum variants | `PascalCase` | Compiler |
| Modules / Files | `snake_case` | Compiler |
| Lifetimes | Short lowercase (`'a`, `'b`) | Convention |
| Type parameters | Single uppercase or `PascalCase` (`T`, `K`, `V`) | Convention |

#### 1.2.3.2. Tooling

1. Formatter: `rustfmt` — zero configuration, output is canonical
1. Linter: `clippy` — all warnings MUST be treated as errors (`#![deny(clippy::all)]`)
1. No `#[allow(...)]` without an accompanying comment explaining why

#### 1.2.3.3. Error Handling

1. `Result<T, E>` and `Option<T>` for all error handling
1. `unwrap()` and `expect()` are PROHIBITED in production code
1. `unwrap()` is permitted in tests only
1. Custom error types MUST implement `std::error::Error`
1. `thiserror` for library error types, `anyhow` for application error types
   > Review: confirm `thiserror`/`anyhow` against dependency whitelist

#### 1.2.3.4. Unsafe

1. `unsafe` blocks are PROHIBITED unless the operation cannot be expressed in safe Rust
1. Every `unsafe` block MUST have a `// SAFETY:` comment explaining the invariants being upheld
1. `unsafe` in a public API MUST be documented in the function's doc comment

### 1.2.4. OCaml

#### 1.2.4.1. Naming

| Entity | Convention | Enforced By |
|--------|-----------|-------------|
| Modules | `PascalCase` | Compiler |
| Module types (signatures) | `PascalCase` | Convention |
| Types | `snake_case` | Convention |
| Functions / Values | `snake_case` | Convention |
| Constructors / Variants | `PascalCase` | Compiler |
| Record fields | `snake_case` | Convention |

#### 1.2.4.2. Tooling

1. Formatter: `ocamlformat` — zero configuration, output is canonical
1. Build system: `dune`
1. All compiler warnings MUST be enabled and treated as errors

#### 1.2.4.3. Error Handling

1. `Result` and `Option` types for all error handling
1. Exceptions are PROHIBITED — use `Result` combinators (`bind`, `map`, `match`)
1. Pattern matching MUST be exhaustive — wildcard catch-all patterns (`_`) are PROHIBITED unless the match is provably complete

#### 1.2.4.4. Module Design

1. Every module MUST have a corresponding `.mli` interface file defining its public API
1. Implementation details MUST NOT be exposed through the interface
1. Functors are preferred over runtime polymorphism where applicable

### 1.2.5. Haskell

#### 1.2.5.1. Naming

| Entity | Convention | Enforced By |
|--------|-----------|-------------|
| Types / Typeclasses | `PascalCase` | Compiler |
| Functions / Values | `camelCase` | Convention |
| Modules | `PascalCase` (dot-separated hierarchy) | Compiler |
| Type variables | Lowercase single letter or short name (`a`, `m`, `f`) | Convention |

#### 1.2.5.2. Tooling

1. Formatter: `ormolu` — zero configuration, output is canonical
1. Linter: `hlint` — all suggestions MUST be reviewed, warnings MUST be resolved
1. Build system: `cabal` or `stack` (project-level decision)
   > Review: standardise on one build system

#### 1.2.5.3. Error Handling

1. `Either` and `Maybe` for all error handling
1. Partial functions (`head`, `tail`, `fromJust`) are PROHIBITED — use total alternatives
1. `error` and `undefined` are PROHIBITED in production code

#### 1.2.5.4. Style

1. Explicit type signatures MUST be provided for all top-level definitions
1. Point-free style is permitted only when it improves readability — do not obfuscate for the sake of conciseness
1. Language extensions MUST be declared per-file, not in the `.cabal` file
   > Review: approved language extensions whitelist

### 1.2.6. Python

#### 1.2.6.1. Naming

1. PEP 8 conventions apply:

| Entity | Convention |
|--------|-----------|
| Classes | `PascalCase` |
| Functions / Methods | `snake_case` |
| Variables | `snake_case` |
| Constants | `SCREAMING_SNAKE_CASE` |
| Modules / Packages | `snake_case` |
| Private | `_leading_underscore` |

#### 1.2.6.2. Tooling

1. Formatter: `ruff format`
1. Linter: `ruff check`
1. Type hints: MUST be used on all function signatures
1. Type checker: `mypy` in strict mode

#### 1.2.6.3. Scope Reminder

1. Python is approved for testing and prototyping ONLY
1. Python MUST NOT be used for production services
1. Code quality standards still apply — prototypes are not an excuse for unprincipled code

### 1.2.7. C\#

#### 1.2.7.1. Naming

| Entity | Convention |
|--------|-----------|
| Classes / Structs / Enums | `PascalCase` |
| Interfaces | `I`-prefix + `PascalCase` (`IDisposable`) |
| Methods / Properties | `PascalCase` |
| Local variables / Parameters | `camelCase` |
| Constants | `PascalCase` |
| Private fields | `_camelCase` |
| Namespaces | `PascalCase` |

#### 1.2.7.2. Tooling

1. Formatter: `dotnet format`
1. Analyser: Roslyn analysers with all warnings as errors

#### 1.2.7.3. Scope Reminder

1. C# is approved for Microsoft platform integration ONLY
1. C# MUST NOT be used outside of Microsoft integration contexts

### 1.2.8. Java

#### 1.2.8.1. Standards

1. Java is approved for legacy maintenance ONLY
1. Java MUST NOT be used for new projects
1. Existing Java code: standard Java conventions apply
1. Formatter: project-level formatter configuration (whatever exists in the legacy project)
1. No new conventions are defined — the goal is maintenance, not improvement

