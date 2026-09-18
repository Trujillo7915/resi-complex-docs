# Design Patterns and Microservices Guide — resi-complex

> This document is the project's pattern catalog.
> For each pattern: when to use it, when NOT to, and an implementation example.
> Patterns are not recipes — they are tools. Use them when the problem requires it.

> **Stack note:** resi-complex is locked to Java 17+ / Spring Boot 3.x
> (`01-context/overview.md`). The pattern *concepts* below are technology-agnostic and kept in
> pseudo-TypeScript for readability, matching the rest of this scaffold — but every concrete
> implementation in this project must follow `_stacks/java-spring.md`'s conventions
> (e.g. Resilience4j for Circuit Breaker, Spring Data JPA for repositories, Spring's
> `ApplicationEventPublisher` or a real broker client for domain events). See
> `hexagonal-architecture.md` for a fully worked Java example using the real
> `MaintenanceRequest` aggregate.

---

## Index

**Design patterns (GoF and SOLID)**
1. [Creational patterns](#creational)
2. [Structural patterns](#structural)
3. [Behavioral patterns](#behavioral)

**Microservices patterns**
4. [System decomposition](#decomposition)
5. [Inter-service communication](#communication)
6. [Resilience](#resilience)
7. [Data and consistency](#data)
8. [Observability](#observability)

---

## Design patterns (GoF) {#creational}

### 1. Factory Method

**Problem:** You want to create objects without exposing the creation logic or coupling code to the concrete type.

**When to use it:**
- When the exact type of object to create is not known until runtime
- When creation has complex logic (validations, configuration)

**Domain example (resi-complex's own `MaintenanceRequest.create(...)` in `hexagonal-architecture.md`):**

```typescript
// Factory Method — inside the Aggregate Root
class MaintenanceRequest {
  // Instead of new MaintenanceRequest(...), a factory method enforces INV-001 and INV-003
  static create(personId: PersonId, unitId: UnitId, type: string, description: string, priority: Priority): MaintenanceRequest {
    if (!personId || !unitId) throw new DomainException('INV-001');
    if (!priority) throw new DomainException('INV-003');
    return new MaintenanceRequest(RequestId.new(), personId, unitId, priority, RequestStatus.PENDING, null);
  }
}
```

---

### 2. Builder

**Problem:** An object has many optional parameters and construction becomes unreadable.

**When to use it:** Complex configuration objects, test data builders — e.g. building an
`AdministrationFee` test fixture with varying `unitType`, `amount`, and `status` combinations.

```typescript
// Builder — especially useful for tests
const fee = new AdministrationFeeBuilder()
  .forUnit('unit-id-123')
  .forPeriod('2026-09')
  .withUnitType(UnitType.COMMERCIAL)
  .withAmount(350000.00)
  .inStatus(FeeStatus.PENDING)
  .build();
```

---

### 3. Singleton (with caution)

**Problem:** A class must have exactly one instance.

**When to use it:** DB connections, configuration registries.

**WARNING:** Singleton makes testing difficult. Prefer dependency injection — Spring's IoC
container already manages singleton scope for every `@Service`, `@Repository`, and
`@Configuration` bean by default, so resi-complex should never need a hand-rolled Singleton.

```typescript
// ✓ Better: Singleton managed by the DI container, not by the class itself
container.registerSingleton(DatabaseConnection, DatabaseConnectionImpl);
```

---

### 4. Adapter (Structural pattern) {#structural}

**Problem:** You want to use an existing class but its interface does not match the one you need.

**When to use it:** Integration with external APIs, third-party libraries. resi-complex has no
external integrations in the MVP (`01-context/scope.md`), but this is exactly the shape of the
Secondary Adapter pattern already used for `JpaMaintenanceRequestRepository implements
MaintenanceRequestRepository` in `hexagonal-architecture.md` — the JPA repository *adapts*
Spring Data's interface to the domain's own port.

```typescript
// The domain defines the interface it needs
interface PaymentGatewayPort {
  charge(amount: Money, card: TokenData): Promise<ChargeResult>;
}

// Future v2 candidate (payments are out of MVP scope, 01-context/scope.md item 3):
class StripePaymentAdapter implements PaymentGatewayPort {
  async charge(amount: Money, card: TokenData): Promise<ChargeResult> {
    // Translate domain model → Stripe model, and back
  }
}
```

---

### 5. Decorator

**Problem:** You want to add behavior to an object without modifying it or inheriting from it.

**When to use it:** Logging, caching, validation, rate limiting around use cases — e.g. caching
`billing-service`'s fee-rate lookup (residential vs. commercial), which is read far more often
than it changes.

```typescript
// Cache decorator around the repository
class CachedFeeRateRepository implements FeeRateRepositoryPort {
  constructor(private readonly repo: FeeRateRepositoryPort, private readonly cache: CachePort) {}

  async findByUnitType(unitType: UnitType): Promise<Money> {
    const cached = await this.cache.get(`rate:${unitType}`);
    if (cached) return Money.fromCache(cached);
    const rate = await this.repo.findByUnitType(unitType);
    await this.cache.set(`rate:${unitType}`, rate, TTL_5_MINUTES);
    return rate;
  }
}
```

---

### 6. Observer / Internal Event Bus {#behavioral}

**Problem:** An object needs to notify others without knowing them directly.

**When to use it:** To publish domain events after persisting the aggregate — exactly what
`MaintenanceRequest.domainEvents()` does in `hexagonal-architecture.md`, and what every one of
the 16 events in `02-domain/domain-events.md` relies on.

```typescript
// The Aggregate accumulates events — the UseCase publishes them
class MaintenanceRequest {
  private readonly _events: DomainEvent[] = [];

  assign(staffId: StaffId): void {
    // ... business logic (INV-002) ...
    this._events.push(new MaintenanceRequestStatusUpdated(this.id, this.status));
  }

  get domainEvents(): DomainEvent[] { return [...this._events]; }
}
```

---

### 7. Strategy

**Problem:** You want to swap algorithms at runtime.

**When to use it:** resi-complex's own `FeeCalculationService`
(`02-domain/entities-and-rules.md`, Domain Services section) is already a Strategy-shaped
solution to a real requirement: applying a residential vs. commercial rate (FR09).

```typescript
interface FeeRateStrategy {
  calculate(unitType: UnitType, residentialRate: Money, commercialRate: Money): Money;
}

class StandardFeeRateStrategy implements FeeRateStrategy {
  calculate(unitType: UnitType, residentialRate: Money, commercialRate: Money): Money {
    return unitType === UnitType.COMMERCIAL ? commercialRate : residentialRate;
  }
}
```

---

### 8. Template Method

**Problem:** An algorithm has a fixed structure but some steps vary.

**When to use it:** Process flows with variations — e.g. `reports-service`'s FR19 exports
(requests-by-status vs. pending-fee-arrears) share the same fetch → transform → render skeleton.

```typescript
abstract class ReportExporter {
  async export(data: ReportData): Promise<Buffer> {
    const validated = await this.validate(data);
    const transformed = await this.transform(validated);
    return this.generate(transformed);
  }
  protected abstract transform(data: ReportData): Promise<TransformedData>;
  protected abstract generate(data: TransformedData): Promise<Buffer>;
  protected async validate(data: ReportData): Promise<ReportData> { return data; }
}
```

---

## Microservices Patterns

### Decomposition {#decomposition}

#### API Gateway

**Problem:** Clients need to call multiple services to get a response.

```
                    ┌─────────────────┐
Web ────────────▶  │   API Gateway   │ ──▶ [iam-service]
                    │                 │ ──▶ [units-service]
                    │                 │ ──▶ [...7 more services]
                    └─────────────────┘
                         Does:
                    - Routing
                    - Auth/AuthZ (JWT validation, centralized)
                    - Rate limiting
                    - CORS
```

**When to use it:** Always, in microservices architectures it is essential. **resi-complex has
not yet decided on this** — tracked as `overview.md` AT-001.

**Tools:** Kong, AWS API Gateway, NGINX, Traefik, Spring Cloud Gateway (the Spring-native option,
consistent with the locked stack).

---

#### Backend for Frontend (BFF)

**Problem:** Mobile and web need data in very different formats but share the same API.

**Not relevant to resi-complex's MVP:** `01-context/scope.md` explicitly excludes a mobile app
("web app only") — a single web client has no need for a BFF split.

---

#### Strangler Fig (Incremental migration)

**Problem:** You need to migrate a monolith to microservices without rewriting it all at once.

**Not relevant to resi-complex:** there is no pre-existing monolith to migrate from — the system
is greenfield (`01-context/overview.md`: "implementation not started").

---

### Inter-service communication {#communication}

#### Synchronous: REST / gRPC

| Aspect | REST | gRPC |
|--------|------|------|
| Protocol | HTTP/1.1 or HTTP/2 | HTTP/2 |
| Serialization | JSON (human-readable) | Protocol Buffers (efficient) |
| Typing | Manual with OpenAPI | Automatic with .proto |
| Recommended use | Public APIs, external communication | Internal service-to-service communication |

**resi-complex's choice:** REST, for all 9 services. `04-requirements/non-functional.md`'s
critical endpoints (`POST /auth/login`, `POST /fees/generate`, `POST /maintenance-requests`,
`POST /visits`) are already specified as REST paths — gRPC was never on the table for this
formative delivery, and `07-api/` is explicitly OpenAPI-based per `00-sdd-guide.md`.

---

#### Asynchronous: Message Broker (Kafka / RabbitMQ)

```
[units-service] ──publishes UnitRegistered──▶ [Topic: units.unit.registered] ──consumes──▶ [billing-service]
                                                                              ──consumes──▶ [access-control-service]
                                                                              ──consumes──▶ [communications-service]
```

**When to use asynchronous communication:** exactly the 16 flows already designed in
`02-domain/domain-events.md` (e.g. `FeeGenerated` → `communications-service` notifies the
Person, `reports-service` updates arrears). **The broker technology itself is still
undecided** — `overview.md` AT-002.

---

### Resilience {#resilience}

#### Circuit Breaker

**Problem:** A slow or failing service causes yours to fail too (failure cascade).

```
CLOSED (normal) → N consecutive failures → OPEN (fail fast) → after T seconds → HALF-OPEN → test call
```

**resi-complex's choice, once adopted:** Resilience4j — the Spring-native option, listed as the
cross-cutting concern's tool in `overview.md` §7. **Not yet implemented** (depends on AT-001/AT-002
existing first — there is little to circuit-break before services actually call each other).

---

#### Retry with Exponential Backoff

**Problem:** Transient failures (unstable network, service restarting).

```typescript
async function withRetry<T>(fn: () => Promise<T>, options = { attempts: 3, backoffBase: 1000 }): Promise<T> {
  for (let attempt = 1; attempt <= options.attempts; attempt++) {
    try { return await fn(); }
    catch (err) {
      if (attempt === options.attempts) throw err;
      await sleep(options.backoffBase * 2 ** (attempt - 1));
    }
  }
}
```

Directly applicable to resi-complex's event consumers per `02-domain/domain-events.md`'s
resilience section ("Retries before DLQ: 3-5, exponential backoff 1s → 2s → 4s → 8s").

---

### Data and consistency {#data}

#### Database per Service

**Rule:** Each of the 9 microservices has its own MySQL database. No service directly accesses
another service's database. **Already the standing decision** in `02-domain/domain-map.md`
(every bounded context's table row says "Database: MySQL (dedicated)").

```
✓ Correct:
  billing-service    → billing_db
  units-service      → units_db

✗ Incorrect:
  billing-service → JOIN with units-service's tables
```

**How does `billing-service` know a unit's type, then?** Via the `UnitRegistered` /
`UnitUpdated` events (`02-domain/domain-events.md`) — never a direct SQL join.

---

#### Saga (Distributed transactions)

**Problem:** A business transaction spans multiple services and you cannot use a distributed
ACID transaction.

**Not currently needed:** resi-complex's cross-context flows in `02-domain/domain-events.md`
are all **one-way reactive Policies** (e.g. "whenever `ExpenseProposalApproved` arrives, notify
the Administrator"), not multi-step transactions requiring compensation. If a future feature
needs a true multi-step distributed transaction (e.g. a v2 online-payment flow that must debit a
fee and confirm a bank transaction together), re-evaluate this pattern then.

---

#### CQRS (Command Query Responsibility Segregation)

**Problem:** The logic for writing data is very different from the logic for reading it.

**Where it already applies, implicitly:** `reports-service` (FR19) is, by definition, a
read-only projection built from other services' domain events (`02-domain/domain-map.md`:
"Report: A read-only, aggregated view built from other contexts' events"). This is CQRS's read
side without the team having to name it that way. **No other service needs CQRS** — their read
and write models are not different enough to justify the added complexity.

---

#### Outbox Pattern (Transactional)

**Problem:** You need to guarantee that when you save to the database, you also publish the
event — without risk of publishing it twice or losing it on a crash.

```sql
-- Outbox table, applicable to every service that publishes events
-- (units, people, maintenance, billing, access-control, finance-approval)
CREATE TABLE outbox (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_type  VARCHAR(100) NOT NULL,
  payload     JSONB NOT NULL,
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  published   BOOLEAN DEFAULT false
);
```

**Directly required by resi-complex's own design:** `02-domain/domain-events.md`'s resilience
section already commits to "at-least-once delivery + idempotent consumers" — the Outbox pattern
is the standard way to guarantee the "at-least-once" half of that promise (publishing a
`FeeGenerated` event can never be silently lost if `billing-service` crashes right after saving
the fee). **Recommended, not yet implemented** — it should land together with whichever broker
is chosen in AT-002.

---

#### Event Sourcing

**Not adopted.** Financial auditing exists in resi-complex (`ExpenseProposal`'s append-only
`Approval[]` history, FR18), but it is modeled as an explicit append-only child collection inside
the aggregate — not as full Event Sourcing of the aggregate's entire state. Full Event Sourcing
would add significant complexity with no corresponding requirement.

---

### Observability {#observability}

#### Sidecar Pattern

**Problem:** You want to add observability, configuration, or network capabilities to a service
without modifying its code.

**Deferred with AT-003** (container orchestration platform undecided) — a sidecar (Envoy/Istio
for service mesh, Filebeat for log shipping) only makes sense once the orchestrator is chosen.

---

## When NOT to use each pattern

| Pattern | Do not use it when... |
|---------|----------------------|
| CQRS | The read and write models are similar — true for every resi-complex service except `reports-service` |
| Event Sourcing | You do not need complete history — true for every aggregate except `ExpenseProposal`'s approval history, which is already modeled without full Event Sourcing |
| Saga | The transaction fits in a single service — true for every current resi-complex flow; revisit only if v2 introduces real distributed transactions (e.g. payments) |
| Circuit Breaker | The call is internal to the same service — never skip it once services actually call each other over the network |
| BFF | Clients have similar needs — true for resi-complex (web-only, `01-context/scope.md`) |

---

## Patterns adopted in this project

| Pattern | Adopted? | Justification / ADR |
|---------|---------|---------------------|
| API Gateway | **Pending — no ADR yet.** See `overview.md` AT-001. | Needed to centralize JWT validation, rate limiting, and CORS across 9 services instead of duplicating them in each |
| Database per Service | **Yes — no dedicated ADR yet, but already the standing decision in `02-domain/domain-map.md`.** Candidate ADR-004. | Each of the 9 bounded contexts already owns its Ubiquitous Language and data; sharing a database would silently reintroduce coupling the domain model was explicitly split to avoid |
| Circuit Breaker | **Deferred — no ADR.** See `overview.md` §6/§7. | Blocked on API Gateway (AT-001) and message broker (AT-002) existing first; premature to add resilience around calls that don't happen yet |
| Saga (choreographed) | **No — not needed by current scope.** | Every cross-context flow in `02-domain/domain-events.md` is a one-way reactive Policy, not a multi-step transaction requiring compensation; reconsider only if a future distributed transaction (e.g. v2 payments) requires it |
| Outbox Pattern | **Recommended, not yet implemented — no ADR.** Candidate to bundle with the message-broker ADR-005. | Directly required by `02-domain/domain-events.md`'s own "at-least-once delivery" commitment — without it, a crash between saving and publishing silently loses an event (e.g. a generated fee never notifies the resident) |
| CQRS | **Partial / implicit — `reports-service` only. No ADR.** | `reports-service` is, by its own FR19 definition, a read-only event-built projection — the read side of CQRS without a full write-side split; no other service's read/write models diverge enough to justify it |
| Event Sourcing | **No.** | `ExpenseProposal`'s append-only `Approval[]` history already satisfies FR18's traceability requirement without replaying the full aggregate from events |
| BFF | **No.** | `01-context/scope.md` confirms web-only, no mobile app in this MVP — a single client has no need for a BFF split |

---

## Correlations

- Hexagonal Architecture (fully worked Java example) → `05-architecture/hexagonal-architecture.md`
- ADR for pattern decisions → `05-architecture/decisions/` (candidates ADR-003 through ADR-006 — see `decisions/README.md`)
- Domain events these patterns support → `02-domain/domain-events.md`
- Architectural technical debt tracking the open decisions (AT-001, AT-002) → `05-architecture/overview.md` §8
- Outbox table would live in → `06-data/models.md` (per service, once written)

