<!-- EXAMPLE: A fully worked criteria file for a Spring Boot microservice. This is a
     GUIDELINE to copy from — replace its contents with your own project's reality when
     you create review-criteria.md. It is NOT loaded by the skill directly. -->

# Code Review Criteria — Orders Service (Spring Boot Microservice)

> The `multipass-review` skill embeds this file into every fresh review agent's prompt.
> Be categorical, not subjective. Define what to skip as explicitly as what to flag.

## 1. Tech stack & context

- **Language / runtime:** Java 21, Spring Boot 3.3, Maven.
- **Frameworks:** Spring Web (REST), Spring Data JPA (PostgreSQL), Spring Security, Spring
  Cloud OpenFeign (inter-service calls), Resilience4j, MapStruct, Lombok, JUnit 5 + Mockito.
- **Architecture:** Layered microservice — `controller → service → repository`. DTOs at the
  edge, JPA entities never leave the service layer. Inter-service calls go through Feign
  clients wrapped in Resilience4j circuit breakers. Kafka for async domain events.
- **What a reviewer must know:**
  - Controllers must be thin: validation + delegation only, no business logic.
  - Entities are never serialized to API responses — always map to a DTO (MapStruct).
  - All public endpoints are secured; a new endpoint without an auth rule is a defect.
  - Money is `BigDecimal`, never `double`/`float`.
  - All external (Feign/Kafka/DB) calls must have timeout + failure handling.

## 2. Severity levels (with a concrete example each)

| Level | Definition | Concrete example | Action |
|-------|------------|------------------|--------|
| **Critical** | Data loss or security vulnerability | `@Query("SELECT u FROM User u WHERE u.email = '" + email + "'")` (string-concatenated query); or a new `@GetMapping` reachable without any `@PreAuthorize`/security rule | Report |
| **High** | Functional bug affecting users | `orderRepository.findById(id).get()` without `isPresent()` → `NoSuchElementException` 500s the request; or `total` computed as `double` for currency | Report |
| **Medium** | Quality issue likely to cause a future bug | `catch (Exception e)` swallowing all exceptions instead of the specific one; Feign call with no `@CircuitBreaker`/timeout; `@Transactional` missing on a multi-write service method | Report |
| **Low** | Style / convention / formatting | Field ordering, missing Javadoc, `import` ordering, line length, var naming | **Never report** — Checkstyle/Spotless/SonarLint handle this | 

## 3. INCLUDE — report these

- **SQL/JPQL injection** — any query built by string concatenation with caller input instead of a bound parameter (`:param`).
- **Unsecured endpoint** — a new/changed `@*Mapping` with no method/class security annotation and not on the documented public allowlist.
- **Unhandled `Optional`** — `.get()` / `.orElseThrow()`-free dereference that can throw on a normal not-found path.
- **Money as floating point** — `double`/`float` (or `Double`/`Float`) used for monetary amounts; must be `BigDecimal`.
- **Missing transaction boundary** — service method doing 2+ writes without `@Transactional`.
- **Resilience gap** — Feign/REST/Kafka call with no timeout and no circuit breaker / fallback.
- **Broad exception swallowing** — `catch (Exception|Throwable)` that logs-and-continues, hiding failures.
- **Entity leakage** — JPA entity returned directly from a controller (must be a DTO).
- **N+1 query** — lazy association accessed in a loop without a fetch join / `@EntityGraph`.
- **Secret/PII in logs** — logging tokens, passwords, full card/PII fields.
- **Null contract violation** — returning `null` from a method whose callers assume non-null (prefer `Optional`/empty collection).

## 4. EXCLUDE — never report these

- Formatting, import order, line length, brace style — **Spotless/Checkstyle** own these.
- Missing Javadoc or comments.
- Lombok vs explicit getters/setters preference.
- Naming conventions / variable names.
- Test naming style.
- Anything SonarLint/the IDE already flags as a style smell.
- Subjective "this could be cleaner" with no concrete bug or risk.

## 5. Per-category examples (anchors)

### Unhandled Optional
```text
BAD:  Order o = orderRepository.findById(id).get();
WHY:  Throws NoSuchElementException → unhandled 500 when the order doesn't exist.
GOOD: Order o = orderRepository.findById(id)
          .orElseThrow(() -> new OrderNotFoundException(id));
```

### Money as floating point
```text
BAD:  double total = price * quantity;
WHY:  Binary floating point rounds money incorrectly (0.1 + 0.2 != 0.3).
GOOD: BigDecimal total = price.multiply(BigDecimal.valueOf(quantity));
```

### Resilience gap on a Feign call
```text
BAD:  PaymentResponse r = paymentClient.charge(req);   // no timeout, no breaker
WHY:  A slow/hung payment service cascades latency and exhausts threads.
GOOD: @CircuitBreaker(name="payment", fallbackMethod="chargeFallback")
      PaymentResponse charge(...) { ... }   // + connect/read timeout configured
```

### Broad exception swallowing
```text
BAD:  try { ... } catch (Exception e) { log.error("failed", e); }   // continues anyway
WHY:  Hides the real failure; caller proceeds on bad state.
GOOD: catch the specific exception and either recover meaningfully or rethrow.
```

## 6. Project-specific rules / known false-positive traps

- **Do NOT flag** `@Transactional(readOnly = true)` on single-read query methods as a "missing transaction" issue — it's intentional and correct here.
- **Do NOT flag** the three documented public endpoints (`/actuator/health`, `/actuator/info`, `/v3/api-docs`) as unsecured — they are intentionally open.
- **Always flag** any direct `new RestTemplate()` usage — this service standardized on Feign clients; ad-hoc RestTemplate bypasses our resilience config.
- **Always flag** `@Autowired` field injection in new code — constructor injection is the standard (testability + immutability).
