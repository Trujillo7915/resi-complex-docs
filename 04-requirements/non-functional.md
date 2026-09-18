# Non-Functional Requirements (NFR)

> NFRs define the **qualities of the system** — not what it does but how well it does it.
> The golden rule: every NFR must have a metric. "The system must be fast" is not an NFR.
> "The P99 latency of the /orders endpoint must be < 200ms under 500 RPS load" is.

---

## How to write a measurable NFR?

| Bad | Good |
|-----|------|
| "The system must be fast" | "P95 latency must be < 300ms under 1000 concurrent RPS" |
| "The system must be secure" | "All endpoints require a valid JWT; tokens expire in 1 hour" |
| "The system must scale" | "The system must support up to 5000 concurrent users without degradation" |
| "The system must be available" | "Availability SLO: 99.9% monthly (maximum 44 min downtime/month)" |

---

## NFR-001: Performance

| Attribute | Metric | Test condition |
|-----------|--------|---------------|
| P95 latency — critical endpoints | < 300ms | Under [N] RPS load |
| P99 latency — critical endpoints | < 500ms | Under [N] RPS load |
| P95 latency — non-critical endpoints | < 1000ms | Normal load |
| Minimum throughput | [N] RPS | Without degradation |
| Service startup time | < 30 seconds | Cold start |

**Defined critical endpoints:**
- `POST /auth/login` (iam-service) — gates access to every other endpoint; a slow login blocks all 5 roles
- `POST /fees/generate` (billing-service) — monthly batch job that generates fees for every unit at once (FR08–FR09)
- `POST /maintenance-requests` (maintenance-service) — created in real time by residents/commercial owners reporting an issue (FR05)
- `POST /visits` (access-control-service) — logged in real time by the Security Guard at the front desk (FR12)

**Load testing tools:**
- k6, Apache JMeter, Locust, Gatling

**Where is it validated?** CI/CD in the staging pipeline before production.

---

## NFR-002: Availability

| Environment | SLO | Maintenance window | Max downtime/month |
|------------|-----|-------------------|-------------------|
| Production | 99.9% | Sundays 2am-4am | 44 minutes |
| Staging | 95% | No restriction | 36 hours |

> **Current status:** `01-context/scope.md` confirms only the Local environment for this formative delivery — Staging and Production are "not yet planned for the formative scope of this project." The SLOs above are the target once those environments exist; they are not measurable yet.

**Monthly error budget in production:** 44 minutes
**Error Budget policy:** If > 50% of the error budget is consumed in the first half of the month,
feature deploys are frozen until the next month and stability is prioritized.

**Health checks:**
- `GET /health` — Liveness: responds 200 if the process is alive
- `GET /health/ready` — Readiness: responds 200 only if it can process traffic (DB connected, dependencies OK)

---

## NFR-003: Scalability

| Scenario | Expected behavior |
|---------|------------------|
| Gradual load growth | Horizontal auto-scaling activated when CPU > 70% |
| Sudden spike (Black Friday, etc.) | System scales in < 2 minutes |
| Load reduction | Scale-down without interrupting active traffic |
| Horizontal scaling limit | Up to 3 instances per service (formative-project scale: a single residential complex, see `01-context/scope.md` assumption #3) |

**Strategy:** Stateless horizontal scaling — each instance does not store state in memory.
State goes in Redis (sessions, cache) or MySQL (persistent data).

---

## NFR-004: Security

### Authentication and Authorization
- All private endpoints require a valid JWT in the `Authorization: Bearer <token>` header
- JWT tokens expire in **1 hour**
- Refresh tokens valid for **7 days**
- RBAC (Role-Based Access Control): roles defined in `00-governance/security-policy.md`

### Data transmission
- HTTPS mandatory in production (TLS 1.2+)
- HTTP only in local development

### Sensitive data
- Passwords: hashing with bcrypt (cost factor ≥ 12) or Argon2id
- PII (personal data): encrypted at rest
- Secrets/keys: only in environment variables or vault, **never in code**

### OWASP Top 10
Code must be reviewed against the OWASP Top 10 on each release.
Tools: SAST (SonarQube/Snyk), dependency scanning, DAST in staging.

### Regulatory compliance
- Ley 1581 de 2012 (Colombian personal data protection / Habeas Data law) — applies because People Management stores residents' and commercial owners/tenants' personal data (`01-context/scope.md`, Constraints; `02-domain/domain-map.md`)

---

## NFR-005: Observability

| Pillar | Requirement | Tool |
|--------|------------|------|
| Logs | Structured JSON format + Correlation ID | Winston / Logback |
| Metrics | RED (Rate, Errors, Duration) per endpoint | Prometheus + Grafana |
| Traces | End-to-end distributed traces | OpenTelemetry + Jaeger |
| Alerts | Alert in < 5 min when SLI violates SLO | Alertmanager / PagerDuty |

**Correlation ID:** Each external request generates a UUID correlationId propagated in all logs and spans of that transaction.

---

## NFR-006: Maintainability

| Metric | Target |
|--------|--------|
| Test coverage | ≥ 80% of lines (≥ 90% in the domain) |
| Cyclomatic complexity | ≤ 10 per function |
| Technical debt | Resolution time < 1 sprint from registration |
| Onboarding time | A new dev can deploy locally in < 1 hour following `10-devops/local-setup.md` |
| Average build time | < 5 minutes in CI |

---

## NFR-007: Portability

- All services are deployed as Docker images
- Images work in any environment with a modern container orchestrator (specific platform/version pending — `01-context/overview.md` marks Infrastructure as "Pending definition")
- No service depends on the host operating system
- Environment variables are the only source of environment-specific configuration

---

## NFR-008: Disaster Recovery (DR / Recovery)

| Scenario | RTO (Recovery Time Objective) | RPO (Recovery Point Objective) |
|---------|------------------------------|-------------------------------|
| Single service failure | < 2 minutes (orchestrator restart) | 0 (stateless) |
| Primary database failure | < 5 minutes (failover to replica) | < 1 second (synchronous replication) |
| Availability zone loss | < 15 minutes — *out of scope for this formative delivery, only the Local environment is confirmed (`01-context/scope.md`)* | < 5 minutes |
| Full region disaster | < 4 hours (DR in secondary region) — *out of scope for this formative delivery* | < 1 hour |
---

## NFR priority matrix

| NFR | Priority (P1/P2/P3) | Validated in CI? | Owner |
|-----|---------------------|-----------------|-------|
| Performance | P1 | Not yet — planned (k6 in staging) | Pending — to be assigned by the team |
| Availability | P1 | Not yet — planned (health checks) | Pending — to be assigned by the team |
| Security | P1 | Not yet — planned (SAST + OWASP) | Pending — to be assigned by the team |
| Scalability | P2 | Not yet — manual, quarterly once staging exists | Pending — to be assigned by the team |
| Observability | P1 | Not yet — planned (smoke test in CI) | Pending — to be assigned by the team |
| Maintainability | P2 | Not yet — planned (coverage in CI) | Pending — to be assigned by the team |

---

## Correlations

- Detailed SLOs and SLAs → `13-operations/README.md`
- Pipeline that validates NFRs → `10-devops/README.md`
- Incidents related to NFR violations → `13-operations/incident-management.md`
- Security checklist → `00-governance/security-policy.md`
