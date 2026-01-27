# 27 – Refactoring Guidelines for Semantic Modular Architecture

**File Path:** rules/07-modularity/27-refactoring-guidelines.md  
**Domain:** TODO: <fill with project domain, e.g., "Reactive e-commerce platform">  
**Last Updated:** 2026-01-25
**Status:** Draft  

<!-- FILE_TITLE: Refactoring Guidelines for Semantic Modular Architecture -->  
<!-- FILE_PATH: rules/07-modularity/27-refactoring-guidelines.md -->  

---

## 🎯 Purpose  
Establish principled, safe refactoring practices that evolve modular boundaries while preserving semantic intent via `@Semantic` and `@Vibe` annotations, maintaining reactive data flows in WebFlux, and ensuring cross-stack consistency between Java backend and Vue/TypeScript frontend. This prevents regressions in modularity, reactivity, and alignment during module evolutions.

## 📌 Scope  
- **Applies to:** Module splits/merges, service extractions, component restructurings, and semantic renamings across Java/WebFlux services and Vue components.  
- **Excludes:** Pure third-party library updates without semantic impact or non-code changes (e.g., documentation-only).  
- **Dependencies:**  
  - [04-modular-architecture.md](04-modular-architecture.md) (core boundaries)  
  - [06-naming-conventions.md](06-naming-conventions.md) (naming standards)  
  - [26-dependency-rules.md](26-dependency-rules.md) (dependency management)  

---

## 🧠 Semantic Refactoring Principles  
Refactoring must anchor on semantic clarity to avoid drift. Key principles:  

1. **Intent Preservation & Evolution**  
   - Retain or evolve `@Semantic` and `@Vibe` annotations to reflect refined intent without breaking contracts.  
   - Example:  
     ```java  
     // Before: Broad service  
     @Semantic("OrderProcessor")  
     @Vibe("Handles full order lifecycle")  
     public class OrderService { ... }  
     
     // After: Semantic split  
     @Semantic("PaymentGateway")  
     @Vibe("Manages payment events and streams")  
     public class PaymentService { ... }  
     ```  

2. **Reactive Continuity**  
   - Preserve non-blocking patterns: All async operations must retain `Flux`/`Mono` return types and suffixes (e.g., `streamOrdersAsync`).  
   - Avoid introducing blocking calls (e.g., no `.block()` in refactors).  
     ```java  
     // Before  
     public Flux<Order> getOrders(String userId) { ... }  
     
     // After: Refactored with preserved reactivity  
     @Semantic("OrderStream")  
     public Flux<Order> streamOrdersAsync(String userId) { ... }  
     ```  

3. **Cross-Stack Alignment**  
   - Ensure backend DTOs and reactive streams mirror frontend types/props. Update Vue interfaces to match evolved Java structures.  
     ```typescript  
     // Vue: Mirrors Java Flux<OrderDto>  
     interface Props {  
       orderStream: OrderDto[];  // Typed array for stream handling  
     }  
     ```  
   - Use shared DTO definitions (e.g., via shared types or codegen) to enforce sync.  

4. **Boundary Integrity**  
   - Refactors must respect module boundaries; visualize changes first with Mermaid diagrams to identify splits/merges.  

---

## 🔧 Technical Guidelines & Patterns  

### Java/WebFlux Refactoring  
Focus on modular decomposition while keeping reactive pipelines intact.  

#### Module Splitting (Decomposition)  
**Trigger:** Module >500 LOC, mixed concerns, or high coupling detected via `gradle dependencyInsight`.  

**Protocol:**  
1. **Diagram First:** Update architecture visuals.  
   ```mermaid  
   graph TD  
     A[Monolithic OrderService] -->|split| B[PaymentGateway Module]  
     A -->|split| C[ShippingModule]  
     style B fill:#f9f,stroke:#333  
     style C fill:#9ff,stroke:#333  
   ```  

2. **Gradle Submodule Setup:**  
   ```kotlin  
   // settings.gradle.kts  
   include(":modules:payment", ":modules:shipping")  
   ```  

3. **Code Migration with Semantics:**  
   - Extract domain logic with boundary annotations.  
     ```java  
     @Semantic("InventoryBoundary")  
     public class InventoryService {  
         public Mono<StockUpdate> processStockFlux(Flux<InventoryEvent> events) {  
             return events.flatMap(this::updateStockAsync);  
         }  
     }  
     ```  
   - Preserve reactive signatures; add suffixes for clarity (e.g., `processAsync`).  

#### Service Extraction  
**Trigger:** Controller/service >300 LOC with mixed responsibilities.  

- Move inline logic to dedicated classes.  
  ```java  
  // Before: In controller  
  public Flux<Order> handleOrders() { ... }  
  
  // After: Extracted service  
  @Semantic("OrderProcessingService")  
  public Flux<Order> processOrdersAsync(Flux<OrderRequest> requests) { ... }  
  ```  
- Inject extracted services via Spring `@Autowired` or constructor.  

### Vue/TypeScript Refactoring  
Align frontend modularity with backend changes.  

#### Component Splitting  
**Trigger:** Component >200 LOC or handles multiple data streams.  

```vue  
<!-- Before: Monolithic component -->  
<template>  
  <div>Order and Payment UI mixed</div>  
</template>  
<script>  
export default { props: ['data'] }; // Untyped  
</script>  

<!-- After: Split with typed props -->  
<template>  
  <OrderStream :events="orderEvents" />  
  <PaymentAlerts :stream="paymentStream" />  
</template>  
<script setup lang="ts">  
interface Props {  
  orderEvents: OrderEvent[];  
  paymentStream: PaymentDto[];  // Matches backend Flux<PaymentDto>  
}  
defineProps<Props>();  
</script>  
```  

- Extract composables for shared reactive logic (e.g., `usePaymentStream`).  
- Use `computed` or `watch` for Vue reactivity mirroring WebFlux streams.  

#### Cross-Stack Sync Protocol  
1. Update Java DTO → Generate/regenerate frontend types.  
2. Type-check with `tsc --strict` to catch mismatches.  
3. Test end-to-end: Backend stream → Vue prop rendering.  

### Module Merging (Composition)  
**Trigger:** Redundant modules with >80% shared logic.  

1. Audit with `gradle checkCircularDeps`.  
2. Merge code, consolidate `@Semantic` tags.  
3. Update Vue by combining components, preserving props.  

---

## ⚙️ Enforcement Matrix  
Integrate checks into CI/CD for automated validation.  

| Check                        | Tool/Method                                          | Action if Failed             |  
|------------------------------|------------------------------------------------------|------------------------------|  
| `@Semantic`/`@Vibe` Coverage | Custom Gradle Task (`verifyVibe`)                    | Fail build / Block PR        |  
| Reactive Naming & Patterns   | ESLint Plugin + SonarQube                            | Warn / Auto-fix suffixes     |  
| Cross-Stack Type Consistency | TypeScript Compiler (`tsc --strict`) + DTO diff tool | Compilation error / PR block |  
| Circular Dependencies        | Gradle Dependency Plugin                             | Fail build                   |  
| Diagram Updates              | Manual review + Git diff                             | Require PR update            |  

**Custom Gradle Task Example:**  
```kotlin  
// build.gradle.kts  
tasks.register("verifyVibe") {  
    doLast {  
        // Scan for missing @Semantic tags  
        // e.g., via AnnotationProcessor or regex check  
    }  
}  
```  

---

## 📋 Compliance Checklist  
For every refactoring PR:  
- [ ] `@Semantic` and `@Vibe` annotations preserved/evolved  
- [ ] Reactive signatures intact (no blocking introduced)  
- [ ] Cross-stack sync: Vue props match Java DTOs/Flux types  
- [ ] No circular dependencies (validated via Gradle)  
- [ ] Mermaid diagrams updated in `/docs/architecture/`  
- [ ] Tests updated (≥80% coverage on refactored code)  
- [ ] Incremental verification: Run `gradle refactorCheck` per step  

---

## 📜 Revision History  
| Version | Date       | Author        | Changes Summary                              |
|---------|------------|---------------|----------------------------------------------|
| 1.0     | 2026-01-25 | LLM-Generated | Initial draft with core structure.           |

---

## Implementation Notes  
- **Start with Visualization:** Always update Mermaid diagrams before code changes to guide boundaries.  
- **Incremental Approach:** Refactor in small commits (e.g., one module at a time) with post-step tests.  
- **Domain Customization:** Fill TODOs with project-specific examples (e.g., e-commerce order flows).  
- **Toolchain Tips:** Integrate with existing CI (e.g., GitHub Actions) via the enforcement matrix. Run `gradle clean build` after every semantic change.  
- **Best Practice:** Pair refactors with code reviews focusing on semantic drift.  

**TODO Items:**  
- Add domain-specific examples (e.g., inventory/payment flows).  
- Customize Gradle/ESLint rules for your repo.  
- Assign owner and set status to "Active" post-review.  
