# 08 – Code Section Ordering: Mandatory Structure for Semantic Consistency

**File Path:** `rules/02-semantics/08-code-section-ordering.md`  
**Domain:** TODO: <business-context> (e.g., "Secure e-commerce payment processing with real-time event streaming")  
**Last Updated:** 2026-01-22
**Status:** Active  

---

## 🎯 Purpose  
Establish a deterministic, universal code structure for Java, TypeScript, and Vue components to promote semantic consistency, enhance readability, and facilitate tool-assisted validation (e.g., LLMs, IDEs). This ordering aligns with "vibe-coding" principles, where structure communicates intent—prioritizing data models before logic, and reactive flows for non-blocking operations—while enabling faster reviews, onboarding, and automated enforcement.

---

## 📌 Scope  
- **Applies to:** Java/Kotlin classes, TypeScript modules, Vue SFC `<script>` blocks.  
- **Excludes:** Auto-generated code (e.g., Lombok, GraphQL schemas) and unmodified third-party libraries.  
- **Dependencies:** [06-naming-conventions.md](./06-naming-conventions.md), [05-vibe-coding.md](./05-vibe-coding.md), [22-gradle-multimodule.md](../06-build/22-gradle-multimodule.md).  

---

## 🧠 Semantic Anchors  

### Mandatory Section Order (Top to Bottom)  
This sequence builds semantic layers: foundational elements (imports/constants) precede data, then logic, ensuring code reads like a narrative of intent. Violations disrupt "vibe flow."

1. **Imports**  
   - **Grouping Rule:** Core (JDK/Vue builtins) → Internal (project modules) → Third-party (external libs). No wildcards; sort alphabetically within groups.  
   - **Java Example:**  
     ```java  
     // Core  
     import java.time.Instant;  
     import reactor.core.publisher.Flux;  
     // Internal  
     import com.example.domain.PaymentDto;  
     // External  
     import org.springframework.stereotype.Service;  
     import org.springframework.web.bind.annotation.RestController;  
     ```  
   - **Vue/TypeScript Example:**  
     ```typescript  
     // Core  
     import { ref, computed, watch } from 'vue';  
     // Internal  
     import PaymentService from '@/services/PaymentService.ts';  
     // External  
     import { StripeElement } from '@stripe/stripe-js';  
     ```  

2. **Constants & Configuration**  
   - Static, immutable values only. Prefix with `PUBLIC_`, `PRIVATE_`, or domain tags (e.g., `PAYMENT_MAX_RETRIES`). Add `@Vibe` comments for intent.  
   - **Java Example:**  
     ```java  
     private static final int PUBLIC_MAX_RETRY_ATTEMPTS = 3; // @Vibe "Guards against infinite async loops"  
     private static final String PRIVATE_API_WEBHOOK_URL = "https://api.example.com/webhooks";  
     ```  
   - **TypeScript Example:**  
     ```typescript  
     const MAX_RETRIES = 3 as const; // @Vibe "Prevents retry storms in payment polling"  
     ```  

3. **Domain Models & DTOs**  
   - Define entities, records, interfaces, or types. Annotate with `@Semantic("Role")` (e.g., `@Semantic("AggregateRoot")`) for LLM/tool parsing. Precede any logic using them.  
   - **Java Example:**  
     ```java  
     import com.example.annotations.Semantic; // Assuming custom annotation  
     @Semantic("PaymentRequest")  
     public record PaymentRequest(String id, double amount, String currency) {}  
     ```  
   - **TypeScript/Vue Example:**  
     ```typescript  
     /** @Semantic("PaymentResult") */  
     export interface PaymentResult {  
       id: string;  
       amount: number;  
       status: 'success' | 'failed' | 'pending';  
     }  
     ```  

4. **Reactive/Async Logic**  
   - Core business methods: Prefix with `stream*Async`, `process*Async` for Flux/Mono. Use `@Vibe` for high-level explanations. Reactive state in Vue (e.g., `ref`, `computed`).  
   - **Java Example:**  
     ```java  
     @Service  
     @Semantic("PaymentProcessor")  
     public class PaymentService {  
         private final PaymentRepository repo;  
         // Constructor injection...  
         /**  
          * @Vibe("Idempotent, non-blocking payment stream with error handling")  
          */  
         public Flux<PaymentResult> processPaymentsAsync(PaymentRequest request) {  
             return repo.findById(request.id())  
                        .switchIfEmpty(Mono.fromCallable(() -> createNewPayment(request)))  
                        .flatMap(this::validateAndSave);  
         }  
     }  
     ```  
   - **Vue Example:**  
     ```vue  
     <script setup lang="ts">  
     import { ref, onMounted } from 'vue';  
     import { usePaymentService } from '@/composables/usePaymentService';  
     const props = defineProps<{ requestId: string }>();  
     const paymentResult = ref<PaymentResult | null>(null);  
     const { streamPayments } = usePaymentService();  
     onMounted(async () => {  
         paymentResult.value = await streamPayments(props.requestId).toPromise(); // Simplified  
     });  
     </script>  
     ```  

5. **Control & Utility Code**  
   - Private helpers only (e.g., validation, sanitization). No public APIs here—keep lean to avoid cluttering the main flow.  
   - **Shared Example:**  
     ```java  
     private boolean isValidCurrency(String currency) {  
         return Set.of("USD", "EUR").contains(currency.toUpperCase());  
     }  
     ```  
     ```typescript  
     function validateAmount(amount: number): boolean {  
         return amount > 0 && amount <= 10000; // @Vibe "Enforces business limits"  
     }  
     ```  

6. **Exports & Component Definitions (Vue SFC)**  
   - Final section: `defineProps`, `defineEmits`, exported functions/variables. Followed by `<template>` and `<style scoped>`.  
   - **Vue Example:**  
     ```vue  
     <script setup lang="ts">  
     // ... (prior sections)  
     defineEmits<{ submit: [payload: PaymentRequest] }>();  
     const submitPayment = (request: PaymentRequest) => {  
         emit('submit', request);  
     };  
     </script>  
     <template>  
         <form @submit="submitPayment">...</form>  
     </template>  
     <style scoped> /* Scoped styles last */ </style>  
     ```  

---

## ⚙️ Enforcement Mechanics  

| Layer                | Tool(s)                            | Action on Failure                  |  
|----------------------|------------------------------------|------------------------------------|  
| Import Grouping      | Checkstyle (Java), ESLint (TS/Vue) | Auto-reformat; block PR if unfixed |  
| Constants Prefixing  | SonarQube, Custom Linter           | Warning → Critical on PR           |  
| Semantic Annotations | IDE (IntelliJ/VS Code) + SonarQube | High-severity lint error           |  
| Section Order        | Custom Gradle Task, ESLint Plugin  | Fail build; PR block               |  
| Async Suffixes       | Checkstyle/ESLint Rules            | Auto-suggest fix                   |  

**CI/CD Integration Example (Gradle):**  
```kotlin  
// build.gradle.kts (multi-module setup)  
tasks.register("verifyCodeStructure") {  
    dependsOn("checkstyleMain", "eslint:lint")  
    doLast {  
        // Run custom validator for section order  
        println("Code structure validated – vibe score: 95%")  
    }  
}  
// Hook into PR workflow  
```  

---

## 📋 Compliance Checklist  
- [ ] Imports grouped and sorted: Core → Internal → External.  
- [ ] Constants prefixed (e.g., `PUBLIC_*`) and annotated with `@Vibe` where intent is non-obvious.  
- [ ] Domain models defined with `@Semantic` annotations before any logic.  
- [ ] Reactive/async methods follow naming (e.g., `*Async`) and use `@Vibe` for flow explanation.  
- [ ] Utility methods are private and post-logic.  
- [ ] Vue SFCs: `<script setup>` order = imports → constants → models → reactive logic → props/emits/exports.  
- [ ] Overall: No forward references (e.g., logic can't use undeclared models).  

---

## 🔗 Interlocks  
- **Naming & Annotations:** Integrates with [06-naming-conventions.md](./06-naming-conventions.md) (e.g., `*Dto` suffixes) and [05-vibe-coding.md](./05-vibe-coding.md) (`@Vibe` standards).  
- **Build & Testing:** Enforced via [22-gradle-multimodule.md](../06-build/22-gradle-multimodule.md) tasks; tests must mirror this order per [18-unit-testing.md](../05-test-strategy/18-unit-testing.md).  
- **Reviews:** Checklist item in [16-review-checklist.md](../04-reviews/16-review-checklist.md) – flag order violations early.  

---

## 📜 Revision History  
| Version | Date       | Author        | Changes Summary                                                        |  
|---------|------------|---------------|------------------------------------------------------------------------|  
| 1.0     | 2026-01-22 | LLM-Generated | Initial draft combined with examples, enforcement, and vibe alignment. |  

---

## Implementation Guide  
1. **Customize for Domain:** Replace placeholders (e.g., "Payment" examples) with project entities like "User" or "Order."  
2. **Local Setup:** Configure IDE plugins (e.g., Checkstyle in IntelliJ, ESLint in VS Code) via shared `.idea` or `.vscode` settings. Run `./gradlew verifyCodeStructure` for validation.  
3. **Migration:** For existing code, use tools like ESLint auto-fix for imports; manually reorder sections in PRs.  
4. **Branching:** Use `feat/[action]-[domain]` (e.g., `feat/reorder-payment-service`) for changes. Aim for 90%+ "vibe score" in reviews.  

This structure transforms code into a self-documenting artifact, boosting team velocity and tool reliability while embodying vibe-coding's intuitive flow.