<!--
  This document uses RFC 2119 keywords (MUST, MUST NOT, SHALL, SHALL NOT,
  SHOULD, SHOULD NOT, MAY) as defined in https://www.ietf.org/rfc/rfc2119.txt.
  Their meaning is strict and unambiguous.

  Divergence from any rule in this document MUST be recorded in an ADR.
-->
# 1. Guiding Principles

1. Every architectural decision MUST follow from the nature of the problem, not from convenience, convention, or compromise
1. Abstraction and generality MUST be preferred — solve the class of problems, not the instance
1. Constraints are features — restrictiveness reduces ambiguity for both human and machine contributors
1. Every external dependency MUST have a documented justification; building in-house is the default
1. All rules in this document are authoritative; divergence MUST be recorded in an ADR

# 2. Purpose

1. This document is the single authoritative source for all architectural and standards decisions
1. All downstream decisions (architecture → standards → code style → implementation) MUST be traceable to this document
1. ADRs serve as case law — they document justified divergences from this document, not alternatives to it
1. This document does not contain justifications; its rules are to be obeyed, not debated
1. Justification, context, and trade-off analysis belong exclusively in ADRs

# 3. Structure

## 3.1. Architecture Patterns

### 3.1.1. Microservices

1. A microservices architecture MUST be used when:
   1. The system MUST serve multiple independent clients or consumers
   1. Components MUST scale independently of one another
   1. The system MUST be deployed as containerised images (e.g., on-prem delivery)
   1. The domain consists of independently deployable bounded contexts with well-defined API boundaries
1. Each microservice MUST expose a versioned API contract
1. Each microservice MUST own its data store exclusively — shared mutable databases between services are PROHIBITED

### 3.1.2. Modular Monolith

1. A modular monolith MUST be used when:
   1. The output is a consumable artefact (library, toolkit, SDK, component system) rather than a running service
   1. The modules are composed and consumed together, not orchestrated independently
1. Module boundaries MUST be enforced at the code level (explicit public interfaces, no cross-module internal access)
1. A modular monolith MUST be structured such that extraction to microservices remains possible without architectural rework

### 3.1.3. Plain Monolith

1. Plain monoliths (monoliths without enforced module boundaries) are PROHIBITED
1. If the system is a single deployment unit, it MUST be a modular monolith

### 3.1.4. Serverless

1. A serverless architecture (AWS Lambda) MUST be used when:
   1. Execution is event-triggered and short-lived (≤15 minutes per invocation)
   1. Traffic is bursty or unpredictable with periods of zero activity
   1. The workload requires no persistent connections (no WebSockets, no gRPC streaming)
   1. Cold start latency is acceptable for the use case
1. Serverless MUST NOT be used when:
   1. The workload requires long-running processes
   1. Predictable low-latency responses are required
   1. Persistent connections are required
   1. The workload is steady-state (containerised reserved capacity is more cost-effective)

### 3.1.5. Containerised

1. A containerised architecture MUST be used when:
   1. The workload is long-running or persistent
   1. Predictable latency is required
   1. Persistent connections are required (WebSockets, gRPC streaming)
   1. The workload is steady-state
1. Containerised is the default deployment model — serverless MUST be justified against these criteria

### 3.1.6. Event-Driven

1. An event-driven architecture MUST be used when:
   1. Services MUST be decoupled in time (producer MUST NOT wait for consumer)
   1. Multiple independent consumers MUST react to the same event
   1. An audit trail of state changes is required with event replay capability
   1. Data synchronisation between bounded contexts is required
1. Approved message broker: Apache Kafka (AWS MSK)
1. Events MUST follow a defined schema contract
   > Review: schema registry tooling (e.g., AWS Glue Schema Registry, Confluent Schema Registry)

### 3.1.7. Data Architecture

1. Data retrieval and set-based transformations (filtering, joining, aggregating, grouping) MUST use SQL
1. Business logic and domain rules MUST live in application code
1. Stored procedures are PROHIBITED — logic MUST NOT be scattered across application and database layers
1. ETL pipelines: SQL for extraction and set-based transformation stages; application code (Rust) for logic-heavy transformation stages
1. The boundary between data operations and logic operations MUST be documented per pipeline in the project
1. Services MUST NOT couple through shared database access — shared data stores are permitted only when the store itself is the explicit bounded context (e.g., data lake, ETL staging area) with a defined write contract

## 3.2. Language Selection Criteria

### 3.2.1. Rust

1. Rust MUST be used when:
   1. The system requires low-level hardware or OS interaction
   1. The workload is performance-critical
   1. The domain maps to ownership semantics (resource lifecycle management, concurrent state)
   1. The system is a production-facing API service
1. Approved web frameworks: Axum, Actix
   > Review: finalise framework choice

### 3.2.2. OCaml

1. OCaml MUST be used when:
   1. Correctness MUST be provable or verifiable
   1. The domain maps naturally to algebraic data types and transformations
   1. The cost of a bug is catastrophically expensive (financial calculations, rule engines, data pipeline transformations, parsers/compilers)
   1. The system is an internal service where correctness dominates all other concerns
1. Status: Production-approved
   > Review: approved web framework (Dream, Eio)

### 3.2.3. Haskell

1. Haskell MAY be used under the same conditions as OCaml when purity guarantees are required beyond what OCaml provides
1. Status: Production-approved
   > Review: determine if Haskell and OCaml should have distinct trigger conditions or if Haskell is a strict superset

### 3.2.4. TypeScript

1. TypeScript MUST be used for:
   1. Frontend applications
   1. Rapid prototyping (when a browser-based interface is required)
1. TypeScript MUST NOT be used for production backend services
1. Approved frontend framework: React

### 3.2.5. Python

1. Python MUST be used for:
   1. Testing tooling and test harnesses
   1. Prototyping and proof-of-concept work
1. Python MUST NOT be used for production services

### 3.2.6. C\#

1. C# MUST be used when:
   1. Microsoft platform compatibility is a hard requirement
1. C# MUST NOT be used outside of Microsoft integration contexts

### 3.2.7. Java

1. Java is approved for maintenance of existing legacy systems ONLY
1. Java MUST NOT be used for new projects

## 3.3. Prohibited Patterns

1. Service-to-service coupling through shared database access is PROHIBITED (shared data stores with a defined write contract, e.g., data lakes and ETL staging areas, are exempt)
1. Stored procedures are PROHIBITED
1. Bidirectional dependencies between modules or services are PROHIBITED
1. Inheritance hierarchies deeper than 2 levels are PROHIBITED
1. Synchronous call chains spanning more than 2 services are PROHIBITED
1. Plain monoliths (no enforced module boundaries) are PROHIBITED

## 4.1. Security

### 4.1.1. Authentication

1. Authentication standard: OIDC
1. SAML MUST NOT be used unless mandated by a client's IdP — divergence MUST be recorded in an ADR

### 4.1.2. Authorization

1. Authorization model: RBAC (Role-Based Access Control)
1. RBAC MUST be built in-house
1. If a project requires ABAC, an ADR MUST be written
1. If ABAC is adopted, the order of preference is: build in-house → OPA/Rego (open source) → proprietary solution
1. Dependency preference (general rule): build in-house → open source → proprietary

### 4.1.3. Encryption

1. Asymmetric encryption: ECC
1. Symmetric encryption: AES-256

### 4.1.4. Input Validation

1. All input MUST be validated at the point of entry (API gateway, handler, boundary)
1. Validation MUST use allowlists — define what IS valid, not what IS NOT
1. Invalid input MUST be rejected — never sanitised and passed through
1. Type-level validation MUST be used where the language supports it (Rust's type system, OCaml's variants)

### 4.1.5. Output Encoding

1. Output encoding MUST be context-aware:
   1. HTML context: HTML entity encoding
   1. SQL context: parameterised queries — string concatenation for SQL is PROHIBITED
   1. URL context: URL encoding
   1. JSON context: proper JSON serialisation via approved libraries

### 4.1.6. Session Management

1. User-facing sessions: server-side session storage
   1. Session ID stored in a secure, HttpOnly, SameSite cookie
   1. Session state stored server-side (Redis or database)
   1. Revocation MUST be immediate on logout or security event
   > Review: session expiry duration
1. Service-to-service authentication: short-lived JWTs (≤15 minutes)
   1. JWTs are permitted ONLY for service-to-service communication
   1. JWT expiry MUST NOT exceed 15 minutes
   > [Note: short-lived JWTs are an interim pattern — the intent is to migrate to pure server-side sessions as infrastructure matures]

### 4.1.7. CORS

1. All origins MUST be denied by default
1. Allowed origins MUST be explicitly allowlisted per service
1. Wildcard origins (`*`) are PROHIBITED in production

### 4.1.8. Rate Limiting

> [Note: rate limiting strategy is under review. Current thinking:
> - Tiered by endpoint sensitivity (auth endpoints low, read endpoints high, write medium, admin low)
> - Token bucket algorithm (aligns with AWS API Gateway native behaviour)
> - Enforce at gateway + application level (defence in depth)
> - 429 Too Many Requests + Retry-After header + audit log on breach
> - Open question: specific numbers per tier need to be determined based on real workload profiles
> - Escalation policy (temp ban after N violations) is undecided]

1. Rate limiting MUST be enforced on all public-facing endpoints
1. Rate limit breaches MUST return `429 Too Many Requests` with a `Retry-After` header
1. All rate limit breaches MUST be logged

### 4.1.9. Audit Logging

1. The following events MUST be logged:
   1. All authentication attempts (success and failure)
   1. All authorization failures
   1. All data mutations (create, update, delete)
   1. All admin and privileged actions
   1. All access to sensitive data
   1. All API calls
1. Audit logs MUST include: timestamp, actor, action, resource, outcome, source IP
1. Audit logs MUST NOT contain secrets, tokens, or credentials
   > Review: audit log retention period

### 4.1.10. Penetration Testing

1. Penetration testing MUST be performed at every major version release

### 4.1.11. Vulnerability Scanning

1. Static application security testing (SAST) and dependency scanning MUST run on every CI pipeline execution
1. Builds with critical or high vulnerabilities MUST fail

### 4.1.12. Dependency Vulnerability Thresholds

1. Critical vulnerabilities: 0 tolerance — MUST be resolved immediately
1. High vulnerabilities: 0 tolerance — MUST be resolved immediately
1. Medium vulnerabilities: MUST be resolved within 7 days

### 4.1.13. Security Headers

1. The following security headers MUST be enforced on all HTTP responses:
   1. Content-Security-Policy (CSP)
   1. Strict-Transport-Security (HSTS)
   1. X-Frame-Options
   1. X-Content-Type-Options
   1. Referrer-Policy
   1. Permissions-Policy

## 4.2. Secrets Management

1. Approved secret storage: AWS Secrets Manager
1. Hardcoded secrets are PROHIBITED
1. Secrets in logs are PROHIBITED
1. Secrets in URLs are PROHIBITED
1. Secrets in source control are PROHIBITED
1. Local development authentication method: peer
   > Review: secret rotation frequency
   > Review: environment variable naming conventions
   > Review: secret access audit requirements
   > Review: emergency secret rotation procedures
   
## 4.3. Availability

1. Target uptime percentage (99.9%, 99.95%, 99.99%)

   > Review

1. Planned maintenance windows: (yes)

   > Review

1. Maximum acceptable downtime per incident

   > Review

1. Health check endpoint requirements

   > Review

1. Graceful degradation requirements

   > Review

1. Multi-region deployment requirements > Review
<!--Autopopulate reqs into repos -->

## 4.4. Fault Tolerance

1. Retry policy standards: exponential backoff
1. Circuit breaker thresholds (failure rate: 50%, recovery time: 30s)

   > Review

1. Timeout defaults by operation type (API: 30s, DB: 5s, external: 10s)

   > Review

1. Fallback behavior requirements

   > Review

1. Data replication requirements

   > Review

1. Disaster recovery RTO/RPO targets

   > Review

1. Idempotency requirements
   > Review

# 5. Performance Expectations

1. API response time targets (p50: 100ms, p95: 500ms, p99: 1s)

   > Review

1. Page load time targets (LCP, FID, CLS thresholds)

   > Review

1. Database query time limits

   > Review

1. Background job execution limits

   > Review

1. Memory usage thresholds

   > Review

1. CPU usage thresholds

   > Review

1. Concurrent user capacity targets

   > Review

1. Payload size limits
   > Review

# 6. Scalability Considerations

1. Horizontal scaling requirements: SHOULD <!--maybe must -->
1. Vertical scaling limits: MUST
1. Auto-scaling trigger thresholds
   > Review
1. Statelessness requirements
   > Review
1. Database connection pooling standards
   > Review
1. Cache strategy requirements (TTL defaults, invalidation rules)
   > Review
1. Queue processing capacity targets
   > Review
1. Sharding strategy guidelines
   > Review

# 7. Compliance Requirements

1. Data residency requirements by region
   > Review
1. GDPR compliance checklist
   > Review
1. PCI-DSS requirements (if applicable)
   > Review
1. SOC 2 control mapping
   > Review
1. HIPAA requirements: Not applicable
1. Data retention policies by data type
   > Review
1. Right to deletion implementation requirements
   > Review
1. Consent management requirements
   > Review
1. Data classification levels (public, internal, confidential, restricted)
   > Review
1. Cross-border data transfer rules
   > Review
1. Audit trail retention period
   > Review
1. Compliance certification renewal schedule
   > Review
1. Third-party processor requirements
   > Review
1. Data processing agreement requirements
   > Review
1. Privacy impact assessment triggers
   > Review
1. Breach notification procedures and timelines
   > Review

# 8. Dependencies

## 8.1. External Dependencies

### 8.1.1. Dependency Whitelist

1. All external dependencies MUST be approved and listed in a single centralised dependency registry
1. The centralised registry is the single source of truth — all repositories MUST reference it
1. Any dependency not on the whitelist is PROHIBITED
1. CI MUST fail if an unapproved dependency is detected in the lock file
1. Language-level standard libraries are exempt from the whitelist
1. Adding a new dependency MUST be recorded in an ADR
   > [Note: create an ADR template for "new dependency approval" — see Section 1.10]

### 8.1.2. Dependency Approval Criteria

1. Order of preference: build in-house → open source → proprietary
1. Every external dependency MUST have a documented justification for why building in-house is not viable
1. The following criteria MUST be met for approval:
   1. License MUST permit commercial use, modification, and distribution without requiring disclosure of source code
   1. Approved licenses: MIT, Apache 2.0, BSD 2-Clause, BSD 3-Clause, ISC
   1. All other licenses require an ADR with legal review
   1. Maturity: the project MUST be at least 5 years old unless an exceptional circumstance is documented in an ADR (e.g., implementation of a newly published research paper)
   1. Maintenance health: active maintenance, multiple contributors, responsive to security issues
      > Review: define specific thresholds (minimum contributors, maximum time since last commit)
   1. Security: no unresolved critical or high vulnerabilities
   1. Transitive dependency count MUST be evaluated as part of the review

### 8.1.3. Dependency Versioning

1. Dependencies MUST be pinned to the approved major version
1. Minor and patch updates within the approved major version are permitted
1. Major version upgrades MUST be treated as a new dependency approval (new ADR)

### 8.1.4. Forking Policy

1. If a dependency meets functional requirements but fails maintenance or maturity thresholds, it MUST be forked and maintained in-house
1. Forked dependencies MUST be treated as in-house code (subject to all internal standards, testing, and review processes)
1. Forked dependencies MUST be stored in the organisation's source control

### 8.1.5. Prohibited Licenses

1. GPL (all versions) is PROHIBITED
1. AGPL is PROHIBITED
1. SSPL is PROHIBITED
1. EUPL is PROHIBITED
1. Creative Commons (for code) is PROHIBITED
1. Any license requiring disclosure of proprietary source code is PROHIBITED

## 8.2. Internal Dependencies

### 8.2.1. Coupling Rules

1. Bidirectional dependencies between modules or services are PROHIBITED
1. Services MUST communicate only through defined interfaces (API contracts, events)
1. Shared libraries between services MUST contain only infrastructure and utility code (serialisation, logging, auth middleware)
1. Shared libraries MUST NOT contain domain or business logic
1. If two services share domain logic, they MUST either be consolidated into one service or communicate via events
1. High-level modules MUST define the interfaces they require; low-level modules MUST implement them (dependency inversion)
1. Dependencies MUST point inward toward core domain logic, never outward toward infrastructure
   > [Note: implementation-level detail on dependency inversion patterns belongs in Section 2 (Coding Standards)]

### 8.2.2. Service Fan-Out

1. A single service SHOULD NOT depend on more than 5 direct service dependencies
   > Review: finalise fan-out limit
1. Synchronous call chains spanning more than 2 services are PROHIBITED (see 1.3.3)
# 9. Interfaces

## 9.1. Universal API Principles

1. All APIs MUST be schema-first — the specification MUST be written before implementation
1. All APIs MUST be versioned using URL path versioning (`/v1/resource`)
1. All endpoints MUST be authenticated unless explicitly marked as public
1. All API changes MUST be backwards compatible within a major version
1. All APIs MUST have documentation auto-generated from the schema — hand-written API documentation that can drift from implementation is PROHIBITED
1. Supplementary documentation (tutorials, examples) MUST be updated within 5 business days of a change

### 9.1.1. Schema Formats

1. REST APIs: OpenAPI 3.x
1. gRPC APIs: Protobuf
1. GraphQL APIs: GraphQL SDL

### 9.1.2. Error Handling

1. REST and GraphQL APIs MUST use RFC 7807 (Problem Details for HTTP APIs) for error responses
1. gRPC APIs MUST use gRPC native status codes with `google.rpc.Status` and `google.rpc.ErrorDetail`
1. Error responses MUST include sufficient detail for the consumer to understand and resolve the issue without inspecting server logs

### 9.1.3. Pagination

1. All list endpoints MUST use cursor-based pagination
1. Offset-based pagination is PROHIBITED

## 9.2. API Type Selection Criteria

### 9.2.1. REST

1. REST MUST be used when:
   1. The API is public-facing or client-facing
   1. The interface is resource-oriented (CRUD operations on entities)
   1. The consumers are browsers, mobile apps, or third-party integrators
1. Endpoint naming: plural nouns, kebab-case (`/v1/order-items`)
1. HTTP methods: strict REST semantics
   1. GET: read (MUST be idempotent, MUST NOT have side effects)
   1. POST: create
   1. PUT: full replacement
   1. PATCH: partial update
   1. DELETE: remove

### 9.2.2. gRPC

1. gRPC MUST be used when:
   1. The communication is service-to-service (internal)
   1. Low latency and high throughput are required
   1. The interface is action-oriented (RPC-style) rather than resource-oriented
   1. Bidirectional streaming is required

### 9.2.3. GraphQL

1. GraphQL MUST be used when:
   1. The consumer requires flexible, client-driven queries (requesting different field subsets per call)
   1. The API aggregates data from multiple backend services into a single consumer-facing interface

## 9.3. Public Contracts

### 9.3.1. Breaking Changes

1. A breaking change is defined as any of the following:
   1. Removing a field from a response
   1. Removing or renaming an endpoint
   1. Changing a field's type
   1. Adding a required field to a request
   1. Changing authentication or authorization requirements
   1. Changing the meaning or semantics of an existing field
   1. Changing error codes for existing failure modes
   1. Changing URL structure
1. Breaking changes MUST result in a new major API version

### 9.3.2. Deprecation and Sunset

1. Deprecated API versions MUST have a minimum 6-month notice period before removal
1. After the 6-month deprecation period, the deprecated version MUST be removed — no extensions
1. Deprecated endpoints MUST include `Deprecation` and `Sunset` HTTP headers in every response
1. Consumers MUST receive direct notification (email or webhook) at the start of the deprecation period

### 9.3.3. Contract Testing

1. Every API MUST have contract tests that verify the published schema is honoured
1. Contract tests MUST run in CI — failures MUST block deployment
1. REST: contract tests MUST be generated from or validated against the OpenAPI spec
1. gRPC: Protobuf compilation serves as the contract enforcement mechanism
1. GraphQL: schema validation tests MUST verify all queries against the published SDL

### 9.3.4. Changelog

1. Every API MUST maintain a changelog
1. Changelog entries MUST be added in the same PR as the code change
1. Changelog format MUST follow Keep a Changelog conventions:
   1. Added
   1. Changed
   1. Deprecated
   1. Removed
   1. Fixed
   1. Security

### 9.3.5. SLA Definitions

1. All public APIs MUST publish an SLA covering:
   1. Availability target
   1. Latency targets (p95, p99)
   1. Error rate threshold
1. Specific values are defined per service

### 9.3.6. Rate Limit Communication

1. All API responses MUST include rate limit headers:
   1. `X-RateLimit-Limit`
   1. `X-RateLimit-Remaining`
   1. `X-RateLimit-Reset`
1. Rate limit breaches MUST return `429 Too Many Requests` with a `Retry-After` header

### 9.3.7. Consumer Notification

1. Breaking changes and deprecations MUST be communicated via:
   1. The API changelog
   1. `Deprecation` and `Sunset` response headers
   1. Direct notification to registered consumers (email or webhook)

### 9.3.8. SDK Support

1. Public APIs SHOULD provide auto-generated client SDKs from the published schema (OpenAPI, Protobuf)
1. Generated SDKs MUST be versioned in lockstep with the API version they target
1. SDK generation tooling and target languages are defined per project
   > Review: define approved SDK generation tooling and target languages when this becomes a requirement
# 10. Architecture Decision Records (ADRs)

## 10.1. General

1. ADRs are the sole mechanism for recording divergences from this document
1. ADRs serve as case law — they document justified divergences, not alternatives
1. All ADRs MUST be stored in a single centralised repository
1. ADRs MUST be the only place where justification, context, and trade-off analysis are recorded

## 10.2. Format

1. Numbering scheme: sequential (`NNNN`)
1. Naming convention: `NNNN-short-title.md`
1. Short title MUST use kebab-case
1. Required sections:
   1. Title
   1. Status
   1. Context
   1. Decision
   1. Consequences
1. Optional sections:
   1. Alternatives considered (MUST include at least 2 alternatives when present)
   1. References
   1. Related ADRs
1. Template enforcement: ADR format MUST be validated by linting and CI check

## 10.3. Status Values

1. `proposed` — under review, not yet approved
1. `accepted` — approved and in effect
1. `deprecated` — no longer recommended but not yet replaced
1. `superseded` — replaced by a newer ADR (MUST reference the superseding ADR)
1. `rejected` — proposed but denied (MUST include rejection rationale in Consequences)

## 10.4. Mandatory Triggers

1. An ADR MUST be written when any of the following occur:
   1. Divergence from any rule in this document (Guiding Principles, Standards, or Processes)
   1. Adoption of a new external dependency
   1. Architecture change not already covered in this document
   1. Security pattern change
   1. Infrastructure change not already covered in this document
   1. New API major version
   1. Adoption of a new technology, framework, or tool

## 10.5. Approval Process

1. ADRs require approval from a single approver (tech lead or architect)
1. The approver MUST NOT be the author of the ADR
1. Approved ADRs MUST be merged to the centralised repository before the associated change is implemented

## 10.6. Lifecycle

1. Existing ADRs MUST be reviewed at every major version release
1. Review MUST assess whether the ADR is still relevant, should be deprecated, or should be incorporated into this document
1. Superseding an ADR MUST create a new ADR that references the superseded one
1. ADRs MUST be linked to related code changes and PRs where applicable

## 10.7. Searchability

1. ADRs MUST be searchable by title, status, and trigger type
1. The centralised repository MUST maintain an index of all ADRs
   > Review: determine index format (auto-generated or manually maintained)

