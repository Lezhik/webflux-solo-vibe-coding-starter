# 26 – Dependency Rules for Modular Architecture

**File Path:** rules/07-modularity/26-dependency-rules.md  
**Domain:** TODO: <fill with project domain, e.g., "E-commerce payment processing">  
**Last Updated:** 2026-01-25
**Status:** Active  

<!-- FILE_TITLE: Modular Dependency Governance -->  
<!-- FILE_PATH: rules/07-modularity/26-dependency-rules.md -->

---

## 🎯 Purpose
Defines strict dependency boundaries between modules to prevent circular dependencies, enforce loose coupling, and promote reactive, DTO-based communication patterns. This ensures semantic isolation across backend (Java/Spring WebFlux) and frontend (Vue/TypeScript) components, facilitating AI-assisted refactoring and human-readable architecture.

## 📌 Scope
- **Applies to:** All Java modules under `backend/` (e.g., `user-module`, `payment-module`), Vue components consuming backend DTOs, and inter-module imports in the Gradle multimodule build.
- **Excludes:** Third-party libraries, transient dependencies, non-critical internal frontend assets.
- **Dependencies:** See `04-modular-architecture.md` for base module structure and `22-gradle-multimodule.md` for build sequencing.

---

## 🧠 Semantic Anchors

### 1. Annotation Taxonomy for Intent Signaling
Use custom annotations to mark boundaries and roles, enabling automated analysis and vibe coding:
```java
// Marks architectural boundaries and roles
@Semantic("PaymentGatewayAdapter")  // e.g., DomainBoundary, ReactiveGateway
@Vibe("Loose coupling for async payments")  // e.g., ReactiveIsolation, AsyncOnly, Eventual consistency guarantee
public class PaymentAdapter {
    // Cross-module: Interface only, no concrete implementations
    private final PaymentService paymentService; 
}
```

### 2. Dependency Boundaries
**Allowed Flow (Acyclic Chain):**
```
Domain (Module A) → Service → DTO/Interface → Controller/Presentation
```
- Cross-module: Services may depend on interfaces/DTOs from other modules.
- Reactive streams (Flux/Mono) for async interactions.

**Forbidden Patterns:**
```java
// ❌ Controller directly accessing foreign repository (cross-module violation)
@RestController
public class BadController {
    @Autowired
    private UserRepository userRepo; // From different module
}

// ❌ Service accessing concrete class (use adapters/interfaces)
@Service
public class TightCoupledService {
    private final OtherModuleConcreteImpl impl; // Forbidden; use interface
}

// ❌ Circular: Module A → B → A
```

**Cross-Module Communication Mandates:**
- Always convert to DTOs before crossing boundaries.
- Use adapters or event buses (e.g., `DomainEventBus`) for indirect coupling.

---

## ⚙️ Enforcement Mechanics

| Rule Type              | Tool/Mechanism                   | Validation Criteria                          | Action on Violation       |
|------------------------|----------------------------------|----------------------------------------------|---------------------------|
| Circular Dependencies  | Gradle Task (`dependencyCheck`)  | Detect cycles in module graph                | Fail build/PR             |
| Layer Violations       | ArchUnit Tests                   | Block imports like controller→repository     | Fail tests                |
| DTO Purity (Frontend)  | ESLint/TypeScript Compiler       | Ensure only typed DTOs are used in Vue       | Type errors/warnings      |
| Semantic Annotation    | Custom Lint Rule/Static Analysis | Verify @Semantic/@Vibe on boundaries         | IDE warnings in CI        |

**Gradle Enforcement Template:**
```kotlin
// build.gradle.kts (root)
tasks.register("validateDependencies") {
    dependsOn(":backend:compileJava")
    doLast {
        // Conceptual cycle detection (integrate with dependency-analyze plugin)
        if (hasCyclicDependencies()) {
            throw GradleException("Module dependency cycle detected!")
        }
        // Run ArchUnit
        exec {
            commandLine("java", "-jar", "archunit.jar", "--packageInclude=com.example..*")
        }
    }
}
```

**ArchUnit Test Example:**
```java
@ArchTest
static final ArchRule noControllerToRepo = 
    noClasses().that().resideInAPackage("..controller..")
    .should().accessClassesThat().resideInAPackage("..repository..");
```

---

## 🔧 Technical Specifications

### Java/WebFlux Module Interaction
**Correct Cross-Module Example:**
```java
// In payment-module
@Semantic("DomainBoundary")
@Vibe("ReactiveIsolation")
public interface PaymentService {
    Mono<PaymentResultDto> processAsync(PaymentRequestDto dto);
}

// In user-module (consuming module)
@Semantic("CrossModuleAdapter")
public class UserPaymentAdapter {
    private final PaymentService paymentService; // Interface only!

    public Flux<OrderEvent> publishOrderStream(String orderId) {
        return orderService.getEvents(orderId)
            .map(this::convertToDTO); // Always DTO
    }
}
```

### Vue/TypeScript Consumption
**Shared DTO Definition:**
```typescript
// frontend/shared/dto/order.ts (or payment.ts)
export interface OrderDTO {
    id: string;
    status: 'PENDING' | 'COMPLETED';
}

export interface PaymentResultDto {
    transactionId: string;
    status: 'SUCCESS' | 'FAILED';
}
```

**Correct Frontend Usage:**
```typescript
// frontend/src/modules/user/payment.vue
<script setup lang="ts">
import type { OrderDTO, PaymentResultDto } from '@/shared/dto';

// Consume via API, no direct backend entity access
const { data: orders } = await useFetch<OrderDTO[]>('/api/orders');
const { data: payment } = await useFetch<PaymentResultDto>('/api/payments');
</script>
```

**Versioning for DTOs:** Use `/static/v1/dto` paths for cache-busting and backward compatibility.

---

## 📋 Compliance Checklist
- [ ] No direct `@Autowired` of foreign module repositories or concrete classes.
- [ ] All cross-module returns use DTOs, Flux/Mono, or interfaces.
- [ ] Frontend imports only from `shared/dto` (no backend internals).
- [ ] ArchUnit tests enforce layer boundaries (controller → repo, etc.).
- [ ] `@Semantic` and `@Vibe` annotations on all boundary classes/interfaces.
- [ ] `validateDependencies` Gradle task integrated into CI pipeline.
- [ ] Mermaid dependency diagrams updated in `/docs` for visualization.

---

## 🔗 Interlocks
- **Naming:** Aligns with `06-naming-conventions.md` (e.g., DTO suffixes like `*Dto`, adapter classes).
- **Build:** Depends on `22-gradle-multimodule.md` for task ordering and cycle detection.
- **Testing:** References `17-test-pyramid.md` for boundary unit/integration tests.
- **API/Static:** Ties into `14-mermaid-diagrams.md` for flow visualization and `23-static-resources.md` for versioned DTOs.
- **Modularity:** Builds on `04-modular-architecture.md` for module definitions.

---

## 📜 Revision History
| Version | Date       | Author          | Changes Summary                          |
|---------|------------|-----------------|------------------------------------------|
| 1.0     | 2026-01-25 | LLM-Generated   | Initial creation with core rules         |

---

## Implementation Guide
1. **Apply Annotations:** Tag all cross-module interfaces and adapters with `@Semantic` and `@Vibe` for intent signaling.
2. **Create Shared DTOs:** Set up a `shared-dto` module (or frontend `shared/dto` folder) for BE/FE alignment; version as needed (e.g., `v1/OrderDTO`).
3. **Add Enforcement:**
   - Integrate ArchUnit tests in each module's `src/test/java`.
   - Add `validateDependencies` to root `build.gradle.kts` and run in CI.
   - Configure ESLint for TypeScript DTO strictness.
4. **Validate Boundaries:** Run `./gradlew validateDependencies` before PRs; use tools like `gradle-dependency-analyze` for cycles.
5. **Document Flows:** Generate Mermaid diagrams:
   ```
   graph TD
       A[Domain A] --> B[Service A]
       B --> C[DTO/Interface]
       C --> D[Controller]
       E[Domain B] --> F[Adapter]
       F --> C
   ```
6. **Branching & PRs:** Use `feat/[domain]-dependency` (e.g., `feat/payment-boundary`); require compliance checks.

**Differentiators:**
- **AI-Ready:** `@Semantic` + `@Vibe` for LLM-parsable graphs and refactoring suggestions.
- **Reactive-First:** Prioritizes Flux/Mono for boundaries to support non-blocking architecture.
- **Full-Stack Governance:** Unified DTO rules prevent leaks from BE to FE.
- **Automation Focus:** Gradle/ArchUnit integration ensures zero-tolerance enforcement without manual reviews.
