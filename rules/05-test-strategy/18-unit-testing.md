# 18 – Unit Testing Standards

**File Path:** rules/05-test-strategy/18-unit-testing.md  
**Domain:** TODO: <business-context> (e.g., "Secure e-commerce payment processing with real-time event streaming")
**Last Updated:** 2026-01-24
**Status:** Active  

<!-- FILE_TITLE: Unit Testing Standards -->  
<!-- FILE_PATH: rules/05-test-strategy/18-unit-testing.md -->

---

## 🎯 Purpose  
To define semantic-driven unit testing practices that ensure deterministic verification of business logic, maintain modular isolation, preserve reactive stream integrity, and promote intent clarity across Java/WebFlux services and TypeScript/Vue components. This approach facilitates automated test gap analysis and aligns with the overall test pyramid strategy (e.g., 70% unit coverage).

## 📌 Scope  
- **Applies to:**  
  - Java service methods handling Mono/Flux reactive streams.  
  - TypeScript logic in Vue components.  
- **Excludes:**  
  - Database integrations, HTTP client interactions, end-to-end (E2E) tests, and performance testing (covered in 19-e2e-testing.md and 21-performance-testing.md).  
- **Dependencies:**  
  - [06-naming-conventions.md](#) for naming alignment.  
  - [09-java-style.md](#) for reactive patterns.  
  - [20-test-data-profiles.md](#) for data sourcing.  
  - [17-test-pyramid.md](#) for coverage ratios.  

---

## 🧠 Semantic Anchors  

### Naming DNA  
Adopt intent-driven naming to make test intent self-documenting:  
1. **Java Tests**: Follow `methodName_shouldActionWhenCondition_ExpectedResult()` pattern.  
   ```java 
   @Test
   void calculateTotal_shouldApplyDiscountWhenEligible_returnsDiscountedTotal() {
       // Test implementation
   }
   ```  
2. **Reactive Tests**: Append `Async` suffix for streams and explicitly handle completion/error states.  
   ```java
   @Test
   void streamOrdersAsync_shouldEmitValidatedEvents_whenUserAuthenticated() {
       // Reactive test with StepVerifier
   }
   ```  
3. **TypeScript/Vue Tests**: Use descriptive phrases like `componentAction_WhenCondition_ExpectedBehavior`.  
   ```typescript
   it('displaysWarning_WhenInventoryLow', () => {
       // Test implementation
   });
   ```  

### Annotation Taxonomy  
- **`@Semantic("DomainRole")`**: Tags the business domain (e.g., `@Semantic("PaymentValidation")`).  
- **`@Vibe("IntentRationale")`**: Captures behavioral intent (e.g., `@Vibe("Ensures backpressure compliance in reactive flows")`).  
- **`@ActiveProfiles("unit")`**: Activates test-specific profiles for isolated execution.  

**Example Usage:**  
```java
@Test
@Semantic("OrderProcessing")
@Vibe("Validates event emission under load")
@ActiveProfiles("unit")
void publishOrderAsync_shouldCompleteSuccessfully_whenOrderValid() {
    // Test body
}
```

### Code Structure Guidelines  
1. **Imports**: Group as core (e.g., JUnit) > project-specific > third-party.  
2. **Constants**: Prefix with `TEST_` (e.g., `private static final String TEST_ORDER_ID = "123";`).  
3. **Test Organization**: Arrange methods by execution flow: setup > happy path > edge cases > error handling.  
4. **Avoidances**: No `Thread.sleep()`, inline data generation, or blocking calls in reactive tests.

---

## ⚙️ Enforcement Mechanics  

| Rule Type              | Tool/Mechanism                  | Action on Failure                 |
|------------------------|---------------------------------|-----------------------------------|
| Naming Patterns        | Checkstyle (Java) / ESLint (TS) | Block CI pipeline / Auto-reformat |
| Annotation Presence    | Custom Gradle task              | Fail build / IDE warnings         |
| Reactive Compliance    | StepVerifier integration        | Report errors in reports          |
| Test Data Sourcing     | Profile analyzer script         | Reject inline data / Warn         |
| Coverage Thresholds    | SonarQube / JaCoCo              | Block merge if <70% unit coverage |

**Gradle Enforcement Snippet** (build.gradle.kts):  
```kotlin
tasks.register("validateUnitTests") {
    dependsOn("test")
    doLast {
        exec {
            commandLine("./scripts/check-test-semantics.sh", "java", "typescript")
        }
    }
}
tasks.register("verifyUnitSemantics") {
    dependsOn(":backend:test", ":frontend:test")
    doLast {
        exec { commandLine("node", "scripts/validate-annotations.js") }
    }
}
```

**Pre-Commit Hook Integration:** Run `./gradlew validateUnitTests` to enforce before git commit.

---

## 🔧 Technical Specifications  

### Java/WebFlux Testing  
1. **Reactive Stream Validation**: Mandate `StepVerifier` for Mono/Flux assertions, including backpressure and termination checks.  
   ```java
   @Test
   @Semantic("PaymentProcessing")
   @Vibe("Guarantees monetary safety boundaries")
   void authorizePaymentAsync_shouldEmitDeclined_whenBalanceInsufficient() {
       PaymentRequest request = TestDataFactory.load("payment_edge_case.yml");
       
       StepVerifier.create(paymentService.authorize(request))
                   .expectNextMatches(result -> result.getStatus() == PaymentStatus.DECLINED)
                   .verifyComplete();  // Or verifyError() for failure paths
   }
   ```  
2. **Mocking Strategy**: Use `@MockBean` for dependencies; avoid mocking reactive repositories—use test profiles instead.  
3. **Edge Cases**: Cover null inputs, invalid states, and timeouts with explicit `@Vibe` annotations.  

### TypeScript/Vue Testing  
1. **Component Isolation**: Test with `mount()` or `render()` using typed props from profiles.  
   ```typescript
   it('displaysLoadingIndicator_WhenTransactionPending', async () => {
       const testData = loadTestProfile('transaction_pending.json');
       const wrapper = mount(TransactionComponent, {
           props: { 
               transactionState: testData.state,
               error: testData.error 
           }
       });
       
       await wrapper.vm.$nextTick();
       expect(wrapper.find('.loading-indicator').exists()).toBe(true);
   });
   ```  
2. **Prop Validation**: Strictly assert on prop types and reactivity.  
3. **Async Handling**: Use `await` for Vue lifecycle and avoid global mocks.  

**Key Innovation:** Integrate `@dataflow` pseudo-annotations (e.g., `@dataflow("Order -> Payment")`) for generating automated dependency graphs in IDEs or CI tools.

---

## 📋 Compliance Checklist  
- [ ] All test methods use intent-driven naming patterns (e.g., `shouldActionWhenCondition`).  
- [ ] Reactive Java tests employ `StepVerifier` with `.verifyComplete()` or `.verifyError()`.  
- [ ] Every test class/method includes `@Semantic` and `@Vibe` annotations.  
- [ ] Test data is loaded exclusively from profiles (e.g., YAML/JSON in `/src/test/resources/profiles/`)—no inline generation.  
- [ ] Zero blocking calls (`Thread.sleep()`, `CountDownLatch`); use virtual time where needed.  
- [ ] Unit test coverage meets 70% threshold per module (enforced by JaCoCo).  
- [ ] Vue tests validate UI behavior with strict prop assertions.  

---

## 🔗 Interlocks  
- **Naming Conventions:** [06-naming-conventions.md](#) for cross-language consistency.  
- **Reactive Patterns:** [09-java-style.md](#) for WebFlux compliance.  
- **Test Data Management:** [20-test-data-profiles.md](#) for standardized fixtures.  
- **Test Pyramid Alignment:** [17-test-pyramid.md](#) for strategy integration.  
- **Build and Reviews:** [22-gradle-multimodule.md](#) and [16-review-checklist.md](#) for CI/CD hooks.  
- **Versioning:** [30-semantic-versioning.md](#) for test evolution tracking.  

---

## 📜 Revision History  
| Version | Date       | Author          | Changes Summary                              |
|---------|------------|-----------------|----------------------------------------------|
| 1.0     | 2026-01-24 | LLM-Generated   | Initial draft with semantic anchors.         |

---

## Implementation Guide  
1. **Setup Test Data Directory:** Create `/src/test/resources/profiles/` for YAML/JSON fixtures (e.g., `order_edge_cases.yml`, `payment_invalid.json`). Use a factory like `TestDataFactory` to load them.  
2. **Annotate and Profile:** Apply `@ActiveProfiles("unit")` to all test classes; ensure `@Semantic` and `@Vibe` on methods.  
3. **Run Validations:** Integrate `./gradlew validateUnitTests` and `./gradlew verifyUnitSemantics` into pre-commit hooks and PR pipelines.  
4. **Tooling Enhancements:** Configure IDEs (e.g., IntelliJ, VS Code) for auto-suggestions on naming and annotations.  
5. **Gap Analysis:** Leverage semantic tags for AI-driven tools to scan and suggest missing test scenarios.  

**Differentiator:** This standard introduces `@Vibe` annotations for human-AI collaboration, enabling vibe-consistent code generation while enforcing reactive best practices. Replace all `TODO` placeholders before production rollout.
