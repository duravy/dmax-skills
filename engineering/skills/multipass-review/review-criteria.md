# Code Review Criteria — Spring Boot Admin (full-stack monorepo)

> **How this file is used:** the `multipass-review` skill embeds this file verbatim into
> every fresh review agent's prompt. Vague criteria → false positives → developers stop
> trusting *all* findings. Be **categorical**, not subjective. Define what to **skip** as
> explicitly as what to **flag**.
>
> This repo is full-stack: a **reactive Java backend** (`spring-boot-admin-server`,
> `-client`, `-cloud`) and a **Vue 3 + TypeScript frontend** (`spring-boot-admin-server-ui`).
> Apply the Java rules (§3a/§5a) to `*.java` and the frontend rules (§3b/§5b) to
> `*.vue`/`*.ts`. Skip the irrelevant half for a given file.

## 1. Tech stack & context

- **Backend:** Java 17, Spring Boot 4.1, Maven. **Fully reactive** (Project Reactor —
  `Mono`/`Flux`, Spring WebFlux) with a parallel servlet (Spring MVC) bridge. Lombok for
  data classes, JSpecify `@Nullable` for nullability, SLF4J for logging.
- **Frontend:** Vue 3 + TypeScript + Vite + Tailwind v4, in `spring-boot-admin-server-ui/src/main/frontend`.
  Mid-migration from Options API → `<script setup>`. State shared via module-singleton
  composables (no Pinia/Vuex). All HTTP through a shared Axios instance (`utils/axios.ts`).
  Vitest + `@testing-library/vue` for tests.
- **Architecture:** The server registers client instances, aggregates actuator data, and
  proxies actuator calls. Controllers are reactive and route through a custom
  `@AdminController` handler mapping. The UI is bundled into the server JAR.
- **What a reviewer must know (key invariants):**
  - **Backend is reactive-first.** Blocking a `Mono`/`Flux` (`.block()`) is a defect
    *except* the one documented call in `servlet/InstancesProxyController.java`.
  - Controllers use `@AdminController` (a marker meta-annotation), **never** `@RestController` —
    a `@RestController` route is invisible to the admin context-path handler mapping.
  - DI is **constructor injection only**. There is zero `@Autowired`/`@Inject` in the source.
  - All proxy/actuator calls go through `InstanceWebProxy`, which centralizes timeout/IO
    error mapping. Calling `InstanceWebClient`/`WebClient` directly skips that.
  - Frontend: all user-facing text goes through vue-i18n `$t()`/`t()`; all HTTP through the
    shared `utils/axios` instance; shared cross-view state via `use*` module-singleton
    composables that **replace** (not mutate) reactive `Set`/`Map` values.

## 2. Severity levels (with a concrete example each)

| Level | Definition | Concrete example | Action |
|-------|------------|------------------|--------|
| **Critical** | Data loss or security vulnerability | Logging the `BulkActionRequest.payload` map (arbitrary user values) unsanitized; serializing instance `metadata` to a response without sanitization; a new endpoint leaking credentials/tokens | Report |
| **High** | Functional bug affecting users | `.block()` on a `Mono` in a reactive controller (thread starvation/deadlock); returning `null` from inside a `flatMap`/`map` (NPE at subscription); mutating a `ref(Set)` in place so Vue never re-renders | Report |
| **Medium** | Quality issue likely to cause a future bug | New `@RestController` instead of `@AdminController`; calling `InstanceWebClient`/`WebClient` directly instead of via `InstanceWebProxy`; outermost `onErrorResume` swallowing errors with no log/result; hardcoded user-facing UI string not via `$t()` | Report |
| **Low** | Style / convention / formatting | Java/import ordering, brace style, Lombok-vs-explicit getters, `LOGGER` vs `log` field name, Prettier/ESLint formatting | **Never report** — Spring Java Format + Checkstyle (Java) and ESLint + Prettier (UI) handle this |

## 3a. INCLUDE — report these (Java backend)

- **Blocking a reactive type** — any `.block()`/`.blockFirst()`/`.blockLast()` outside the one
  documented call in `servlet/InstancesProxyController.java`.
- **`null` inside a Reactor operator** — returning `null` from `map`/`flatMap`/`switchIfEmpty`
  (use `Mono.empty()`/`Mono.just(...)` instead) → NPE at subscribe time.
- **`@RestController` usage** — must be `@AdminController` + class-level `@ResponseBody`.
- **Bypassing `InstanceWebProxy`** — new actuator/proxy calls that hit `InstanceWebClient` or a
  raw `WebClient` directly and re-implement (or omit) timeout/IO error mapping.
- **Silent error swallowing in reactive chains** — outermost-level `onErrorResume` that neither
  logs nor maps to a meaningful response/domain result.
- **`Mono.error()` inside `flatMap` with no downstream handler** — terminates the whole sequence.
- **`@Autowired`/`@Inject` / field injection** — codebase is 100% constructor injection.
- **New `BulkAction` enum value with no matching `switch` case** — relies on exhaustive switch.
- **Shared mutable non-concurrent state in background tasks** — plain `HashMap`/`ArrayList`
  written from multiple threads (existing pattern uses `ConcurrentHashMap`/single-thread schedulers).
- **Sensitive data in logs** — logging `metadata`, `payload`, registration credentials, or tokens.
- **Undisposed schedulers/subscriptions** — a `Schedulers.new*`/`subscribe()` with no paired
  `dispose()`/`stop()` (existing background beans dispose in `stop()`).

## 3b. INCLUDE — report these (Vue / TypeScript frontend)

- **In-place mutation of a reactive `Set`/`Map`** — `selectedIds.value.add(id)` instead of
  reassigning `selectedIds.value = new Set(...)`; breaks Vue reactivity on `ref(Set)`.
- **Hardcoded user-facing text** — visible label/button/heading/status/error/confirm rendered
  as a string literal instead of `$t()`/`t()` (placeholders/sample hints are exempt — see §6).
- **i18n key with no entry** — a `$t('...')` key not added to the co-located `i18n.en.json`.
- **Bypassing shared axios** — `axios.create()` without re-applying `addLanguageHeaderInterceptor`
  + `redirectOn401()`, or using raw `fetch`; loses CSRF/credentials/401-redirect/Accept-Language.
- **Unstable `v-for` key** — `:key="index"` on a list of domain objects (instances/applications)
  that can reorder/filter; must key on a stable id (`:key="instance.id"`).
- **Composable not following the singleton/return contract** — shared state declared inside the
  function body instead of module scope (state no longer shared), or filename/export not `use*`.
- **`any` in new service/type signatures** — new service code is fully typed (`bulk-action.ts`
  is the precedent); avoid `any` where an interface is feasible.
- **Prop mutation** — mutating a prop directly instead of emitting `update:modelValue`.
- **Interactive element with no accessible name** — button/input/control with no `aria-label`
  or associated label (also breaks the `getByRole`/`getByLabelText` test convention).

## 4. EXCLUDE — never report these

- **Java formatting / imports / braces / line length** — Spring Java Format + Checkstyle own these.
- **`LOGGER` vs `log` logger field name**, and `@lombok.Foo` vs `import lombok.Foo` style — both
  coexist intentionally across the codebase.
- **Lombok-generated getters/setters** vs hand-written — `@Data`/`@Value` is the norm.
- **Missing Javadoc / comments.**
- **Frontend formatting / quote style / semicolons** — ESLint + Prettier own these.
- **Existing Options API components** not touched by the change — the Options→`<script setup>`
  migration is intentionally incomplete.
- **`@ts-ignore`/`as any` in `.spec.ts`** when mocking composables via `vi.mock` — established
  test pattern.
- **No explicit HTTP timeout on axios calls / no JSR-303 validation** — neither is used anywhere
  here; absence is the baseline, not a defect.
- **Method-level Spring Security annotations being absent** — security is configured at
  infrastructure level, not via `@PreAuthorize`. (Do flag if someone *introduces* them.)
- Subjective "this could be cleaner" with no concrete bug or risk.

## 5a. Per-category anchors (Java)

### Blocking a reactive type
```text
BAD:  BulkActionResponse r = bulkActionFlux.collectList().block();
WHY:  Blocks a Reactor/event-loop thread; the one sanctioned .block() is the documented
      servlet/InstancesProxyController.java header-commit case — nowhere else.
GOOD: return bulkActionFlux.collectList().map(BulkActionResponse::of);
```

### null inside a Reactor operator
```text
BAD:  .flatMap(instance -> { if (notFound) return null; ... })
WHY:  Reactor throws NullPointerException at subscription — operators must return a Publisher.
GOOD: .switchIfEmpty(Mono.just(BulkActionResult.notFound(id)))
```

### Wrong controller annotation / bypassing the proxy
```text
BAD:  @RestController class FooController { WebClient wc; ... wc.get()... }
WHY:  @RestController routes are invisible to AdminControllerHandlerMapping; direct WebClient
      skips InstanceWebProxy's timeout/IO error mapping.
GOOD: @AdminController @ResponseBody class FooController {
        FooController(InstanceWebClient c){ this.proxy = new InstanceWebProxy(c); } }
```

## 5b. Per-category anchors (Vue / TS)

### In-place reactive Set mutation
```text
BAD:  selectedIds.value.add(id);          // ref(Set) internals are not reactive
WHY:  Vue only tracks reassignment of .value, so the UI never updates.
GOOD: selectedIds.value = new Set(selectedIds.value).add(id);
```

### Hardcoded user-facing text
```text
BAD:  <button>Deregister</button>
WHY:  Bypasses vue-i18n; untranslatable. (placeholder="de.codecentric" sample hints are exempt.)
GOOD: <button>{{ $t('applications.bulk.deregister') }}</button>   // + key in i18n.en.json
```

### Bypassing the shared axios instance
```text
BAD:  fetch('/instances/bulk-action', { method: 'POST', ... })
WHY:  Skips CSRF header, withCredentials, 401-redirect, and Accept-Language interceptors.
GOOD: import axios from '../utils/axios'; axios.post('instances/bulk-action', request);
```

## 6. Project-specific rules / known false-positive traps

- **Do NOT flag** the single `.block()` in `servlet/InstancesProxyController.java` — its comment
  documents why headers must commit synchronously on the servlet stack.
- **Do NOT flag** notifiers wrapping a blocking `RestTemplate` call inside `Mono.fromRunnable()`
  (e.g. `SlackNotifier`, `MailNotifier`) — that is the established notifier bridge pattern.
- **Do NOT flag** the imperative `validate()`-returns-`String` pre-check in `BulkActionController`
  / `NotificationFilterController` as "should use `@Valid`/`Assert`" — `Assert` is reserved for
  domain-object constructors; controllers validate imperatively here by convention.
- **Do NOT flag** `BulkActionRequest` having mutable `@Data`/`@NoArgsConstructor` fields — it is a
  Jackson deserialization target and must be mutable.
- **Do NOT flag** `new InstanceWebProxy(instanceWebClient)` constructed inline in a controller —
  both proxy controllers do exactly this; it is the accepted pattern, not a testability issue.
- **Do NOT flag** missing method-level security annotations, missing axios timeouts, or
  `@ts-ignore`/`as any` inside `.spec.ts` — all are intentional baselines (see §4).
- **Do NOT flag** the wallboard's non-scoped `<style>` blocks or `<script setup>` without
  `lang="ts"` on existing components — pre-existing; only flag in new typed-data components.
- **Always flag** any new `.block()`, `@RestController`, `@Autowired`, or direct
  `RestTemplate`/`WebClient`/`fetch` usage in new code — these contradict firm conventions.
- **Always flag** a new `BulkAction` enum constant without a matching `switch` branch in
  `BulkActionController` (the exhaustive switch is the safety net).
