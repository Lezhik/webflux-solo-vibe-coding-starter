# 28 – Semantic Discoverability

**File Path:** rules/07-modularity/29-semantic-discoverability.md  
**Domain:** TODO: <Provide business domain, e.g., "E-commerce order management">  
**Last Updated:** 2026-01-25
**Status:** Active  

---

## 🎯 Purpose  
Enable developers, AI tools (e.g., LLMs), and automated systems to quickly infer system behavior, domain roles, data flows, and execution models through consistent semantic naming conventions, annotations, and structural patterns. This reduces cognitive load, accelerates onboarding, refactoring, and cross-stack integration, while promoting "vibe coding" principles for intuitive, reactive clarity in under 5 seconds per method or component.

---

## 📌 Scope  
- **Applies to:** All Java/WebFlux services, Vue components, TypeScript utilities, and cross-stack DTOs in modular components.  
- **Excludes:** Third-party libraries, generated code, CI/CD configuration files, and internal tooling metadata.  
- **Dependencies:** Aligns with [06-naming-conventions.md](../02-semantics/06-naming-conventions.md) and [27-dependency-rules.md](../07-modularity/27-dependency-rules.md).  

---

## 🧠 Semantic Anchors  

### 1. Verb-Domain-Context Pattern (Core Naming Mechanism)  
**Why:** Encodes intent, domain focus, and execution context (e.g., reactive vs. blocking) directly in names, allowing inference without reading implementations.  

**Pattern:** `[Verb][DomainEntity][Context]`  
- **Verbs:** Action-oriented (e.g., `stream*`, `fetch*`, `validate*`, `publish*`, `transform*`, `observe*`).  
- **Domains:** Business entities (e.g., `Order*`, `Payment*`, `User*`, `Inventory*`).  
- **Context:** Execution hints (e.g., `Async`, `Stream`, `Dto`, `Event`, `Repo`).  

**Enforcement Rules:**  
- Reactive methods (e.g., `Flux`/`Mono` returns) must include `Async` or `Stream` suffixes.  
- Avoid generics like `getData`, `handleInput`, or `process`.  

**Examples:**  
```java  
// ✅ Good: Reactive intent and domain clear in <3 seconds  
@Semantic("OrderStream")  
public Flux<OrderEventDto> streamPendingOrderEventsAsync(OrderId orderId) { ... }  

// ✅ Good: Blocking alternative, but explicit  
public Mono<OrderConfirmation> fetchOrderConfirmationSync(OrderId orderId) { ... }  

// ❌ Bad: Ambiguous purpose and blocking implication  
public List<Object> getData(String id) { ... }  
```  

### 2. Domain Role Indicators (Boundary and Role Signaling)  
**Why:** Explicitly marks classes/components as Aggregate Roots, Adapters, Services, etc., to define module boundaries and responsibilities.  

**How:** Use mandatory suffixes/prefixes and annotations.  
- Suffixes: `*Adapter` (integrations), `*AggregateRoot` (domain entities), `*Service` (business logic), `*Repository` (data access), `*Component` (UI elements).  
- Annotations: `@Semantic("RoleDescription")` for runtime/discoverability hints; `@ModuleBoundary` for entry/exit points; `@Vibe("Brief intent summary")` for vibe coding flavor.  

**Examples:**  
```java  
@ModuleBoundary  
@Semantic("AggregateRoot")  
@Vibe("Core order lifecycle management")  
public class OrderAggregateRoot { ... }  

@Semantic("DataflowBoundary")  
public class PaymentGatewayAdapter {  
  public Mono<PaymentResultDto> processPaymentIntentAsync(PaymentIntentDto intent) { ... }  
}  
```  

### 3. Cross-Stack Sync (Backend-to-Frontend Alignment)  
**Why:** Ensures seamless data handoff, preventing type mismatches and enabling reactive streams to flow naturally to UI props.  

**How:** Frontend types/props must mirror backend DTOs; reactive patterns (e.g., `Flux<T>`) translate to arrays or observables in TS.  

**Validation Rule:** TypeScript compile-time checks for parity (e.g., via shared DTO schemas).  

**Examples:**  
```java  
// Backend: Reactive stream  
public Flux<PaymentEventDto> streamProcessedPaymentsAsync(String merchantId) { ... }  
```  

```typescript  
// Frontend: Vue component mirroring backend  
<script setup lang="ts">  
interface Props {  
  paymentEvents: PaymentEventDto[];  // Direct match to Flux<PaymentEventDto>  
}  
defineProps<Props>();  
</script>  
```  

| Backend Pattern | Frontend Equivalent      | Validation Rule              |  
|-----------------|--------------------------|------------------------------|  
| `Flux<T>`       | `T[]` or `Observable<T>` | TypeScript/ESLint type check |  
| `Mono<T>`       | `T`                      | Prop type parity             |  
| `*Dto`          | Interface matching DTO   | Shared schema enforcement    |  

---

## 🧠 Vibe Coding Imperatives  
- **Clarity First:** Names should evoke the "vibe" of the operation (e.g., real-time streaming vs. batch processing) without ambiguity.  
- **Reactive Purity:** All non-blocking flows must signal asynchronicity; default to reactive unless explicitly synchronous.  
- **LLM-Friendly:** Patterns optimized for AI inference—e.g., no vague terms; aim for 90% discoverability score via automated scans.  
- **Anti-Patterns:** Ban generics (`data`, `input`, `output`); enforce domain-specificity in public APIs.  

---

## ⚙️ Enforcement Mechanics  
**Tools and Rules:**  
| Rule                  | Tool/Validator                  | Action on Violation              |  
|-----------------------|---------------------------------|----------------------------------|  
| Naming Pattern        | Custom Gradle Task + Checkstyle | Fail build; suggest fixes        |  
| Role Annotations      | Annotation Processor            | Warn on missing `@Semantic`      |  
| Cross-Stack Sync      | TypeScript Compiler + ESLint    | Error on type mismatch           |  
| Vibe Compliance       | LLM-based Scanner (optional)    | Report low discoverability score |  

**CI Pipeline Integration:**  
```kotlin  
// build.gradle.kts (Gradle task for validation)  
tasks.register("validateDiscoverability") {  
  dependsOn("backend:compileJava", "frontend:compileTypeScript", "backend:checkstyleMain")  
  doLast {  
    exec { commandLine("node", "scripts/validate-naming-conventions.js") }  // Custom JS for semantic scan  
    // Optional: Integrate LLM tool for vibe scoring  
  }  
}  
```  

**Additional Tooling:**  
- ESLint rule for Vue/TS: Enforce prop naming mirrors DTOs.  
- PR Hooks: Run `validateDiscoverability` in CI; block merges if score < 90%.  

---

## 🔧 Technical Specifications  

### Java/WebFlux (Backend)  
- **Module Structure:** Use `@ModuleBoundary` on entry classes; organize as `<Domain>Module` (e.g., `OrderModule.java`).  
- **Reactive Focus:** Prefer `Flux`/`Mono` with async suffixes; annotate data flows with `@DataFlowDirection("inbound" | "outbound")`.  
```java  
@Semantic("PaymentBoundary")  
public class PaymentProcessingService {  
  @Vibe("Real-time inventory reservation and payment validation")  
  public Mono<OrderConfirmation> reserveAndValidateInventoryAsync(OrderRequestDto request) { ... }  
}  
```  

### Vue/TypeScript (Frontend)  
- **Component Design:** Suffix with `*Component`; props must align with backend DTOs.  
- **Reactive Handling:** Use arrays for streams; integrate with Vue's reactivity.  
```vue  
<template>  
  <div v-for="event in paymentEvents" :key="event.id">...</div>  
</template>  

<script setup lang="ts">  
interface Props {  
  paymentEvents: PaymentEventDto[];  // Mirrors backend Flux  
}  
defineProps<Props>();  
</script>  
```  

---

## 📋 Compliance Checklist  
- [ ] All public methods follow `[Verb][Domain][Context]` pattern with reactive suffixes where applicable.  
- [ ] Domain roles are indicated via suffixes (e.g., `*Adapter`) and `@Semantic` annotations.  
- [ ] Vue props and TS types mirror backend DTO structures (cross-stack parity).  
- [ ] No ambiguous/generic names (e.g., `handleData`, `processInput`); vibe clarity achieved.  
- [ ] Enforcement tasks (e.g., `validateDiscoverability`) integrated into CI.  

---

## 🔗 Interlocks  
- **Naming DNA:** Extends suffix/prefix rules from [06-naming-conventions.md](../02-semantics/06-naming-conventions.md).  
- **Modularity Boundaries:** Complements dependency isolation in [27-dependency-rules.md](../07-modularity/27-dependency-rules.md) and [04-modular-architecture.md](../01-project/04-modular-architecture.md).  
- **Build & Frontend Standards:** Ties into Gradle multimodule setup [22-gradle-multimodule.md](../06-build/22-gradle-multimodule.md) and TS styling [10-typescript-style.md](../03-coding-standards/10-typescript-style.md).  

---

## 📜 Revision History  
| Version | Date       | Author        | Changes Summary                              |
|---------|------------|---------------|----------------------------------------------|
| 1.0     | 2026-01-25 | LLM-Generated | Initial draft with core structure.           |

---

## Implementation Guide  
1. **Backend Setup:** Annotate services with `@Semantic` and enforce naming in new modules (e.g., `src/main/java/payments/OrderModule.java`).  
2. **Frontend Alignment:** Define TS interfaces matching backend DTOs; update Vue components to use mirrored props.  
3. **Tooling Rollout:** Add `validateDiscoverability` Gradle task and ESLint rules; test in a sample PR.  
4. **Customization:** Fill TODO fields (Domain, Owner); scan existing code for retrofits.  
5. **Metrics:** Track adoption via CI pass rates; aim for 100% compliance in new features.  

*Example Domain Folder Structure:*  
```bash  
/src/main/java/  
└── payments/  
    ├── PaymentModule.java              # @Semantic("PaymentProcessing")  
    ├── PaymentStreamService.java       # streamProcessedPaymentsAsync()  
    └── dto/  
        └── PaymentEventDto.java        # Mirrors TS interface  
```  
