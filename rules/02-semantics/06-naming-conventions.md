# 06 – Naming Conventions – Semantic Intent & Cross-Stack Vibe Alignment

**File Path:** rules/02-semantics/06-naming-conventions.md  
**Domain:** TODO: <business-context> (e.g., "Secure e-commerce payment processing with real-time event streaming")  
**Last Updated:** 2026-01-22  
**Status:** Active  

<!-- FILE_TITLE: Semantic Naming Standards for Vibe Coding -->  
<!-- FILE_PATH: rules/02-semantics/06-naming-conventions.md -->  

---

## 🎯 Purpose
Define deterministic, intent-driven naming conventions that harmonize semantic meaning across Java/WebFlux backend, Vue/TypeScript frontend, and supporting tools. These rules embed vibe coding principles—ensuring names evoke emotional and functional clarity (e.g., `@Vibe("Secure and responsive")`)—while facilitating automated validation, reducing ambiguity in reactive flows, and enabling seamless cross-stack communication. The goal is self-documenting code that accelerates development and maintenance.

## 📌 Scope
- **Applies to:** Java classes (services, DTOs, controllers), Vue components and composables, TypeScript interfaces/props, event emissions, test files, and Gradle/Maven build scripts.
- **Excludes:** Third-party library internals, auto-generated code (e.g., JPA entities from tools), and raw configuration (e.g., YAML properties).
- **Dependencies:** Builds on `05-vibe-coding.md` (core principles), `12-annotations.md` (annotation usage), `09-java-style.md` (backend specifics), and `10-typescript-style.md` (frontend specifics).

---

## 🧠 Semantic Naming Framework

### Core Principles
- **Intent Encoding:** Names must reveal action, domain, execution model (e.g., reactive vs. synchronous), and vibe (e.g., security, performance).
- **Cross-Stack Mirroring:** Backend DTOs directly inspire frontend props; event names align between Java and Vue emits.
- **Vibe Integration:** Pair names with `@Semantic` (functional intent) and `@Vibe` (emotional/qualitative tone) annotations.
- **Pattern Structure:** `[Verb][Domain][Qualifier]?[ExecutionSuffix]`

### Type Identification Suffixes
| Context              | Pattern              | Java Example                  | TS/Vue Example                 | Enforcement Tool          |
|----------------------|----------------------|-------------------------------|--------------------------------|---------------------------|
| Data Transfer Object | `*Dto`               | `OrderPaymentDto`             | `OrderPaymentProps` (interface)| Checkstyle/ESLint         |
| Domain Entity/Model  | `*Entity`            | `UserEntity`                  | N/A (use DTO mirrors)          | Hibernate/JPA validators  |
| Reactive Single      | `*Mono` or `*Async`  | `fetchUserMono()`             | `loadUserAsync()` (composable) | Custom Gradle task        |
| Reactive Stream      | `*Flux` or `*Stream` | `streamOrdersFlux()`          | `orderStream` (prop)           | WebFlux type checks       |
| Service Layer        | `*Service`           | `PaymentProcessingService`    | `usePaymentService()`          | Naming plugin             |
| Vue Component        | PascalCase           | N/A                           | `OrderTracker.vue`             | ESLint vue/component-name |
| Event/Domain Event   | `*Event` (Pascal)    | `OrderCompletedEvent`         | `order-completed` (kebab emit) | PropType validators       |

**Vibe Example:**  
```java
// Backend: Encodes security vibe
@Semantic("SecureTransactionFlow")
@Vibe("Atomic and resilient")
public Mono<PaymentResultDto> processSecurePaymentAsync(PaymentIntentDto intent) { ... }
```
```typescript
// Frontend: Mirrors with vibe comment
interface Props {
  paymentStream: PaymentEventDto[];  // @Vibe("Real-time, secure updates")
}
const emit = defineEmits<{ (e: 'payment-processed', payload: PaymentEventDto): void }>();
```

### Cross-Stack Alignment Rules
1. **DTO ↔ Prop Synchronization:** Backend `XyzDto` becomes frontend `XyzProps`. Fields must match exactly, with TypeScript type guards for runtime validation.
2. **Reactive Signatures:** All `Mono`/`Flux` methods append `*Async` or `*Stream`. Vue composables mirror with `use[Domain][Action]Async()`.
3. **Event Naming Parity:** Backend events use PascalCase (e.g., `handleOrderUpdatedEvent`); Vue emits use kebab-case (e.g., `order-updated`). Endpoints match emit names where possible.
4. **Timestamp/Qualifier Patterns:** Use `*On` for timestamps (e.g., `createdOn`, `processedOn`) in ISO-8601 format; avoid generics like `date` or `time`.

**Anti-Patterns & Corrections:**
```java
// ❌ Vague and blocking
public List<Order> getOrders(String userId) { ... }

// ✅ Semantic, reactive, vibe-aligned
@Semantic("LiveOrderFeed")
@Vibe("Responsive and filtered")
public Flux<OrderDto> streamUserOrdersAsync(String userId) { ... }
```
```vue
<!-- ❌ Untyped, ambiguous prop -->
<OrderTracker :data="orders" />

<!-- ✅ Typed, mirrored prop -->
<template>
  <OrderTracker :order-stream="userOrders" @order-updated="handleUpdate" />
</template>

<script setup lang="ts">
interface OrderStreamProps {
  orderStream: OrderDto[];  // Mirrors backend Flux<OrderDto>
}
defineProps<OrderStreamProps>();
</script>
```

---

## ⚙️ Enforcement Mechanics

### Annotation-Driven Checks
- **Java:** Use `@Semantic` for intent and `@Vibe` for tone on all public methods/DTOs. Integrate with SpotBugs or a custom Lombok processor.
- **TypeScript/Vue:** Embed vibe comments (e.g., `// @Vibe("...")`) above props/emits; enforce with JSDoc and ESLint.

### Build-Time Validation
| Rule Category       | Tool/Integration                          | Action on Violation             |
|---------------------|-------------------------------------------|---------------------------------|
| Suffix Compliance   | Checkstyle (Java) + ESLint (TS)           | Fail build/PR                   |
| Cross-Stack Parity  | Custom Gradle task + npm script           | Generate report, block deploy   |
| Annotation Presence | Gradle plugin (e.g., AnnotationProcessor) | Warn in IDE, error in CI        |
| Reactive Consistency| WebFlux/WebTestClient assertions          | Integration test failures       |
| Vue Prop Typing     | Volar/Vetur + TypeScript strict           | Compilation errors              |

**Gradle Cross-Validation Task (build.gradle.kts):**
```kotlin
tasks.register("validateNamingConventions") {
    dependsOn(":backend:checkstyleMain", ":frontend:lint")
    doLast {
        exec {
            commandLine("node", "scripts/verify-cross-stack-names.js")  // Compares DTOs vs. props
        }
        // Additional: Run Mermaid validation for semantic diagrams if referenced
    }
}
```

**npm Script Example (package.json):**
```json
{
  "scripts": {
    "lint:naming": "eslint --rule 'vue/require-prop-types: error' src/**/*.vue && ts-node scripts/naming-audit.ts"
  }
}
```

**Pre-Commit Hook (Husky):**
- Run `./gradlew validateNamingConventions` and `npm run lint:naming` before git commit.

### CI/CD Integration
- **GitHub Actions/Jenkins:** Add `validateNamingConventions` as a mandatory step in PR workflows.
- **Audit Script:** A Node.js tool (`scripts/naming-validator.js`) that parses Java/TS files, checks suffixes, and flags mismatches (e.g., missing `*Async` on `Mono` returns).

---

## 📋 Compliance Checklist
- [ ] All DTOs/Entities use mandatory suffixes (`*Dto`, `*Entity`) and mirror across stacks.
- [ ] Reactive methods include `*Async`/`*Stream` and return `Mono`/`Flux` where appropriate.
- [ ] Vue props are fully typed interfaces mirroring backend DTOs (no `any` or loose typing).
- [ ] Event emits and backend handlers use aligned names (kebab vs. PascalCase).
- [ ] `@Semantic` and `@Vibe` annotations/comments present on public APIs and props.
- [ ] Timestamps follow `*On` pattern; no generic names (e.g., avoid `data`, `process` without domain).
- [ ] Build validation passes: `./gradlew validateNamingConventions` && `npm run lint:naming`.
- [ ] Tests follow conventions (e.g., `PaymentServiceSpec.java`, `OrderFlowE2E.spec.ts`).

---

## 🔗 Interlocks with Other Rules
- **Vibe Coding (`05-vibe-coding.md`):** Names must align with vibe annotations to "feel right" emotionally and functionally.
- **Annotations (`12-annotations.md`):** Required for semantic markers; extend with custom `@NamingCompliant`.
- **Java/TypeScript Style (`09-java-style.md`, `10-typescript-style.md`):** Layer-specific extensions (e.g., camelCase for TS vars).
- **Workflows (`13-feature-lifecycle.md`):** Naming audits during PR reviews; use Mermaid for visualizing cross-stack flows (e.g., `payment-flow.mmd` with semantic labels).
- **Testing (`14-testing-strategy.md`):** Test names encode scenarios (e.g., `shouldProcessPaymentAsyncSuccessfully`).

---

## 📜 Revision History
| Version | Date       | Author          | Changes Summary                                               |
|---------|------------|-----------------|---------------------------------------------------------------|
| 1.0     | 2026-01-22 | LLM-Generated   | Initial vibe-aligned conventions with cross-stack enforcement |

**Implementation Notes:**
- **Customization:** Replace `TODO` placeholders with project specifics (e.g., domain context). Adapt suffixes if using non-WebFlux backends.
- **Adoption Path:** Start with a team spike: Audit existing code, integrate Gradle/ESLint rules, then enforce via PR templates.
- **Vibe Check:** Names should "resonate"—e.g., `processSecurePaymentAsync` evokes trust and speed. Run vibe workshops for team buy-in.
- **Extensions:** For monorepo setups, add a shared `naming-schema.json` for dynamic validation across modules.
