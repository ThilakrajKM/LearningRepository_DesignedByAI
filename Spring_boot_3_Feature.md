# Spring Boot 3 Architect-Level Interview Guide

This document consolidates core Spring Boot 3 architecture topics into an interview-focused reference, including:
- concise cheat sheets,
- a system-design textual architecture,
- top 50 interview Q&A mapped to sections.

---

## 1) IoC Container & Dependency Injection Architecture
Spring’s IoC container (`ApplicationContext`) manages object creation, wiring, lifecycle, and cross-cutting integrations.

### Core concepts
- **BeanDefinition**: metadata blueprint (class, scope, init/destroy methods, dependencies).
- **BeanFactory vs ApplicationContext**:
  - `BeanFactory`: minimal DI container.
  - `ApplicationContext`: enterprise features (events, i18n, AOP auto-proxying, environment, resource loading).
- **Dependency Injection styles**:
  - Constructor injection (preferred; immutable, testable).
  - Setter injection (optional dependencies).
  - Field injection (avoid in production design discussions).

### Interview architecture angle
- Explain how inversion of control decouples modules from concrete implementations.
- Mention `@Primary`, `@Qualifier`, and profiles for resolving multiple implementations.
- Discuss startup phases:
  1. Read bean definitions.
  2. Instantiate singletons.
  3. Apply post-processors.
  4. Publish context-ready event.

---

## 2) Auto-Configuration Architecture
Spring Boot 3 auto-configuration uses conditional bean registration to provide opinionated defaults.

### How it works
- Triggered by `@SpringBootApplication` (includes `@EnableAutoConfiguration`).
- Auto-config classes loaded from `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`.
- Conditions:
  - `@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`, etc.
- Order/tuning via `@AutoConfiguration(before/after=...)`.

### Interview points
- “Auto-config backs off when user-defined bean exists.”
- Use `--debug` or condition evaluation report to diagnose why bean was/wasn’t created.
- Prefer custom starter + auto-config for platform teams.

---

## 3) Spring MVC Architecture (Servlet Stack)
Thread-per-request synchronous model, built on Servlet container (Tomcat/Jetty/Undertow).

### Request flow
1. `DispatcherServlet` receives request.
2. `HandlerMapping` resolves controller method.
3. `HandlerAdapter` invokes handler.
4. Validation/data binding (`@Valid`, converters).
5. Return `ModelAndView` / response body via `HttpMessageConverter`.
6. Exception flow through `HandlerExceptionResolver` / `@ControllerAdvice`.

### Best use
- CRUD apps, blocking JDBC, straightforward request-response workloads.

---

## 4) Spring WebFlux (Reactive Stack)
Non-blocking reactive model built on Project Reactor (`Mono`, `Flux`) and Reactive Streams.

### Architecture
- Can run on Netty (event loop) or Servlet container with reactive adapters.
- Backpressure-aware pipelines.
- Functional routing (`RouterFunction`) or annotation-based controllers.

### Interview points
- Good for high-concurrency I/O-bound workloads.
- Avoid mixing blocking calls inside reactive pipeline unless isolated with bounded schedulers.
- Reactive end-to-end stack should include reactive DB/clients for full benefit.

---

## 5) Spring Security 6 Architecture
Security is filter-chain based and deeply integrated with Spring MVC/WebFlux.

### Core building blocks
- `SecurityFilterChain` bean replaces deprecated `WebSecurityConfigurerAdapter`.
- `AuthenticationManager`, `AuthenticationProvider`, `UserDetailsService`.
- Authorization via `AuthorizationManager`.
- Method security (`@EnableMethodSecurity`, `@PreAuthorize`).

### OAuth2/JWT architecture
- Resource Server validates JWT (signature, claims, scopes).
- Stateless session with CSRF strategy based on client type (browser vs API).
- Principle of least privilege + defense-in-depth.

---

## 6) Spring Data Architecture
Repository abstraction over data stores.

### Layers
- Domain entities/aggregates.
- Repository interfaces (`JpaRepository`, `ReactiveCrudRepository`, etc.).
- Query derivation (`findBy...`), `@Query`, Specifications/QueryDSL.

### Architecture insights
- Keep repositories persistence-focused, business logic in services.
- For complex read models use projections and dedicated query models.
- Understand JPA pitfalls: N+1, lazy loading boundaries, transaction scope.

---

## 7) Transaction Management
Declarative transaction management via AOP proxies and `@Transactional`.

### Concepts
- Propagation (`REQUIRED`, `REQUIRES_NEW`, etc.).
- Isolation levels.
- Rollback rules (runtime exceptions by default).
- Transaction managers: JPA/JDBC/JTA/reactive variants.

### Interview points
- Transactions are boundary concerns at service layer.
- Self-invocation bypasses proxy (classic gotcha).
- Align transaction scope with aggregate consistency boundaries.

---

## 8) Caching Architecture
Abstraction with pluggable providers (Caffeine, Redis, Hazelcast, etc.).

### Key annotations
- `@Cacheable`, `@CachePut`, `@CacheEvict`, `@Caching`.
- Key strategy, TTL, eviction policy, cache stampede prevention.

### Architecture practices
- Cache-aside pattern.
- Use Redis for distributed cache in microservices.
- Define cache invalidation as first-class design concern.

---

## 9) Spring Boot Actuator & Observability
Production diagnostics, health, metrics, tracing hooks.

### Components
- Endpoints: `/actuator/health`, `/metrics`, `/info`, `/env`, etc.
- Micrometer for metrics abstraction.
- OpenTelemetry integration for traces/log correlation.

### Interview points
- Differentiate liveness vs readiness probes.
- Use custom `HealthIndicator`.
- Control endpoint exposure/security explicitly.

---

## 10) Externalized Configuration
Hierarchical configuration from property files, env vars, command-line, config servers.

### Mechanisms
- `@ConfigurationProperties` (preferred for typed config).
- Profiles (`application-dev.yml`, etc.).
- Property source precedence and relaxed binding.

### Enterprise view
- Immutable config objects, fail-fast validation.
- Secrets from vault/KMS, not plain files.
- Separate deploy-time config from build artifact.

---

## 11) Spring Cloud Architecture (Microservices)
Patterns for distributed systems with Spring ecosystem.

### Typical components
- Config Server, Service Discovery, API Gateway.
- Circuit breaking, retries, bulkheads.
- Distributed tracing and centralized logging.

### Interview framing
- Explain CAP trade-offs and eventual consistency.
- Prefer choreography/orchestration based on domain complexity.
- Apply BFF/API composition where needed.

---

## 12) Messaging Architecture (RabbitMQ / Kafka)
Asynchronous communication for decoupling and resilience.

### RabbitMQ vs Kafka
- RabbitMQ: broker queues, routing, work queues, request-reply patterns.
- Kafka: distributed log, partitions, consumer groups, replay/event sourcing.

### Architecture concerns
- At-least-once semantics, idempotent consumers.
- DLQ/DLT strategy.
- Schema evolution (Avro/JSON schema + registry).

---

## 13) REST API Design & Exception Handling
Design APIs as stable contracts.

### Design principles
- Resource-oriented URIs, proper HTTP verbs/status codes.
- Pagination/filtering/sorting; versioning strategy.
- Idempotency for PUT/DELETE and safe retries.

### Error architecture
- Global handling via `@RestControllerAdvice`.
- Use RFC 7807 Problem Details (`ProblemDetail` in Spring 6).
- Structured error codes + trace correlation ID.

---

## 14) Testing Architecture
Testing pyramid with isolation and confidence balance.

### Layers
- Unit tests: pure logic (fast, isolated).
- Slice tests: `@WebMvcTest`, `@DataJpaTest`.
- Integration tests: `@SpringBootTest`.
- Contract tests for service boundaries.
- Testcontainers for realistic infra dependencies.

### Interview points
- Prefer deterministic tests.
- Avoid overusing full-context tests for every case.
- Use architecture tests (e.g., layering rules) in mature teams.

---

## 15) AOT Processing & GraalVM Native Image
Spring Boot 3 introduced robust AOT support for native compilation.

### Why it matters
- Faster startup, lower memory footprint.
- Great for serverless and rapid scale-out.

### Architecture implications
- Reflection/proxy/resource hints may be required.
- Prefer framework-managed patterns compatible with AOT transformations.
- Validate native builds in CI for critical services.

---

## 16) Bean Lifecycle & Scopes
Understanding lifecycle helps with initialization cost and memory behavior.

### Lifecycle steps
1. Instantiation
2. Dependency injection
3. Aware callbacks
4. `BeanPostProcessor` before init
5. Init methods (`@PostConstruct`)
6. Post-process after init
7. Destroy callbacks (`@PreDestroy`)

### Scopes
- `singleton` (default), `prototype`, `request`, `session`, `application`.
- Scoped proxies needed when injecting shorter-lived bean into singleton.

---

## 17) Performance Optimization & Best Practices
### High-impact tactics
- Keep startup lean (exclude unused auto-config).
- Connection pool tuning (HikariCP).
- Avoid N+1 with fetch joins/entity graphs.
- Use async/messaging for slow non-critical flows.
- Add caching where read-heavy and stable.
- Profile before tuning (JFR, Micrometer, APM).

### Design best practices
- Hexagonal/modular architecture.
- Clear bounded contexts.
- Separate command vs query models when complexity demands.

---

## 18) Jakarta EE Namespace Migration
Spring Boot 3 / Spring Framework 6 moved from `javax.*` to `jakarta.*`.

### Migration architecture notes
- Update all imports (`javax.servlet` → `jakarta.servlet`, etc.).
- Ensure dependencies are Jakarta-compatible (Servlet 6, JPA 3, Validation 3).
- Re-test filters, interceptors, security, and validation layers after migration.

### Interview-ready summary
- “Boot 3 is not just version bump; it requires API namespace migration and compatible dependency ecosystem.”

---

## 19) Architect-Level Cheat Sheet (Rapid Revision)

### Core Container & Bootstrapping
- IoC decouples creation from usage; constructor injection preferred.
- `ApplicationContext` = DI + events + environment + post-processors.
- Auto-config uses conditions and backs off when custom bean exists.

### Web Stack Selection
- MVC: blocking, thread-per-request, simpler operationally.
- WebFlux: non-blocking/event loop, better for high concurrency I/O.

### Security & Data
- Security 6 with `SecurityFilterChain` and method security.
- Repositories for persistence, services for business logic.
- Watch JPA fetch plans and transaction boundaries.

### Consistency & Resilience
- `@Transactional` at service boundaries.
- Caching needs invalidation strategy and TTL discipline.
- Messaging requires idempotency and dead-letter handling.

### Operations
- Actuator + Micrometer + OTel for observability.
- Externalized typed config with profile/env overrides.
- Spring Cloud patterns for distributed reliability.

### API, Tests, Runtime
- Standardized API errors (`ProblemDetail`).
- Testing pyramid with Testcontainers for realism.
- AOT/native for startup/memory gains.
- Complete Jakarta namespace migration for Boot 3 compatibility.

---

## 20) System-Design Style Reference Architecture Diagram (Textual)

```text
[ Clients ]
  |-- Web SPA / Mobile App / Partner API
  |
  v
[ API Gateway (Spring Cloud Gateway) ]
  - TLS termination
  - Auth token relay / rate limit / routing
  |
  +-----------------------------+
  |                             |
  v                             v
[ Auth Service ]            [ Domain Microservices (Spring Boot 3) ]
(Spring Security 6,         - Service A (Orders)
 OAuth2/OIDC)               - Service B (Inventory)
                            - Service C (Payments)
                            - Service D (Notifications)

Each Service (typical internal architecture):
  [Controller Layer]
      |
      v
  [Application/Service Layer] --- @Transactional boundary
      |               \
      |                \--> [Event Publisher]
      v
  [Domain Model + Business Rules]
      |
      v
  [Repository Layer (Spring Data)]

Data + integration per service:
  - Primary DB (PostgreSQL/MySQL)
  - Cache (Redis) via Spring Cache abstraction
  - Outbox table for reliable events
  - Message broker:
      * Kafka (event streams, replay)
      * RabbitMQ (task/workflow queues)

Cross-cutting platform components:
  [Config Server]  ---> externalized config/profiles/secrets refs
  [Service Registry] ---> discovery (if used)
  [Observability Stack]
     - Actuator + Micrometer
     - Prometheus/Grafana metrics
     - OpenTelemetry traces
     - Centralized logs (ELK/OpenSearch)
  [Resilience]
     - Circuit breaker/retry/timeouts (Spring Cloud CircuitBreaker/Resilience4j)

Request flow (sync + async hybrid):
  Client -> Gateway -> Service A -> DB/Cache
                            |
                            +-> publish domain event to Kafka
                                      |
                                      v
                              Service B/C consume event
                              (idempotent handler + DLQ)

Deployment/runtime:
  - Containerized services (Kubernetes)
  - Readiness/Liveness probes from Actuator
  - HPA scales stateless services
  - Native image optional for fast scale/start
```

---

## 21) Top 50 Spring Boot 3 Interview Q&A (Mapped)

### Section 1: IoC Container & DI Architecture
1. **What problem does IoC solve?**  
   It decouples object creation from usage, improving modularity and testability.
2. **BeanFactory vs ApplicationContext?**  
   BeanFactory is minimal; ApplicationContext adds enterprise integrations.
3. **Why constructor injection?**  
   Immutable dependencies, explicit contracts, easier tests.
4. **How to resolve multiple beans?**  
   `@Qualifier`, `@Primary`, profiles, conditionals.

### Section 2: Auto-Configuration Architecture
5. **How does Boot pick auto-configurations?**  
   Through classpath/property/bean conditions.
6. **What does “backs off” mean?**  
   Custom bean presence prevents default auto-config bean creation.
7. **How to debug auto-config decisions?**  
   Condition report (`--debug`) and actuator diagnostics.

### Section 3: Spring MVC Architecture
8. **Explain MVC request lifecycle.**  
   DispatcherServlet routes, controller executes, converters serialize.
9. **Where should validation happen?**  
   Boundary validation + domain invariant enforcement.
10. **How do you handle global exceptions?**  
    `@RestControllerAdvice` with consistent response schema.

### Section 4: WebFlux Architecture
11. **When choose WebFlux over MVC?**  
    High-concurrency I/O and reactive dependency ecosystem.
12. **What is backpressure?**  
    Demand-driven flow control between producer/consumer.
13. **Common WebFlux mistake?**  
    Blocking operations on event-loop threads.

### Section 5: Spring Security 6 Architecture
14. **Replacement for WebSecurityConfigurerAdapter?**  
    `SecurityFilterChain` bean.
15. **Authentication vs authorization?**  
    Identity verification vs permission decision.
16. **How to secure stateless APIs?**  
    OAuth2 resource server with JWT validation and scope mapping.
17. **Where to enforce fine-grained access?**  
    Method security + domain rule checks.

### Section 6: Spring Data Architecture
18. **Repository abstraction benefits?**  
    Less boilerplate, consistency, easier testing.
19. **Derived query vs @Query?**  
    Derived for simple predicates; `@Query` for complex logic.
20. **How to fix N+1?**  
    Fetch joins, entity graphs, projections, query tuning.

### Section 7: Transaction Management
21. **Why service-layer transactions?**  
    Service methods define business use-case atomicity.
22. **REQUIRED vs REQUIRES_NEW?**  
    Join existing transaction vs start isolated new transaction.
23. **Why self-invocation fails for @Transactional?**  
    Proxy interception is bypassed within same instance.

### Section 8: Caching Architecture
24. **When is caching valuable?**  
    Read-heavy, expensive queries with acceptable staleness.
25. **Difference: Cacheable/Put/Evict?**  
    Read caching, forced refresh, invalidation.
26. **How to manage stale data risk?**  
    Event-driven invalidation + TTL + versioned keys.

### Section 9: Actuator & Observability
27. **Liveness vs readiness probes?**  
    Process alive vs safe to receive traffic.
28. **Why Micrometer?**  
    Uniform instrumentation across metrics backends.
29. **What is minimum observability set?**  
    Metrics, traces, structured logs, correlation IDs.

### Section 10: Externalized Configuration
30. **Why ConfigurationProperties?**  
    Typed, validated, cohesive configuration.
31. **How does precedence help deployments?**  
    Env/CLI override defaults without rebuilding.
32. **How do you manage secrets?**  
    Vault/KMS/runtime injection, never plain-text in repo.

### Section 11: Spring Cloud Architecture
33. **Why API Gateway?**  
    Central policy enforcement and routing control.
34. **Is service discovery always required on Kubernetes?**  
    Not always; platform DNS may suffice for many cases.
35. **Why circuit breakers?**  
    Prevent cascading failures and improve graceful degradation.

### Section 12: Messaging Architecture
36. **Kafka vs RabbitMQ core distinction?**  
    Kafka = append-only stream log; RabbitMQ = brokered queues/routing.
37. **How to ensure consumer correctness?**  
    Idempotency keys, retries with backoff, DLQ.
38. **Exactly-once delivery practical?**  
    Usually at-least-once + idempotency is pragmatic.

### Section 13: REST API Design & Exception Handling
39. **What makes REST APIs interview-grade?**  
    Consistency, standards compliance, evolvability, explicit errors.
40. **Why use ProblemDetail?**  
    Standardized machine-readable error format.
41. **How to evolve APIs safely?**  
    Backward-compatible additive changes + deprecation strategy.

### Section 14: Testing Architecture
42. **Why not only end-to-end tests?**  
    Slow and brittle; balanced pyramid gives speed + confidence.
43. **Where does Testcontainers shine?**  
    Realistic integration testing with disposable infra.
44. **What is contract testing?**  
    Enforces provider/consumer schema behavior compatibility.

### Section 15: AOT & GraalVM Native Image
45. **Why native image?**  
    Lower startup time/memory for scale-sensitive workloads.
46. **What usually breaks in native mode?**  
    Reflection/proxy/resource assumptions without hints.

### Section 16: Bean Lifecycle & Scopes
47. **Singleton vs prototype?**  
    Shared instance vs new instance per lookup.
48. **How to inject request-scoped bean into singleton?**  
    Scoped proxy/provider indirection.

### Section 17: Performance Optimization & Best Practices
49. **First step in performance tuning?**  
    Profile and measure bottlenecks before changing design.

### Section 18: Jakarta EE Namespace Migration
50. **Biggest Boot 3 migration risk?**  
    Partial `javax` to `jakarta` migration across dependencies.

---

## 22) 30-Second Architect Interview Closing Script

> “In Spring Boot 3, I design for clear boundaries and operational excellence: transactional consistency for writes, async messaging for decoupling, cache-optimized reads, secure stateless APIs with Security 6, and observability by default using Actuator and Micrometer. I externalize configuration for portability, apply resilience patterns for distributed failures, and validate quality with layered testing plus Testcontainers. Where startup or memory is critical, I evaluate AOT/native images, ensuring Jakarta-compatible dependencies end-to-end.”
