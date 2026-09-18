# Hexagonal Architecture (Ports & Adapters) — resi-complex

> Hexagonal architecture, proposed by Alistair Cockburn, organizes a service so that the
> **business domain is completely independent** of the surrounding technology.
> The database, the web framework, the message broker — all are interchangeable details.
> What matters is the business logic, which lives at the center.

> **Stack note:** resi-complex's stack is **locked** to Java 17+ / Spring Boot 3.x
> (`01-context/overview.md`, `01-context/_template-project-profile.md`). This document therefore
> uses Java exclusively — unlike the scaffold's original generic version, it is not a
> multi-stack reference. For the concrete Maven/Gradle folder layout, dependencies, and naming
> conventions, see [`_stacks/java-spring.md`](../_stacks/java-spring.md).

> **Domain note:** all examples below use the real `MaintenanceRequest` aggregate from
> `02-domain/entities-and-rules.md` (Maintenance bounded context, `maintenance-service`),
> including its actual invariants INV-001, INV-002, and INV-003 — not a placeholder domain.
> The same structure applies identically to `AdministrationFee` (`billing-service`),
> `Correspondence` (`access-control-service`), and every other aggregate in the system.

---

## The problem it solves

```
❌ Traditional layered architecture:

  [HTTP Controller]
       ↓
  [Service]
       ↓
  [Repository]
       ↓
  [Database]

Problem: The "Service" mixes business logic with framework calls.
If you change the framework, you break the business. If you want to test the business,
you need to simulate the database.
```

```
✓ Hexagonal Architecture:

  [HTTP Controller]  [Kafka Consumer]  [Test]  ← Primary Adapters (enter the hexagon)
          │                 │            │
          └─────────────────┴────────────┘
                       │
                 [Driving Port]  ← Interface that defines the domain's API
                       │
               ┌───────────────┐
               │               │
               │    DOMAIN     │  ← Pure business logic, no external dependencies
               │  (Maintenance │
               │   Request)    │
               └───────────────┘
                       │
                 [Driven Port]  ← Interface the domain needs from the outside world
                       │
          ┌────────────┴──────┐
          │                   │
  [JPA Repository]  [Event Publisher]  ← Secondary Adapters (exit the hexagon)
```

---

## Folder structure (`maintenance-service`, per `_stacks/java-spring.md`)

```
maintenance-service/
└── src/
    └── main/
        └── java/com/resicomplex/maintenance/
            ├── domain/                              # No Spring dependencies — pure POJOs
            │   ├── model/
            │   │   ├── MaintenanceRequest.java       # Aggregate Root — owns INV-001..INV-003
            │   │   ├── RequestStatus.java             # Enum: PENDING/ASSIGNED/IN_PROGRESS/RESOLVED
            │   │   └── Priority.java                  # Enum: LOW/MEDIUM/HIGH/URGENT
            │   ├── event/
            │   │   ├── MaintenanceRequestCreated.java
            │   │   └── MaintenanceRequestStatusUpdated.java
            │   └── port/
            │       ├── in/
            │       │   ├── CreateMaintenanceRequestUseCase.java
            │       │   └── AssignMaintenanceRequestUseCase.java
            │       └── out/
            │           ├── MaintenanceRequestRepository.java
            │           └── EventPublisher.java
            │
            ├── application/                          # Orchestrates — uses Spring for DI, not web
            │   └── usecase/
            │       ├── CreateMaintenanceRequestService.java
            │       └── AssignMaintenanceRequestService.java
            │
            └── infrastructure/                        # Adapters — Spring Web, JPA, broker client
                ├── web/                                # Primary adapter: REST
                │   ├── MaintenanceRequestController.java
                │   └── dto/
                │       ├── CreateMaintenanceRequestRequest.java
                │       └── MaintenanceRequestResponse.java
                ├── persistence/                         # Secondary adapter: JPA (MySQL)
                │   ├── JpaMaintenanceRequestRepository.java   # implements MaintenanceRequestRepository
                │   └── entity/
                │       └── MaintenanceRequestJpaEntity.java   # @Entity — separate from domain model
                ├── messaging/                            # Secondary adapter: broker client (AT-002 pending)
                │   └── MaintenanceEventPublisher.java        # implements EventPublisher
                └── config/
                    └── AppConfig.java                        # Spring @Configuration — dependency wiring
```

**Dependency rule:** `domain/` does not import anything from `org.springframework.*` or
`jakarta.persistence.*`. POJOs only — exactly as `_stacks/java-spring.md` mandates project-wide.

---

## The Ports

Ports are **interfaces** (abstract contracts). The domain defines them; adapters implement them.

### Driving Port (Input Port)

Defines what the domain can do — its public API from the outside's perspective.

```java
// domain/port/in/CreateMaintenanceRequestUseCase.java
package com.resicomplex.maintenance.domain.port.in;

import com.resicomplex.maintenance.domain.model.Priority;

public interface CreateMaintenanceRequestUseCase {

    record CreateMaintenanceRequestCommand(
        String personId,
        String unitId,
        String type,
        String description,
        Priority priority
    ) {}

    String execute(CreateMaintenanceRequestCommand command); // returns the created request's id
}
```

### Driven Port (Output Port)

Defines what the domain needs from the outside world — without knowing how it is implemented.

```java
// domain/port/out/MaintenanceRequestRepository.java
package com.resicomplex.maintenance.domain.port.out;

import com.resicomplex.maintenance.domain.model.MaintenanceRequest;
import java.util.Optional;

public interface MaintenanceRequestRepository {
    void save(MaintenanceRequest request);
    Optional<MaintenanceRequest> findById(String id);
}
```

```java
// domain/port/out/EventPublisher.java
package com.resicomplex.maintenance.domain.port.out;

import com.resicomplex.maintenance.domain.event.DomainEvent;

public interface EventPublisher {
    void publish(DomainEvent event);
}
```

---

## The Domain — Aggregate Root with real invariants

This is the actual business logic from `02-domain/entities-and-rules.md` — nothing simplified.

```java
// domain/model/MaintenanceRequest.java
package com.resicomplex.maintenance.domain.model;

import com.resicomplex.maintenance.domain.event.DomainEvent;
import com.resicomplex.maintenance.domain.event.MaintenanceRequestCreated;
import com.resicomplex.maintenance.domain.event.MaintenanceRequestStatusUpdated;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

public class MaintenanceRequest {

    private final String id;
    private final String personId;
    private final String unitId;
    private final String type;
    private final String description;
    private final Priority priority;
    private String assignedTo;
    private RequestStatus status;
    private final List<DomainEvent> domainEvents = new ArrayList<>();

    private MaintenanceRequest(String id, String personId, String unitId, String type,
                                String description, Priority priority) {
        this.id = id;
        this.personId = personId;
        this.unitId = unitId;
        this.type = type;
        this.description = description;
        this.priority = priority;
        this.status = RequestStatus.PENDING;
    }

    // Factory method — enforces INV-001 and INV-003 at creation time
    public static MaintenanceRequest create(String personId, String unitId, String type,
                                             String description, Priority priority) {
        if (personId == null || personId.isBlank() || unitId == null || unitId.isBlank()) {
            throw new DomainException("INV-001: personId and unitId are required");
        }
        if (priority == null) {
            throw new DomainException("INV-003: priority is required on creation");
        }
        MaintenanceRequest request = new MaintenanceRequest(
            UUID.randomUUID().toString(), personId, unitId, type, description, priority
        );
        request.domainEvents.add(new MaintenanceRequestCreated(request.id, personId, unitId,
            type, description, priority, RequestStatus.PENDING));
        return request;
    }

    // Enforces INV-002 — status can only move forward, never backward
    public void assign(String staffId) {
        if (this.status != RequestStatus.PENDING) {
            throw new DomainException("INV-002: only a PENDING request can be assigned");
        }
        this.assignedTo = staffId;
        this.status = RequestStatus.ASSIGNED;
        this.domainEvents.add(new MaintenanceRequestStatusUpdated(this.id, this.status));
    }

    public void start() {
        if (this.status != RequestStatus.ASSIGNED) {
            throw new DomainException("INV-002: only an ASSIGNED request can move to IN_PROGRESS");
        }
        this.status = RequestStatus.IN_PROGRESS;
        this.domainEvents.add(new MaintenanceRequestStatusUpdated(this.id, this.status));
    }

    public void resolve() {
        if (this.status != RequestStatus.IN_PROGRESS) {
            throw new DomainException("INV-002: only an IN_PROGRESS request can be RESOLVED");
        }
        this.status = RequestStatus.RESOLVED;
        this.domainEvents.add(new MaintenanceRequestStatusUpdated(this.id, this.status));
    }

    public List<DomainEvent> domainEvents() {
        return List.copyOf(domainEvents);
    }

    public void clearEvents() {
        domainEvents.clear();
    }

    // Getters omitted for brevity — no setters: state changes only through the methods above
}
```

Notice: **zero Spring annotations, zero JPA, zero imports outside the JDK.** This class can be
unit tested in milliseconds with no Spring context, no database, and no HTTP server.

---

## The Adapters

### Primary Adapter — HTTP Controller

Translates the HTTP request to the domain use case. It contains no business logic.

```java
// infrastructure/web/MaintenanceRequestController.java
package com.resicomplex.maintenance.infrastructure.web;

import com.resicomplex.maintenance.domain.port.in.CreateMaintenanceRequestUseCase;
import com.resicomplex.maintenance.infrastructure.web.dto.CreateMaintenanceRequestRequest;
import com.resicomplex.maintenance.infrastructure.web.dto.MaintenanceRequestResponse;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/maintenance-requests")
public class MaintenanceRequestController {

    // Inject the port, NOT the concrete implementation
    private final CreateMaintenanceRequestUseCase createMaintenanceRequest;

    public MaintenanceRequestController(CreateMaintenanceRequestUseCase createMaintenanceRequest) {
        this.createMaintenanceRequest = createMaintenanceRequest;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public MaintenanceRequestResponse create(@RequestBody CreateMaintenanceRequestRequest body) {
        // Translate HTTP DTO → domain command
        var command = new CreateMaintenanceRequestUseCase.CreateMaintenanceRequestCommand(
            body.personId(), body.unitId(), body.type(), body.description(), body.priority()
        );
        String id = createMaintenanceRequest.execute(command);
        return new MaintenanceRequestResponse(id);
    }
}
```

> Note (P5, `overview.md` §5): a real controller in resi-complex must also enforce
> **ownership-level authorization** here or in a shared interceptor — e.g. a Person can only
> read requests they created (`requests:read:own`), and Maintenance Staff can only update
> requests assigned to them (`requests:update:assigned`), per `01-context/glossary.md`.

### Secondary Adapter — JPA Repository

Implements the driven port. The domain does not know MySQL or JPA exist.

```java
// infrastructure/persistence/JpaMaintenanceRequestRepository.java
package com.resicomplex.maintenance.infrastructure.persistence;

import com.resicomplex.maintenance.domain.model.MaintenanceRequest;
import com.resicomplex.maintenance.domain.port.out.MaintenanceRequestRepository;
import com.resicomplex.maintenance.infrastructure.persistence.entity.MaintenanceRequestJpaEntity;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public class JpaMaintenanceRequestRepository implements MaintenanceRequestRepository {

    interface SpringDataJpaRepository extends JpaRepository<MaintenanceRequestJpaEntity, String> {}

    private final SpringDataJpaRepository jpa;

    public JpaMaintenanceRequestRepository(SpringDataJpaRepository jpa) {
        this.jpa = jpa;
    }

    @Override
    public void save(MaintenanceRequest request) {
        jpa.save(MaintenanceRequestMapper.toJpaEntity(request)); // Mapper lives in infrastructure/, not domain/
    }

    @Override
    public Optional<MaintenanceRequest> findById(String id) {
        return jpa.findById(id).map(MaintenanceRequestMapper::toDomain);
    }
}
```

---

## The Use Case (Application Service)

Orchestrates the domain. Uses driving and driven ports. Contains **no** business logic —
that lives in the Aggregate.

```java
// application/usecase/CreateMaintenanceRequestService.java
package com.resicomplex.maintenance.application.usecase;

import com.resicomplex.maintenance.domain.model.MaintenanceRequest;
import com.resicomplex.maintenance.domain.port.in.CreateMaintenanceRequestUseCase;
import com.resicomplex.maintenance.domain.port.out.EventPublisher;
import com.resicomplex.maintenance.domain.port.out.MaintenanceRequestRepository;
import org.springframework.stereotype.Service;

@Service // Spring manages the lifecycle; the interface itself belongs to the domain
public class CreateMaintenanceRequestService implements CreateMaintenanceRequestUseCase {

    private final MaintenanceRequestRepository repository;
    private final EventPublisher eventPublisher;

    public CreateMaintenanceRequestService(MaintenanceRequestRepository repository,
                                            EventPublisher eventPublisher) {
        this.repository = repository;
        this.eventPublisher = eventPublisher;
    }

    @Override
    public String execute(CreateMaintenanceRequestCommand command) {
        // 1. Create the aggregate (business logic lives HERE, in the domain)
        MaintenanceRequest request = MaintenanceRequest.create(
            command.personId(), command.unitId(), command.type(),
            command.description(), command.priority()
        );

        // 2. Persist (through the port — the use case does not know which DB is used)
        repository.save(request);

        // 3. Publish domain events (through the port — broker technology is AT-002, still pending)
        request.domainEvents().forEach(eventPublisher::publish);
        request.clearEvents();

        return request.getId();
    }
}
```

---

## The Dependency Rule

> **Dependencies always point inward.**
> The domain does not import anything from application or infrastructure.
> Infrastructure imports from the domain (but never the other way around).

```
infrastructure/ → application/ → domain/
                                    ↑
                         CANNOT import anything from application/ or infrastructure/
```

### Dependency inversion (DI) in practice

```java
// ✓ Correct — domain defines the interface, infrastructure implements it
// In domain/port/out/:
public interface MaintenanceRequestRepository { ... }

// In infrastructure/persistence/:
public class JpaMaintenanceRequestRepository implements MaintenanceRequestRepository { ... }

// Spring's IoC container wires the concrete implementation automatically via @Repository/@Service —
// no manual bootstrap file is needed, unlike frameworks without a DI container.
```

---

## Advantages for TDD

Hexagonal architecture is ideal for TDD (mandatory per `CONTRIBUTING.md`: "Team rule: User
Stories are implemented using TDD. No exceptions.") because:

1. **The domain is testable without framework mocks.** `MaintenanceRequest` above needs no
   Spring context, no database, and no HTTP server to test its invariants.
2. **Driven ports can be faked easily.** In tests, an in-memory `MaintenanceRequestRepository`
   replaces the real JPA one.
3. **Invariants are explicit** (INV-001, INV-002, INV-003) and tested in isolation.

```java
// Domain unit test — zero external dependencies, per _stacks/java-spring.md's test layout
class MaintenanceRequestTest {

    @Test
    void rejects_creation_without_priority() {
        assertThatThrownBy(() ->
            MaintenanceRequest.create("p1", "u1", "plumbing", "leak", null)
        ).isInstanceOf(DomainException.class)
         .hasMessageContaining("INV-003");
    }

    @Test
    void cannot_be_assigned_twice() {
        MaintenanceRequest request = MaintenanceRequest.create("p1", "u1", "plumbing", "leak", Priority.URGENT);
        request.assign("staff-1");

        assertThatThrownBy(() -> request.assign("staff-2"))
            .isInstanceOf(DomainException.class)
            .hasMessageContaining("INV-002");
    }
}

// Use case test with a FAKE repository (not a real DB, not @SpringBootTest)
class CreateMaintenanceRequestServiceTest {

    @Test
    void saves_the_request_and_publishes_the_event() {
        var fakeRepo = new InMemoryMaintenanceRequestRepository();
        var fakePublisher = new InMemoryEventPublisher();
        var useCase = new CreateMaintenanceRequestService(fakeRepo, fakePublisher);

        var command = new CreateMaintenanceRequestUseCase.CreateMaintenanceRequestCommand(
            "p1", "u1", "plumbing", "Water leak in the main bathroom", Priority.URGENT
        );

        String id = useCase.execute(command);

        assertThat(fakeRepo.findById(id)).isPresent();
        assertThat(fakePublisher.published()).hasSize(1);
    }
}
```

> Full TDD flow (Red → Green → Refactor) and test doubles conventions →
> `11-quality/tdd-guide.md` (not yet created).

---

## Hexagonal Architecture Checklist

When reviewing a PR for any of the 9 services, verify:

- [ ] `domain/` has no imports from `infrastructure/` or `application/`
- [ ] `domain/` has no imports from Spring, JPA, or any other framework
- [ ] Every repository interface lives in `domain/port/out/`
- [ ] Every use case interface lives in `domain/port/in/`
- [ ] Mappers (`toDomain` / `toJpaEntity`) live in `infrastructure/`, not in `domain/`
- [ ] HTTP request/response DTOs live in `infrastructure/web/dto/`, not in `domain/`
- [ ] There is a unit test for each Aggregate invariant (see `02-domain/entities-and-rules.md`
      for the full invariant list per aggregate — every `INV-NNN` and `AGGR-INV-NNN` needs a test)
- [ ] Ownership-level authorization (P5) is enforced at the controller or a shared interceptor,
      not left implicit

---

## Common mistakes (anti-patterns)

| Anti-pattern | Why it is bad | Solution |
|-------------|--------------|---------|
| `import org.springframework.data.jpa.repository.JpaRepository;` inside `domain/model/` | Couples the domain to Spring Data JPA | Define your own port interface in `domain/port/out/` |
| Business logic in `MaintenanceRequestController` (e.g. computing whether a transition is valid) | If you change the endpoint, you change the business | Move to `MaintenanceRequest` |
| `JpaMaintenanceRequestRepository` returning a JPA entity instead of the domain `MaintenanceRequest` | The domain cannot validate invariants against a persistence-shaped object | Use a Mapper to reconstruct the Aggregate |
| A use case with 10+ constructor dependencies | It probably does too much — split it | Split into smaller, single-responsibility use cases |
| `Object` or raw `Map<String,Object>` in port interfaces | You lose the typed contract | Always use explicit typing (records, enums, value objects) |
| Enforcing `requests:read:own` only in the frontend | Any direct API call bypasses it | Enforce at the service layer, per P5 |

---

## References and correlations

- Bounded Contexts → `02-domain/domain-map.md`
- Entities, invariants, and aggregates for every service → `02-domain/entities-and-rules.md`
- Domain events → `02-domain/domain-events.md`
- Stack conventions (Maven, folder layout, naming) → `_stacks/java-spring.md`
- Complementary patterns (Outbox, Saga, CQRS) → `05-architecture/pattern-guide.md`
- Service catalog and container diagram → `05-architecture/overview.md`
- TDD applied to hexagonal architecture → `11-quality/tdd-guide.md` (not yet created)
- Service template with hexagonal structure → `09-microservices/_template/service/` (not yet created)
