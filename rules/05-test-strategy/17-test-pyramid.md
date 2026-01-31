# 17 – Test Pyramid Strategy  
**File Path:** rules/05-test-strategy/17-test-pyramid.md  
**Domain:** TODO: <business-context> (e.g., "Secure e-commerce payment processing with real-time event streaming")
**Last Updated:** 2026-01-24
**Status:** Active  

<!-- FILE_TITLE: Test Pyramid Strategy for Reactive Applications -->  
<!-- FILE_PATH: rules/05-test-strategy/17-test-pyramid.md -->  

---

## 🎯 Purpose  
This document outlines the mandatory test distribution strategy for reactive applications, emphasizing a balanced pyramid of unit (70%), integration (20%), and end-to-end (E2E) (10%) tests. The goal is to deliver fast feedback loops, ensure reactive architecture compliance (e.g., non-blocking WebFlux patterns), and maintain high code quality through semantic tagging and automated enforcement. This approach minimizes flakiness in E2E tests while maximizing coverage of core business logic.

---

## 📌 Scope  
- **Applies to:** Java/Spring Boot/WebFlux backend services, TypeScript/Vue frontend components, and MongoDB data interactions.  
- **Excludes:** Manual exploratory testing, load/performance testing, and non-reactive legacy modules.  
- **Dependencies:**  
  - Aligns with `rules/02-semantic-standards/06-naming-conventions.md` for test naming.  
  - Integrates with `rules/03-coding-practices/10-java-reactive.md` for WebFlux testing.  
  - References `rules/04-data-management/20-test-data-profiles.md` for fixture handling.  

---

## 🔧 Technical Specifications  

### Core Pyramid Ratios (±2% Tolerance)  
| Layer          | Target % | Primary Objective                                       | Recommended Tools                               | Semantic Tag                       |  
|----------------|----------|---------------------------------------------------------|-------------------------------------------------|------------------------------------|  
| **Unit**       | 70%      | Validate isolated business logic (no I/O)               | JUnit 5, Mockito, Reactor Test                  | `@Semantic("UnitBoundary")`        |  
| **Integration**| 20%      | Verify component interactions (e.g., WebFlux + MongoDB) | Testcontainers, WebTestClient, Spring Boot Test | `@Semantic("IntegrationBoundary")` |  
| **E2E**        | 10%      | Confirm full user journeys with real flows              | Cypress, Vue Testing Library, Playwright        | `@Semantic("E2EBoundary")`         |  

### Test Data Management  
- **Structure:** Use versioned YAML fixtures named `<feature>_<scenario>.yml` (e.g., `payment_process_declined.yml`).  
- **Storage:** Place under `src/test/resources/data/profiles/`.  
- **Activation Example (Spring Boot):**  
  ```java  
  @SpringBootTest  
  @ActiveProfiles("payment_process_declined")  
  @Semantic("IntegrationBoundary")  
  class PaymentIntegrationTests {  
      // Test logic here  
  }  
  ```  
- **Isolation:** Testcontainers auto-provisions ephemeral MongoDB instances; purge data post-test to avoid state leakage.  

### Reactive Testing Patterns  
Prioritize non-blocking assertions with Reactor's `StepVerifier`.  

```java  
// Unit test example: Pure reactive logic  
@ExtendWith(MockitoExtension.class)  
@Semantic("UnitBoundary")  
class PaymentValidatorTest {  
  
    @Mock  
    private PaymentRepository repository;  
  
    @Test  
    void processPayment_InvalidCurrency_ThrowsException() {  
        when(repository.findById(any())).thenReturn(Mono.just(mockPayment()));  
        Mono<PaymentResult> result = validator.processPayment(invalidPayment);  
        StepVerifier.create(result)  
            .expectErrorMatches(throwable -> throwable instanceof InvalidCurrencyException)  
            .verify();  
    }  
}  
```  

```typescript  
// E2E test example: Vue component flow  
import { mount } from '@vue/test-utils';  
import { it, describe } from 'vitest';  
  
describe('Checkout Flow', () => {  
  it('@Semantic("E2EBoundary") completes payment with valid card', async () => {  
    const wrapper = mount(CheckoutComponent);  
    await wrapper.find('[data-test="card-input"]').setValue('valid-card');  
    await wrapper.find('[data-test="submit"]').trigger('click');  
    expect(wrapper.emitted('payment-success')).toBeTruthy();  
  });  
});  
```  

---

## 🧠 Semantic Anchors  
- **Layer Identification:** All test classes must include one of:  
  - `@Semantic("UnitBoundary")`: No external dependencies; focuses on algorithms and pure functions.  
  - `@Semantic("IntegrationBoundary")`: Tests wiring between modules (e.g., controller → service → repository).  
  - `@Semantic("E2EBoundary")`: Simulates real user interactions across frontend/backend.  
- **Naming Convention:** Follow `<action>_<condition>_<expectedOutcome>` (e.g., `submitPayment_ExpiredCard_RejectsRequest`). This aligns with behavioral-driven development (BDD) principles for readability.  
- **Additional Tags:** Use `@Semantic("ReactiveCompliant")` for tests verifying non-blocking patterns (e.g., Flux/Mono usage).  

---

## ⚙️ Enforcement  
| Enforcement Aspect        | Tool/Mechanism                  | Action on Violation                  | Configuration Snippet                  |  
|---------------------------|---------------------------------|--------------------------------------|----------------------------------------|  
| **Ratio Compliance**      | Custom Gradle task              | Fail build if outside ±2% tolerance  | See Gradle example below               |  
| **Semantic Tag Presence** | Checkstyle (Java), ESLint (TS)  | Block PR merge                       | Integrate with SonarQube ruleset       |  
| **Reactive Compliance**   | SonarQube + custom plugin       | Warn on blocking calls in tests      | `sonar.sources=src/main` in properties |  
| **Data Isolation**        | Testcontainers lifecycle hooks  | Auto-cleanup; fail on data pollution | Docker Compose for MongoDB stubs       |  

**Gradle Enforcement Task:**  
```kotlin  
// build.gradle.kts  
tasks.register("verifyTestPyramid") {  
    dependsOn("test", "integrationTest", "e2eTest")  
    doLast {  
        val unitCount = test.testResults.files.count { it.name.contains("Unit") }  
        val total = unitCount + integrationTest.testResults.files.size + e2eTest.testResults.files.size  
        check((unitCount.toDouble() / total) in 0.68..0.72) { "Unit tests must be 70% ±2% of total" }  
        exec { commandLine("node", "scripts/validate-ratios.js", "--tolerance=2") }  
    }  
}  
```  
Integrate into CI: Run `verifyTestPyramid` before deployment.

---

## 📋 Compliance Checklist  
- [ ] Total test distribution meets 70/20/10 ratios (±2%).  
- [ ] All test classes include required `@Semantic` annotations.  
- [ ] Unit tests cover 100% of business methods with edge cases.  
- [ ] Integration tests use live Testcontainers for MongoDB (no mocks for DB).  
- [ ] E2E tests include at least one full user flow (e.g., login → checkout → confirm).  
- [ ] Test data profiles are versioned, prefixed by feature, and activated via `@ActiveProfiles`.  
- [ ] No blocking operations (e.g., `.block()`) in reactive tests.  
- [ ] Run `./gradlew verifyTestPyramid` passes locally.  

---

## 🔗 Interlocks  
- **Naming Standards:** Inherits patterns from `rules/02-semantic-standards/06-naming-conventions.md`.  
- **Build Integration:** Requires multimodule Gradle setup from `rules/06-build-tools/22-gradle-multimodule.md`.  
- **Data Handling:** Uses profiled fixtures as defined in `rules/04-data-management/20-test-data-profiles.md`.  
- **Quality Gates:** Feeds into SonarQube dashboards for automated reporting.  

---

## 📜 Revision History  
| Version | Date       | Author              | Changes Summary                          |  
|---------|------------|---------------------|------------------------------------------|  
| 1.0     | 2026-01-24 | LLM-Generated       | Initial draft with ratios and semantics  |  

---

## 🏁 Implementation Guide  
1. **Project Setup:**  
   - Create subfolders: `src/test/java/{unit,integration,e2e}` and `src/test/resources/data/profiles/`.  
   - Add `@Semantic` annotations to existing tests during refactoring.  

2. **Tooling Installation:**  
   - Java: Include `reactor-test` and `testcontainers-mongodb` in `build.gradle.kts`.  
   - Frontend: Install `cypress` and `@vue/test-utils` via npm.  
   - Enforcement: Set up the `verifyTestPyramid` Gradle task in the root build file.  

3. **Migration Steps:**  
   - Audit current tests with `gradle test --info` to baseline ratios.  
   - Refactor high-E2E suites into integration/unit layers.  
   - Validate: Run `./gradlew clean verifyTestPyramid` on feature branches.  

4. **Monitoring:** Generate reports via SonarQube; aim for <5% flakiness in E2E runs.  

**Differentiators:**  
- **AI-Ready:** Semantic tags allow for automated test categorization and gap analysis.  
- **Reactive-Optimized:** Built-in checks prevent common pitfalls like blocking in WebFlux tests.  
- **Scalable:** Ratios adapt to project growth, with CI blocking non-compliant merges.  

**Note:** Replace placeholders (e.g., dates, owners) with project-specific values before adoption. This strategy is enforced in all CI pipelines to uphold Kilo Code's quality standards.