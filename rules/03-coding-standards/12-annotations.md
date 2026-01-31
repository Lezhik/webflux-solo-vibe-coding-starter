# 12 – Semantic Annotations – Cross-Stack Intent and Flow Markers

**File Path:** rules/03-coding-standards/12-annotations.md  
**Domain:** TODO: <business-context> (e.g., "Secure e-commerce payment processing with real-time event streaming")  
**Last Updated:** 2026-01-23  
**Status:** Active  

<!-- FILE_TITLE: Semantic Annotations for Reactive and Vibe-Driven Codebases -->  
<!-- FILE_PATH: rules/03-coding-standards/12-annotations.md -->

---

## 🎯 Purpose
This guide establishes a unified annotation framework to embed semantic roles, developer intent, and data flow tracking across Java/Spring Boot/WebFlux (backend) and Vue/TypeScript (frontend) codebases. Annotations promote:
- **Semantic Consistency**: Clear classification of code elements (e.g., `@Semantic` for roles).
- **Intent Documentation**: Human-readable explanations via `@Vibe` to aid maintenance and LLM analysis.
- **Reactive Flow Visibility**: Explicit markers like `@dataflow` for non-blocking streams, enabling automated diagramming and validation.
- **Cross-Stack Alignment**: Shared vocabulary to synchronize backend and frontend behaviors.

This approach differentiates from generic Javadoc by enforcing build-time validation and integrating with vibe-coding principles for readable, intent-driven code.

## 📌 Scope
- **Applies to:** Java classes/services (including WebFlux Mono/Flux), TypeScript modules, Vue components, and shared DTOs.
- **Excludes:** Third-party libraries, configuration files, and auto-generated code.
- **Dependencies:** 
  - [05-vibe-coding.md](../02-semantics/05-vibe-coding.md) (core vibe principles)
  - [06-naming-conventions.md](../02-semantics/06-naming-conventions.md) (naming alignment)
  - [09-java-style.md](../03-coding-standards/09-java-style.md) (WebFlux specifics)
  - [10-typescript-style.md](../03-coding-standards/10-typescript-style.md) (Vue composition rules)

---

## 🧠 Semantic Anchors

### 1. Core Annotation Taxonomy
| Annotation                       | Stack       | Purpose                                             | Example Usage                                                        |
|----------------------------------|-------------|-----------------------------------------------------|----------------------------------------------------------------------|
| `@Semantic("")`                  | Java/TS     | Declares the code's architectural/conceptual role   | `@Semantic("AggregateRoot")`                                         |
| `@Vibe("")`                      | Java/TS     | Captures contextual intent and behavioral rationale | `@Vibe("Ensures atomic settlement with retries on race conditions")` |
| `@dataflow("")`                  | Java/TS     | Tracks reactive data streams across layers          | `@dataflow("backend-to-frontend")`                                   |
| `@ReactiveStream("")` (Optional) | Java/TS     | Marks non-blocking boundaries for clarity           | `@ReactiveStream("PaymentEvents")`                                   |

These annotations are implemented as custom JSDoc (for TS/Vue) or Java annotations (for backend), with runtime retention where needed for tools.

### 2. Placement and Usage Rules
- **Classes/Components/DTOs:** Apply `@Semantic` and `@Vibe` at the top level.
- **Methods (Reactive):** Add `@dataflow` for streams (e.g., Flux/Mono in Java, event props in Vue).
- **Fields/Props:** Use `@Semantic` for serialization-sensitive elements.
- **Cross-Stack Rule:** Frontend annotations must mirror backend equivalents (e.g., same `@dataflow` path) to facilitate automated sync checks.

**Java/WebFlux Example:**
```java
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;

/**
 * @Semantic("PaymentGatewayAdapter")
 * @Vibe("Handles idempotent payment processing with rollback for failures")
 * @dataflow("upstream-banking-to-service")
 */
@RestController
public class PaymentController {
    
    /**
     * @Vibe("Non-blocking transaction with retry logic")
     * @dataflow("service-to-controller")
     */
    public Mono<PaymentResult> processPaymentFlux(@Semantic("PaymentIntent") PaymentRequest request) {
        // Reactive pipeline implementation
        return paymentService.processAsync(request);
    }
}
```

**Vue/TypeScript Example:**
```vue
<script setup lang="ts">
import { defineProps, PropType } from 'vue';

/**
 * @Semantic("LivePaymentRenderer")
 * @Vibe("Visualizes real-time payment lifecycle with event-driven updates")
 * @dataflow("backend-to-frontend")  // Mirrors backend flow
 */
const props = defineProps({
    paymentStream: {
        type: Array as PropType<PaymentEvent[]>,
        required: true
    }
});

// Composition logic for rendering
</script>

<template>
  <div v-for="event in paymentStream" :key="event.id">
    <!-- Render payment status -->
  </div>
</template>
```

### 3. Anti-Patterns
```java
// ❌ Anti-Pattern: Missing semantic context in reactive methods
public Flux<OrderEvent> getOrderStream(OrderId id) { ... }  // Unclear role and flow

// ✅ Corrected:
@Semantic("OrderEventEmitter")
@Vibe("Streams order updates via WebSocket for real-time UI")
@dataflow("db-to-frontend")
public Flux<OrderEvent> getOrderStream(OrderId id) { ... }
```

---

## ⚙️ Enforcement Mechanics

| Rule                                      | Tool/Mechanism                             | Error Message                                | Severity       |
|-------------------------------------------|--------------------------------------------|----------------------------------------------|----------------|
| Missing `@Semantic` on classes/components | Checkstyle (Java) / ESLint (TS)            | "SEMANTIC_ROLE_REQUIRED_FOR_PUBLIC_ELEMENTS" | Build Fail     |
| Empty or vague `@Vibe`                    | Custom Gradle Task / ESLint Rule           | "VIBE_INTENT_MUST_DESCRIBE_BEHAVIOR"         | Warning → Fail |
| Invalid `@dataflow` path (unregistered)   | AST Analysis Script (Node.js + JavaParser) | "DATAFLOW_PATH_NOT_IN_ARCHITECTURE_MAP"      | PR Block       |
| Mismatched cross-stack annotations        | Gradle Multi-Module Task                   | "FRONTEND_BACKEND_ANNOTATION_MISMATCH"       | Build Fail     |

**CI/CD Integration:**
- **Gradle Task (Backend/Full Stack):**
  ```kotlin
  tasks.register("validateAnnotations") {
      dependsOn("checkstyleMain", ":frontend:lint")
      doLast {
          javaexec {
              mainClass.set("com.kilo.AnnotationValidator")
              args = listOf("--strict", "--scan-all", "--cross-stack")
          }
          exec {
              commandLine("node", "scripts/validate-dataflows.js", "--project-root")
          }
      }
  }
  ```
- **Pre-Commit Hook:** Run `./gradlew validateAnnotations` and `npm run lint:annotations`.
- **GitHub Actions Snippet:**
  ```yaml
  # .github/workflows/ci.yml
  jobs:
    validate:
      steps:
        - name: Run Annotation Checks
          run: ./gradlew validateAnnotations
  ```

Annotations are processed via custom processors in `buildSrc/` for Java and a TypeScript plugin for JSDoc parsing.

---

## 🔧 Technical Specifications

### Java Annotation Definitions
Define in a shared module for runtime/access:
```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface Semantic {
    String value();
}

@Retention(RetentionPolicy.RUNTIME)
public @interface Vibe {
    String value();
}

@Retention(RetentionPolicy.RUNTIME)
public @interface dataflow {
    String value();  // Format: "source-to-destination"
}
```

### Vue/TypeScript JSDoc Integration
Use JSDoc for static analysis:
```typescript
/**
 * @annotation Semantic("DataConsistencyGuard")
 * @annotation Vibe("Prevents stale state in reactive UIs")
 * @annotation dataflow("service-to-ui")
 */
export function usePaymentValidator(stream: PaymentEvent[]): ValidationResult { ... }
```
- **IDE Support:** Configure VS Code IntelliSense and IntelliJ templates for auto-insertion.
- **Automated Diagramming:** `@dataflow` values feed into tools like PlantUML for generating architecture diagrams (e.g., via Gradle plugin).

### Key Implementation Notes
1. **Replace Placeholders:** Update TODOs with project details (e.g., domain, owners).
2. **Extension Points:** Add `@ReactiveStream` only for complex WebFlux/Vue event handling; keep core set minimal.
3. **Validation Extensions:** Store valid `@dataflow` paths in a central `architecture-flows.json` for reference.
4. **Interlocks:** Annotations must align with naming conventions (e.g., `@Semantic("EventStream")` pairs with `orderEventStream$` names).

---

## 📋 Compliance Checklist
- [ ] All domain classes, services, and components include `@Semantic` and `@Vibe`.
- [ ] Reactive methods (Mono/Flux, event props) use `@dataflow`.
- [ ] Cross-stack parity: Frontend annotations match backend flows.
- [ ] `./gradlew validateAnnotations` passes without errors.
- [ ] `@Vibe` descriptions are concise yet informative (1-2 sentences).
- [ ] Reviewed in PRs per [16-review-checklist.md](../04-workflow/16-review-checklist.md).

---

## 🔗 Interlocks with Other Rules
- **Vibe Coding:** `@Vibe` enforces readable intent from [05-vibe-coding.md].
- **Naming:** Semantic roles guide conventions in [06-naming-conventions.md] (e.g., suffix `*Stream` for `@dataflow` targets).
- **Build Workflow:** Integrates with [22-gradle-multimodule.md](../05-build/22-gradle-multimodule.md) for validation.
- **Reviews:** Mandatory annotation checks in pull requests.

---

## 📜 Revision History
| Version | Date       | Author          | Changes Summary                              |
|---------|------------|-----------------|----------------------------------------------|
| 1.0     | 2026-01-23 | LLM-Generated   | Initial draft with core taxonomy             |

**Differentiators from Generic Guides:**
1. **Cross-Stack Unified Approach**: Single taxonomy for JVM and TS ecosystems.
2. **Intent Tracing**: `@Vibe` adds human/LLM-readable rationale beyond dry docs.
3. **Flow Automation**: `@dataflow` supports diagram generation and validation.
4. **Strict Enforcement**: CI-blockers ensure compliance, not just linter warnings.