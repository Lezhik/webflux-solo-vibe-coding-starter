# 21 – Boundary Scenarios – Testing Edge Cases

**File Path:** rules/05-test-strategy/21-boundary-scenarios.md  
**Domain:** TODO: <business-context> (e.g., "Secure e-commerce payment processing with real-time event streaming")
**Last Updated:** 2026-01-24
**Status:** Active

<!-- FILE_TITLE: Boundary Testing Scenarios -->  
<!-- FILE_PATH: rules/05-test-strategy/21-boundary-scenarios.md -->  

## 🎯 Purpose  
Define standardized templates and patterns for boundary testing to validate edge cases, failure modes, and resilience in reactive WebFlux pipelines (Java services) and Vue components. Use semantic annotations like `@Semantic` and `@Vibe` to embed intent, emotional context for failures, and cross-stack alignment, ensuring tests are clear, maintainable, and focused on qualitative validation rather than metrics.

## 📌 Scope  
- **Applies to:** Unit, integration, and end-to-end tests for Java/WebFlux services (Mono/Flux streams) and Vue components (props, inputs, event handling).  
- **Excludes:** Happy-path tests; manual exploratory testing; performance benchmarking.  
- **Dependencies:** [18-unit-testing.md](../18-unit-testing.md) for test structure; [20-test-data-profiles.md](../20-test-data-profiles.md) for edge-case data profiles.  

---

## 🧠 Semantic Anchors  
Semantic annotations and naming conventions clarify test intent and failure reasoning, promoting "vibe coding" for emotional and technical resonance.  

1. **Annotation Taxonomy**  
   - `@Semantic("BoundaryScenario")`: Tags tests or methods focused on edge cases (e.g., nulls, overflows, timeouts).  
   - `@Vibe("...")`: Captures the "why" behind failure handling (e.g., `@Vibe("Graceful degradation to prevent user frustration")` for empty streams).  

2. **Naming Patterns**  
   - **Backend (Java/WebFlux):** `test<BoundaryType><Domain>Scenario()` e.g., `testNullInputPaymentValidationBoundary()`.  
     ```java  
     @Test  
     @Semantic("BoundaryScenario")  
     @Vibe("Rejects invalid inputs to safeguard transaction integrity")  
     void testNegativeAmountPaymentBoundary() {  
         // Assert Mono.error for negative BigDecimal in Flux pipeline  
     }  
     ```  
   - **Frontend (Vue/TypeScript):** `Guard<Domain><Boundary>` e.g., `PaymentAmountGuard`.  
     ```typescript  
     /**  
      * @Semantic("BoundaryScenario")  
      * @Vibe("Enforces non-negative values to avoid business losses")  
      */  
     const PaymentAmountGuard = (amount: number): boolean => amount >= 0;  
     ```  

3. **Data Representation**  
   - Use dedicated DTOs for boundaries (e.g., `BoundaryPaymentInput` with extreme values).  
   - Align backend DTOs (Java records) with frontend props (TypeScript interfaces) for contract testing.  

---

## 🔧 Boundary Scenario Templates  
Populate these tables during feature development with domain-specific examples. Each row represents a testable edge case, emphasizing root causes and cross-stack implications. Aim for 3+ scenarios per feature.  

### 1. Data Validation Boundaries (Inputs: nulls, formats, constraints)  
| Input (Type / Value)                               | Expected Output                                 | Why Fail (Root Cause)                             | Cross-Stack Notes                                                   |  
|----------------------------------------------------|-------------------------------------------------|---------------------------------------------------|---------------------------------------------------------------------|  
| Null userId (String)                               | Mono.error(IllegalArgumentException)            | Required field missing; domain contract violation | Vue: Prop guard in defineProps() rejects null; disable form submit  |  
| Empty password (String)                            | Flux.empty() or Mono.error(ValidationException) | Empty value rejected by auth rules                | TS: `<input required>` + client-side check before API call          |  
| Negative paymentAmount (BigDecimal, e.g., -10.00)  | Mono.error(NegativeValueException)              | Business rule: amounts must be positive           | Vue: Numeric input with `min="0"` attribute and reactive validation |  
| Oversized payload (>1MB, byte[])                   | Mono.error(DataSizeExceededException)           | Exceeds request body limits                       | Frontend: File upload guard truncates or alerts before submission   |  
| Invalid email format (e.g., "test@invalid..com")   | Mono.empty() (no entity created)                | Regex/pattern mismatch in validation layer        | TS: Email regex validator in Vue form; real-time feedback           |  
| TODO: [Domain-specific, e.g., max-length username] | Flux.error(StringLengthException)               | Exceeds DB schema constraints                     | Vue: Input maxlength enforcement + char counter UI                  |  

**Backend Test Example (Parameterized JUnit):**  
```java  
@ParameterizedTest  
@ValueSource(strings = {"null", "", "test@invalid..com"})  
@Semantic("BoundaryScenario")  
@Vibe("Prevents malformed data from propagating in streams")  
void testUserInputValidationBoundaries(String input) {  
    Mono<User> result = userService.createUser(input);  
    // Assert error or empty based on input  
    StepVerifier.create(result).verifyError(IllegalArgumentException.class);  
}  
```  

### 2. Reactive Flow Boundaries (Streams: concurrency, timeouts, emptiness)  
| Scenario (Input Trigger)                              | Expected Output                                              | Why Fail (Root Cause)                             | Cross-Stack Notes                                                        |  
|-------------------------------------------------------|--------------------------------------------------------------|---------------------------------------------------|--------------------------------------------------------------------------|  
| High concurrency (100+ parallel Flux.merge requests)  | Flux.error(ConcurrencyLimitException) or backpressure signal | Scheduler/thread pool saturation                  | Vue: Throttle API calls with debounce; show loading spinner              |  
| Downstream timeout (DB query >5s)                     | Mono.error(TimeoutException)                                 | Latency exceeds `.timeout(Duration.ofSeconds(5))` | Frontend: Timeout handler in fetch() with user-friendly error message    |  
| Empty Flux source (no events emitted)                 | Flux.just(fallbackEvent) via switchIfEmpty()                 | Data source absence (e.g., empty DB query)        | TS: Default props or computed fallback UI (e.g., "No data available")    |  
| Error mid-stream (e.g., network partition in Flux)    | Flux.error(...) with onErrorResume/retry logic               | Unhandled exception in operator chain             | Vue: Error boundary component catches and retries or shows offline state |  
| TODO: [Domain-specific, e.g., rate-limited API calls] | Mono.error(RateLimitException)                               | Exceeds external service quota                    | Frontend: Retry button with exponential backoff                          |  
| Infinite stream without termination                   | Test timeout or Flux.error(IllegalStateException)            | Missing .take() or completion signal              | Vue: Watch for stream end; prevent infinite re-renders                   |  

**Frontend Test Example (Vitest):**  
```typescript  
import { describe, it, expect } from 'vitest';  
import { PaymentAmountGuard } from '@/guards';  

describe('@Semantic("BoundaryScenario")', () => {  
  it('handles negative amount boundary', () => {  
    // @Vibe("Protects against invalid financial inputs")  
    expect(PaymentAmountGuard(-10)).toBe(false);  
  });  
});  
```  

### 3. Cross-Module Integration Boundaries (Handoffs: Backend ↔ Frontend)  
| Backend Output (Response Type)                                  | Frontend Expected Handling                  | Why Fail (Root Cause)                         | Cross-Stack Notes                                                    |  
|-----------------------------------------------------------------|---------------------------------------------|-----------------------------------------------|----------------------------------------------------------------------|  
| DTO with extra/unexpected fields (e.g., JSON with legacy props) | Ignore extras (strict parsing mode)         | Schema drift between versions                 | Use Zod for runtime schema validation in TS; contract tests via Pact |  
| Mono.empty() (no data)                                          | Receive empty array; render "No results" UI | End-of-stream without emissions               | Props allow optional/empty types; default to skeleton loaders        |  
| Invalid JSON in Flux response                                   | Parsing error; trigger error boundary       | Deserialization failure (e.g., type mismatch) | Defensive parsing with try-catch in Vue; fallback to error toast     |  
| High-volume events (>1000/sec in Flux)                          | Batch and throttle updates                  | Overwhelms UI render cycle                    | Backend: Flux.buffer(); Frontend: lodash.throttle on watchers        |  
| TODO: [Domain-specific, e.g., auth token expiry mid-session]    | Session redirect or re-auth                 | Token invalidation in reactive chain          | Vuex/Pinia store with token refresh guard; route guard enforcement   |  
| Malformed error response (non-JSON)                             | Generic error display                       | API contract violation                        | Shared error DTO; TS union types for error payloads                  |  

**Integration Test Example (Spring Boot + Vue):**  
```java  
@SpringBootTest  
@ActiveProfiles("boundary-test")  
@Semantic("BoundaryScenario")  
@Vibe("Validates data handoff resilience across stacks")  
class CrossStackBoundaryIntegrationTest {  
    @Test  
    void testInvalidJsonHandoff() {  
        // Simulate malformed Flux response  
        // Assert Vue component (via WireMock or mock) handles error gracefully  
        // Why: Ensures frontend doesn't crash on backend serialization issues  
    }  
}  
```  

---

## ⚙️ Enforcement Mechanics  
| Rule Type             | Tool/Mechanism                                     | Failure Action                      | Configuration Example                                                            |  
|-----------------------|----------------------------------------------------|-------------------------------------|----------------------------------------------------------------------------------|  
| Annotation Presence   | Checkstyle + Custom Gradle Task                    | Fail build/test phase               | `checkstyle.xml`: Require @Semantic in test files; `./gradlew verifyAnnotations` |  
| Table Completeness    | Markdown linter (e.g., remark) or Script           | Block PR if tables incomplete       | GitHub Action: Scan MD for TODOs and empty rows                                  |  
| Prop/DTO Alignment    | TypeScript (strict mode) + Contract Tests          | Compile error or test fail          | `tsconfig.json`: `"strictNullChecks": true`; OpenAPI for DTO gen                 |  
| Coverage of Scenarios | JaCoCo (focus on error branches) + Custom Reporter | Warn if <80% boundary paths covered | `pom.xml` or `build.gradle`: Exclude happy paths from reports                    |  
| Vue Guards            | ESLint + Vitest                                    | Lint error; test failure            | ESLint rule: `vue/require-default-prop: error`; Parameterized Vitest for guards  |  

**CI/CD Integration:**  
- Run `./gradlew validateBoundaryScenarios` on every PR (scans for annotations, tables, and coverage).  
- Auto-generate test skeletons from tables using tools like Testify or custom scripts.  

---

## 📋 Compliance Checklist  
- [ ] All boundary tests include `@Semantic("BoundaryScenario")` and relevant `@Vibe` tags.  
- [ ] Tables are populated with at least 3 scenarios per category (data validation, reactive flow, integration).  
- [ ] Each row explains "Why Fail" qualitatively (no metrics).  
- [ ] Backend DTOs map 1:1 to frontend props/types (verified via shared schemas).  
- [ ] Vue components have TypeScript guards for inputs/streams; tests cover edge props.  
- [ ] Negative tests use profile-specific data (e.g., `edge_cases.yml`).  
- [ ] Cross-stack notes address handoff resilience.  
- [ ] No TODOs left unresolved for core project features.  

---

## 📜 Revision History  
| Version | Date       | Author             | Changes Summary                                                           |
|---------|------------|--------------------|---------------------------------------------------------------------------|
| 1.0     | 2026-01-24 | LLM-Generated      | Initial template combining prior versions; added examples and enforcement |

---

## 📌 Implementation Notes  
1. **Customization:** Replace TODOs with project details (e.g., domain like "User authentication flows"). Extend tables with feature-specific cases (e.g., for payments: currency overflow boundaries).  
2. **Vibe Coding Emphasis:** `@Vibe` tags should evoke resilience—e.g., "Prevents cascading failures to maintain user trust."  
3. **Best Practices:**  
   - Use StepVerifier for WebFlux assertions (error paths).  
   - In Vue, leverage composables for reusable guards (e.g., `useBoundaryValidation()`).  
   - Focus on decoupling: Boundary tests run in isolated profiles without external deps.  
4. **Next Steps:**  
   - Integrate with existing rules (e.g., link to [05-vibe-coding.md](../02-semantics/05-vibe-coding.md) for annotation standards).  
   - Run initial audit: Scan codebase for missing boundaries and generate stubs.  
   - Evolve: Add E2E scenarios for full-stack flows once unit coverage is solid.  

This refined template combines structured tables, concrete code examples, and enforcement from prior versions while emphasizing semantic clarity and cross-stack sync. It eliminates redundancy, adds domain-agnostic yet extensible examples, and prioritizes qualitative failure reasoning for reactive systems.
