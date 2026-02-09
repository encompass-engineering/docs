# 1. General

1. Observability is a first-class concern — it is a product feature, not an afterthought
1. All services (internal and client-deployed) MUST emit telemetry
1. A centralised observability platform MUST be operated as a shared service across all projects
1. The platform MUST support multi-tenancy with SSO and per-project authorisation
1. Clients MUST have read access to their own project's telemetry data
1. Internal teams MUST have access scoped to their projects unless explicitly granted broader access
1. OpenTelemetry is the mandatory standard for all telemetry (metrics, traces, logs)
1. Proprietary observability agents are PROHIBITED unless justified by ADR
1. Specific platform choice (Grafana stack, SigNoz, etc.) is an infrastructure decision requiring an ADR

# 2. Telemetry Pipeline

1. All services MUST emit telemetry to an OpenTelemetry Collector
1. The Collector handles routing, transformation, and export — services MUST NOT emit directly to the observability backend
1. Client-deployed services MUST emit telemetry to the centralised platform across network boundaries
1. The telemetry pipeline MUST support:
   1. Buffering and retry on network failure
   1. Tenant identification on all telemetry data
   1. Sampling configuration (per-service, per-environment)
   1. Data export in OpenTelemetry Protocol (OTLP) format
1. Telemetry data retention and export policies are per-project decisions — they MUST be documented

# 3. Logging Standards

## 3.1. Format

1. All logs MUST be structured JSON — plaintext logs are PROHIBITED
1. Every log entry MUST contain the following fields:

| Field | Description | Example |
|-------|-------------|---------|
| `timestamp` | ISO 8601 with timezone | `2026-02-08T14:30:00.000Z` |
| `level` | Severity level | `INFO` |
| `service` | Service name | `fault-finding-api` |
| `trace_id` | Distributed trace identifier | `abc123def456` |
| `message` | Human-readable description | `Request processed` |

1. Additional context fields (user ID, request ID, resource identifiers) SHOULD be included where applicable
1. Sensitive data in logs is PROHIBITED (as defined in Section 1.4.2)

## 3.2. Log Levels

1. Log levels MUST be used consistently across all services:

| Level | Meaning | When to Use |
|-------|---------|-------------|
| `TRACE` | Fine-grained diagnostic detail | Internal function flow, variable state (development only) |
| `DEBUG` | Diagnostic information for developers | Request/response detail, cache hits/misses |
| `INFO` | Normal operational events | Service started, request completed, job finished |
| `WARN` | Degraded but self-recovering condition | Retry succeeded, fallback activated, approaching threshold |
| `ERROR` | Something is broken and needs human attention | Unhandled failure, dependency down, data corruption |
| `FATAL` | Service cannot continue | Startup failure, unrecoverable state |

1. Misuse of log levels (e.g., logging expected conditions as `ERROR`, logging noise as `INFO`) is a code review rejection
1. `TRACE` and `DEBUG` MUST be disabled in production by default
1. Log level MUST be configurable at runtime without redeployment

## 3.3. Output

1. Services MUST log to stdout/stderr only — the infrastructure handles collection and routing
1. Log aggregation, indexing, and storage are the responsibility of the observability platform
1. Services MUST NOT write to log files, syslog, or any other destination directly

# 4. Metrics

## 4.1. Mandatory Service Metrics

1. Every API endpoint MUST emit the following metrics:
   1. Request count (by endpoint, method, status code)
   1. Error count (by endpoint, error type)
   1. Latency distribution (p50, p95, p99 by endpoint)
1. Every service MUST emit:
   1. Uptime
   1. Active connections / in-flight requests
   1. Dependency health (status of downstream services and datastores)

## 4.2. Method

1. The RED method MUST be used for service-level metrics:
   1. Rate — requests per second
   1. Errors — failed requests per second
   1. Duration — latency distribution
1. The USE method MUST be used for resource-level metrics:
   1. Utilisation — percentage of resource capacity in use
   1. Saturation — queue depth or backlog
   1. Errors — resource-level error count

## 4.3. Custom Metrics

1. Each service MUST define and document its own business-specific metrics
1. Business metrics MUST be reviewed as part of the service's design review
1. Metric naming MUST follow OpenTelemetry semantic conventions

# 5. Distributed Tracing

1. Distributed tracing MUST be implemented across all service-to-service calls
1. OpenTelemetry MUST be used for trace propagation
1. Every inbound request MUST generate or propagate a trace ID
1. Trace context MUST be propagated through all downstream calls:
   1. HTTP: `traceparent` and `tracestate` headers (W3C Trace Context)
   1. Message queues: trace context in message metadata
   1. gRPC: trace context in metadata
1. Traces MUST include span attributes for: service name, operation name, status, and error details (if applicable)
1. Sensitive data MUST NOT appear in span attributes or trace context

# 6. Error Tracking

1. All unhandled errors MUST be captured and reported to the observability platform
1. Error reports MUST include: stack trace, trace ID, service name, environment, and request context
1. Errors MUST be deduplicated and grouped by root cause
1. New error types MUST trigger a notification — repeated occurrences of known errors MUST NOT generate duplicate alerts
1. Error resolution MUST be tracked (open, acknowledged, resolved)

# 7. Health Checks

1. Every service MUST expose the following endpoints:
   1. `/health` — liveness check (is the process running)
   1. `/ready` — readiness check (can the service accept traffic, are dependencies available)
1. Health check endpoints MUST NOT require authentication
1. Health check responses MUST include:
   1. Status (`ok`, `degraded`, `unhealthy`)
   1. Dependency status (for readiness checks)
   1. Service version
1. Health check endpoints MUST respond within 1 second
1. Kubernetes liveness and readiness probes MUST point to these endpoints

# 8. Alerting

1. Every production service MUST define alerts for:
   1. Error rate exceeding threshold
   1. Latency (p99) exceeding threshold
   1. Availability dropping below threshold
   1. Health check failures
1. Every alert MUST have a linked runbook documenting: what the alert means, how to investigate, and how to remediate
1. Alerts without runbooks are PROHIBITED in production
1. Alert fatigue is a defect — alerts that are routinely ignored MUST be fixed, re-thresholded, or removed
1. Alert thresholds MUST be reviewed quarterly
   > Review: confirm review cadence

# 9. Client-Facing Observability

1. Client-deployed services MUST emit telemetry to the centralised observability platform
1. Each client/project MUST be isolated as a tenant — cross-tenant data access is PROHIBITED
1. Clients MUST have access to:
   1. Service health status and history
   1. Performance metrics (latency, error rates)
   1. Alerting on their own service instances
1. Client access MUST be authenticated via SSO and authorised per project
1. Client-facing dashboards and health reports MUST be provided as a standard deliverable for all client-deployed services
1. Data retention and export policies MUST be agreed per client and documented in the project scope

# 10. Service Level Objectives (SLOs)

1. Every production service MUST define SLOs for:
   1. Availability (e.g., 99.9%)
   1. Latency (e.g., p99 < 200ms)
   1. Error rate (e.g., < 0.1%)
1. SLOs MUST be measurable from the telemetry data
1. SLO compliance MUST be tracked and reported
1. SLO breaches MUST trigger review — repeated breaches MUST result in corrective action or SLO revision
