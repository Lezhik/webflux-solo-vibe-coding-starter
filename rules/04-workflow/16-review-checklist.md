# 16 – Code Review Checklist – Vibe-Centric Compliance
<!-- FILE_TITLE: "Vibe-Centric PR Review Checklist" -->
<!-- FILE_PATH: rules/04-workflow/16-review-checklist.md -->

**File Path:** rules/04-workflow/16-review-checklist.md
**Domain:** TODO: <business-context> (e.g., "Secure e-commerce payment processing with real-time event streaming")  
**Last Updated:** 2026-01-23
**Status:** Active  

---

## 🎯 Purpose 
Enforce semantic consistency, architectural compliance, and vibe coding integrity through structured peer reviews before merging. This checklist ensures code changes align with reactive patterns, cross-stack synchronization, and automated validation gates.

## 📌 Scope
- **Applies to:** All `feat/*`, `fix/*`, and `refactor/*` branches targeting `main`.
- **Excludes:** `chore/*` branches (e.g., documentation-only or dependency updates).
- **Dependencies:** 
  - [05-vibe-coding.md](/rules/02-semantics/05-vibe-coding.md) for annotation and naming standards.
  - [13-feature-lifecycle.md](/rules/04-workflow/13-feature-lifecycle.md) for overall workflow context.
  - [06-naming-conventions.md](/rules/02-semantics/06-naming-conventions.md) for branch and method naming.

---

## 🧠 Semantic Review Anchors
1. **Intent Transparency**  
   - Reactive methods must use `*Async`/`*Flux`/`*Mono` suffixes (e.g., `streamPaymentsFluxAsync()`).  
   - `@Semantic` annotations declare domain roles (e.g., `@Semantic("PaymentGateway")`).  
   - `@Vibe` annotations convey execution context (e.g., `@Vibe("Non-blocking inventory reservation with optimistic locking")`).

2. **Cross-Stack Alignment**  
   - Backend DTOs mirror frontend props (e.g., `PaymentDto` in Java ↔ `PaymentProps` in TypeScript/Vue).  
   - Event streams maintain identical terminology (e.g., Java `Flux<OrderEvent>` ↔ Vue `orderEventStream` prop).  
   - No `any` types in TypeScript; all Vue props strictly typed.

3. **Diagram and Documentation**  
   - Include a valid Mermaid diagram for feature flows in `/features/[feature]/diagram.md`.  
   - Static resources must be versioned (e.g., `<script src="/static/v1/payment.js" defer></script>`; no inline JS).

---

## ⚙️ Enforcement Mechanics
| Check Type                | Validation Method                                        | Failure Action              |
|---------------------------|----------------------------------------------------------|-----------------------------|
| Branch Naming             | Git hook with regex `^(feat\|fix\|refactor)/[a-z0-9-]+$` | Reject push                 |
| Semantic Annotations      | Gradle task `./gradlew verifyVibe`                       | Block PR / Fail build       |
| Reactive Naming           | ESLint rule `reactive-naming` (TS) / Custom lint (Java)  | Auto-fix suggestion / Block |
| DTO/Props Synchronization | ESLint `reactive-dto-mirror` rule                        | Block PR                    |
| Test Coverage             | JaCoCo (Java ≥80%) / Istanbul/Vitest (TS ≥80%)           | Warn / Block if <80%        |
| Mermaid Diagram Validity  | Mermaid CLI: `mmdc -i diagram.md`                        | Reject commit / PR          |
| Circular Dependencies     | `./gradlew dependencyCheck`                              | Fail build                  |

**CI/CD Integration:**
```bash
# Pre-merge pipeline (e.g., GitHub Actions)
./gradlew clean verifyVibe test
npm run lint -- --rule reactive-naming=error reactive-dto-mirror=error
mermaid validate ./features/**/*.md
npm run test:coverage  # Ensure ≥80%
./gradlew dependencyCheck
```

---

## 🔧 Technical Specifications

### Mandatory Review Points
1. **Branch Naming**  
   - Pattern: `feat/<verb>-<domain>` or `fix/<issue>-<context>` (e.g., `feat/stream-payment-events`, `fix/race-condition-checkout`).  
   - Invalid examples: `update-stuff`, `bugfix-123` (must be descriptive and semantic).

2. **Semantic Markers**  
   ```java
   // Backend (Java/Spring) example
   @Semantic("OrderAggregate")
   @Vibe("Handles concurrent updates via optimistic locking and event streaming")
   public Flux<OrderEvent> updateOrderFluxAsync(OrderUpdateDto dto) { ... }
   ```
   ```typescript
   // Frontend (Vue/TypeScript) example
   /**
    * @Semantic("DataVisualizer")
    * @Vibe("Renders live payment events with rollback indicators")
    */
   interface PaymentProps {
     eventStream: PaymentEvent[];  // Mirrors backend Flux<OrderEvent>
   }
   ```

3. **Test Coverage and Scenarios**  
   - ≥80% line coverage for new/changed code (unit + integration).  
   - Include boundary tests (e.g., empty streams, race conditions) using JUnit/Mockito for Java, Vitest for TS.  
   - E2E tests for critical flows (e.g., Cypress for payment processing).

4. **Additional Checks**  
   - No circular dependencies across modules.  
   - Changelog entry in `CHANGELOG.md` following semantic versioning (e.g., `## 1.3.0 - YYYY-MM-DD\n- Added real-time streaming (feat/stream-payment-events)`).  
   - Vue components use scoped CSS (`<style scoped>`).  
   - `@dataflow` tags for cross-module streams if applicable.

### PR Readiness Requirements
- [ ] Branch name follows pattern.  
- [ ] All methods have `@Semantic`/`@Vibe` annotations (or JSDoc equivalents).  
- [ ] DTOs/props are synchronized across stacks.  
- [ ] Mermaid diagram exists and validates.  
- [ ] Coverage reports attached (≥80%, with boundary scenarios).  
- [ ] Static resources versioned and no inline scripts.  
- [ ] CI pipeline passes all checks.  
- [ ] Changelog updated.  

**Non-Compliance Triggers:**  
- Missing annotations or naming issues → Immediate rejection/block.  
- Coverage <80% → Require additional tests before merge.  
- Invalid diagrams or deps → Fail build.

---

## 📋 Compliance Verification
1. **Automated Steps:** Run CI tasks; must pass before human review.  
2. **Manual Review Steps:**  
   - Confirm semantic tags and intent clarity.  
   - Validate cross-stack alignment and diagram accuracy.  
   - Check for edge cases in tests.  
   - Ensure 2 approvers (e.g., one backend, one frontend).  
3. **Reviewer Confirmation:** Comment "✅ Checklist passed" on PR.  
4. **Merge Rules:** Only after CI green + manual approvals. Emergency overrides require linked incident ticket.

---

## 🔗 Interlocks
- **Naming & Semantics:** Cross-validate with [06-naming-conventions.md](/rules/02-semantics/06-naming-conventions.md).  
- **Testing:** Follow [17-test-pyramid.md](/rules/05-test-strategy/17-test-pyramid.md) and [20-test-data-profiles.md](/rules/05-test-strategy/20-test-data-profiles.md).  
- **Build & Resources:** Align with [22-gradle-multimodule.md](/rules/06-build/22-gradle-multimodule.md) and [23-static-resources.md](/rules/06-build/23-static-resources.md).  
- **Discoverability:** Post-merge audit via [29-semantic-discoverability.md](/rules/02-semantics/29-semantic-discoverability.md).

---

## 📜 Revision History
| Version | Date       | Author          | Changes Summary                                                                                            |
|---------|------------|-----------------|------------------------------------------------------------------------------------------------------------|
| 1.0     | 2026-01-23 | LLM-Generated   | Initial version combining semantic anchors, automation, and cross-stack rules for vibe-coding enforcement. |

---

## Implementation Notes
- **Reviewer Assignment:** Require 2 domain-specific approvers (e.g., payments team for related features).  
- **Tool Setup:** Configure Gradle/ESLint/Mermaid CLI in repo; add pre-commit hooks for early validation.  
- **Project Customization:** Replace `TODO:` placeholders with specifics (e.g., domain, owners). Focus on actionable, non-metric checks to maintain semantic integrity without speculative elements.  
- **Post-Merge:** Optional LLM analysis for vibe coherence to ensure long-term maintainability.