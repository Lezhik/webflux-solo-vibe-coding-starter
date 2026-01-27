# 09 – Java Style Guide – WebFlux & Semantic Coding  
**File Path:** rules/03-coding-standards/09-java-style.md  
**Domain:** TODO: [Business Context – e.g., "Reactive payment processing backend"]  
**Last Updated:** 2026-01-23  
**Status:** Draft  

## 🎯 Purpose  
Establish reactive coding standards for Java/Spring WebFlux that enforce semantic naming, non-blocking patterns, and alignment with frontend TypeScript components through strict DTO contracts. This guide promotes "vibe coding" by ensuring code intent is instantly readable, with mandatory annotations for LLM-assisted parsing and cross-stack consistency.

## 📌 Scope  
- **Applies to:** All Java modules using Spring WebFlux (Mono/Flux), MongoDB repositories, controllers, services, and DTOs in the `backend/` directory.  
- **Excludes:** Blocking JDBC/JPA code, legacy servlet-based components, third-party library internals, and non-reactive test utilities.  
- **Dependencies:** Naming rules from `06-naming-conventions.md`, semantic tags from `07-semantic-comments.md`, modular structure from `04-modular-architecture.md`, and TypeScript alignment from `10-typescript-style.md`.  

## 🧠 Semantic Anchors  

### 1. Reactive Naming DNA  
- **Why:** Communicate execution model (reactive vs. blocking) through method names to prevent ambiguity and enforce non-blocking intent.  
- **How:** Adopt a Verb-Domain-ReactiveType pattern for all public methods. Use suffixes like `Async`, `Flux`, or `Mono` to signal reactivity. Deprecate blocking alternatives.  
  ```java
  // Correct: Clear reactive intent with suffix and semantic annotation
  @Semantic("PaymentStream")
  @Vibe("Emits real-time payment events with backpressure handling")
  public Flux<PaymentEvent> streamPendingPaymentsAsync(String merchantId) {
      return paymentRepo.findPendingByMerchant(merchantId)
          .delayElements(Duration.ofMillis(100));
  }

  // Anti-pattern: Ambiguous blocking name – deprecate and redirect
  @Deprecated(since = "2.0", forRemoval = true, because = "Use reactive alternative: streamPendingPaymentsAsync")
  public List<Payment> getPayments(String merchantId) { ... }
  ```

### 2. Annotation Taxonomy  
Mandatory metadata for semantic clarity, LLM parsing, and vibe alignment:  
- `@Semantic`: Describes the core purpose or layer (e.g., "AggregateRoot", "DataflowBoundary").  
- `@Vibe`: Captures the "feel" or rationale (e.g., "Ensures atomic processing with rollbacks").  

  ```java
  @Vibe("Ensures atomic payment processing with rollbacks on failures")
  @Semantic("AggregateRoot")
  public class PaymentProcessor {
      @Semantic("DataflowBoundary")
      public Mono<PaymentResult> processPaymentAsync(PaymentIntent intent) {
          // Reactive pipeline logic
      }
  }
  ```

### 3. Code Section Ordering  
Standardize structure for scannability:  
1. **Imports** (grouped: Java stdlib > Spring Boot/WebFlux > Project-specific > Third-party; static imports first).  
2. **Constants** (static final fields, prefixed as `PUBLIC_CONSTANT` or `PRIVATE_CACHE_KEY`).  
3. **DTO/Entity/Record definitions** (with `@Semantic` where applicable).  
4. **Reactive service methods** (grouped by dataflow: e.g., streams before fetches).  
5. **WebFlux handlers and configuration** (controllers last).  

## 🧠 Vibe Coding Imperatives  
- **3-Second Intent Test:** Every public method name must instantly reveal its async/reactive nature and high-level action (e.g., `publishPaymentStreamFlux` > `handlePayments`).  
- **Layer Isolation:** Controllers must delegate entirely to services – no business logic or direct repo access in handlers.  
- **DTO Mirroring:** Java DTOs (prefer `record` types) must match TypeScript interfaces 1:1, including field names, types, and nullability. Use MapStruct for entity-to-DTO mapping.  
- **Reactive Purity:** Zero blocking calls in pipelines; use BlockHound for runtime detection. Prefer `Mono<Void>` for terminal ops like deletions.  

**Anti-Patterns:**  
❌ `public List<User> findAll()` (blocking return; violates reactivity).  
✅ `public Flux<User> streamActiveUsersAsync()` (signals stream, non-blocking).  
❌ Raw MongoDB documents exposed in responses (leaks internal structure).  
✅ `Flux<UserDto> streamUsers() .map(UserMapper::toDto)` (explicit, secure mapping).  

## ⚙️ Enforcement Mechanics  

| Rule Type         | Tool/Mechanism                     | Failure Action            |  
|-------------------|------------------------------------|---------------------------|  
| Reactive Naming   | Custom Checkstyle Rule             | Build failure             |  
| Annotation Checks | ArchUnit Tests                     | CI Pipeline Block         |  
| DTO Contracts     | JSON Schema Validation + MapStruct | Test Phase Failure        |  
| Reactive Purity   | BlockHound Integration             | Runtime Detection/Warning |  
| Layer Isolation   | Spring Context Segmentation        | Startup Failure           |  

**Gradle Snippet (build.gradle.kts):**  
```kotlin
checkstyle {
    configFile = file("config/checkstyle/semantic-checks.xml")
    maxWarnings = 0
}

tasks.register("validateReactiveStyle") {
    dependsOn("checkstyleMain", "test")
    doLast {
        exec { commandLine("java", "-jar", "blockhound.jar", "--check") }
        exec { commandLine("java", "-jar", "semantic-linter.jar") }
    }
}

tasks.named("check") { dependsOn("validateReactiveStyle") }
```

**Checkstyle Config Example (semantic-checks.xml):**  
```xml
<module name="Regexp">
    <property name="format" value="public (Mono|Flux)&lt;.*&gt; \w+(Async|Flux|Mono)\(\)"/>
    <property name="message" value="Reactive methods must use Async/Flux/Mono suffixes"/>
</module>
```

## 🔧 Technical Specifications  

### WebFlux Handler Standards  
Handlers focus on routing and DTO transformation; no orchestration.  
```java
@RestController
@RequestMapping("/api/payments")
@Semantic("PaymentAPI")
@Vibe("Reactive endpoints for live payment streams")
public class PaymentController {
    private final PaymentService service;
    
    @GetMapping("/stream")
    public Flux<PaymentDto> streamPayments(@RequestParam String userId) {
        return service.streamUserPaymentsAsync(userId)
            .map(PaymentMapper::toDto)
            .timeout(Duration.ofSeconds(30));
    }

    @PostMapping
    public Mono<PaymentResult> createPayment(@RequestBody PaymentIntent intent) {
        return service.processPaymentAsync(intent)
            .map(PaymentMapper::toResultDto);
    }
}
```

### Reactive Service Patterns  
Build composable pipelines with error resilience.  
```java
@Service
@RequiredArgsConstructor
@Semantic("PaymentPipeline")
public class PaymentService {
    private final PaymentRepository repo;
    private final FraudCheckClient fraudClient;

    @Vibe("Transforms input stream with fraud screening and persistence")
    public Flux<Payment> processPaymentsAsync(Flux<PaymentRequest> requests) {
        return requests
            .flatMap(this::validateRequestAsync)
            .transform(fraudClient::screenPaymentsFlux)
            .flatMap(this::persistPaymentAsync)
            .onErrorContinue((req, err) -> log.warn("Payment failed: {}", req, err));
    }

    private Mono<Payment> validateRequestAsync(PaymentRequest req) {
        return Mono.just(req).doOnNext(this::checkValidity);
    }
}
```

### DTO Example (Using Records for Immutability)  
```java
public record PaymentDto(
    @Semantic("UUIDv7 Identifier") String id,
    @Vibe("Amount in cents to prevent floating-point issues") long amountCents,
    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss") Instant createdAt
) {}
```

## 📋 Compliance Checklist  
- [ ] All async methods use `Flux`/`Mono` suffixes (e.g., `stream*Async`).  
- [ ] `@Semantic` and `@Vibe` annotations on all public classes/methods.  
- [ ] DTOs mirror frontend TypeScript interfaces exactly (validate via schema).  
- [ ] No blocking calls in reactive chains (BlockHound passes).  
- [ ] MapStruct mappers used for all entity-DTO transformations.  
- [ ] Code sections ordered as specified.  
- [ ] Checkstyle and ArchUnit tests pass with zero violations.  

## 🔗 Interlocks  
- **Frontend:** TypeScript DTOs and streams (`10-typescript-style.md`).  
- **Build:** Gradle multi-module validation (`22-gradle-multimodule.md`).  
- **Tests:** Reactive testing with StepVerifier (`17-test-pyramid.md`).  
- **Reviews:** Mandatory checklist in PRs (`16-review-checklist.md`).  

## 📜 Revision History  
| Version | Date       | Author        | Changes Summary                                     |  
|---------|------------|---------------|-----------------------------------------------------|  
| 1.0     | 2026-01-23 | LLM-Generated | Initial draft with WebFlux focus and vibe semantics |  

**Implementation Notes:**  
1. Replace all TODOs with project-specific details (e.g., domain context).  
2. Implement custom Checkstyle/ArchUnit rules in the repo.  
3. Integrate `validateReactiveStyle` task into CI/CD pipelines.  
4. Before "Active" status, run full validation and update revision history.  

This document ensures reactive integrity, semantic clarity, and seamless backend-frontend harmony in the Kilo Code project.