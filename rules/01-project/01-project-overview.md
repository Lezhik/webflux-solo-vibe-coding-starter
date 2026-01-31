# 01 – Project Overview – Foundation

**File Path:** rules/01-project/01-project-overview.md  
**Domain:** TODO: <business-context> (e.g., "Secure e-commerce payment processing with real-time event streaming")  
**Last Updated:** 2026-01-22  
**Status:** Active  

<!-- FILE_TITLE: Project Overview – Foundation -->  
<!-- FILE_PATH: rules/01-project/01-project-overview.md -->  

---

## 🎯 Purpose  
This document establishes the foundational principles, boundaries, semantic anchors, and constraints for the project. It ensures a cohesive, reactive-first architecture spanning the Java/Spring Boot WebFlux backend, Vue/TypeScript frontend, and Gradle-based build system. The focus is on type-safe contracts, non-blocking I/O, and vibe-driven coding to support scalable domain-specific workflows (e.g., payment processing or order fulfillment).

## 📌 Scope  
- **Applies to:**  
  - Backend modules (Java 21+, Spring Boot 3.5+, WebFlux, MongoDB reactive repositories).  
  - Frontend components (Vue 3.4+, TypeScript 5+, Composition API).  
  - Build and deployment (Gradle 8.5+, multimodule monorepo, static asset versioning).  
- **Excludes:**  
  - Blocking synchronous patterns (e.g., JDBC or RestTemplate without async wrappers).  
  - Unapproved third-party libraries (see `02-technology-stack.md`).  
  - Inline styles/scripts in frontend or unversioned static resources.  
- **Dependencies:**  
  - `02-technology-stack.md` for version pins and constraints.  
  - `03-folder-structure.md` for monorepo layout.  
  - `05-vibe-coding.md` for annotation and naming rules.  

---

## 🧠 Semantic Anchors  
### 1. Project Identity and Goals  
**Project Name:** TODO: `<project-name>` (e.g., "Kilo Code Platform")  
**Primary Domain:** TODO: `<primary-domain>` (e.g., "Real-time order fulfillment and payment automation")  

**Key Architectural Goals:**  
1. Achieve non-blocking, event-driven flows with WebFlux Mono/Flux for high-concurrency scenarios.  
2. Maintain strict DTO synchronization between backend and frontend for type safety.  
3. Enforce semantic intent through annotations to reduce cognitive load and enable automated tooling.  
4. Version static assets and integrate frontend builds seamlessly into backend JARs.  

### 2. Cross-Stack Contracts  
Semantic annotations like `@Semantic` (for domain roles) and `@Vibe` (for intent) are mandatory for public APIs. They facilitate code reviews, static analysis, and cross-module tracing.

```java
// Backend DTO example (reactive projections)
@Semantic("CoreEntity")
@Vibe("Represents immutable payment state for event streaming")
public record PaymentDto(String id, BigDecimal amount, PaymentStatus status) {}
```

```typescript
// Frontend interface mirroring backend DTO
interface PaymentDto {
  id: string;
  amount: number;  // Parsed from BigDecimal
  status: 'PENDING' | 'COMPLETED' | 'FAILED';
}
```

### 3. Boundary Markers and Naming  
- Reactive methods must use suffixes like `*Flux` or `*Mono` (e.g., `streamPaymentsFlux()`).  
- Avoid ambiguous names (e.g., no plain `getPayments()`; use `fetchActivePaymentsAsync()`).  
- Example interface:  
  ```java
  @Vibe("Async boundary for idempotent payment settlement with retry")
  @Semantic("PaymentGateway")
  public interface PaymentService {
      Mono<PaymentResult> processPaymentAsync(PaymentRequest request);
      Flux<PaymentEvent> streamPaymentEventsFlux(String userId);
  }
  ```

### 4. Domain Context  
TODO: `<domain-context>` (2-3 sentences describing the business problem).  
Example: "This project powers a secure e-commerce platform handling concurrent payment requests. It supports real-time inventory updates via WebFlux streams and provides a responsive Vue frontend for customer interactions, ensuring eventual consistency across microservices."

---

## 🔧 Technical Specifications  
### Backend (Java/Spring Boot WebFlux)  
- **Reactive Patterns:** Default to `Mono` for single results, `Flux` for streams. Use Project Reactor operators (e.g., `flatMap`, `retryWhen`).  
- **Static Resources:** Serve versioned assets from `/src/main/resources/static/v[version]/` (e.g., `/static/v1/app.js`).  
- **Example Endpoint:**  
  ```java
  @RestController
  @Semantic("OrderStreamController")
  public class OrderController {
      @GetMapping("/orders/{userId}/stream")
      @Vibe("Server-sent events for live order updates")
      public Flux<OrderEvent> streamOrdersFlux(@PathVariable String userId) {
          return orderService.streamUserOrdersAsync(userId);
      }
  }
  ```

### Frontend (Vue/TypeScript)  
- **Structure:** Use Composition API with typed props/interfaces. Mirror backend DTOs; avoid `any` types.  
- **Asset Rules:** Externalize styles/scripts; defer loading for performance (e.g., `<script src="/static/v1/app.js" defer>`).  
- **Example Component:**  
  ```vue
  <template>
    <div v-for="event in orderEvents" :key="event.id">
      {{ event.title }}
    </div>
  </template>

  <script setup lang="ts">
  import { Ref, ref } from 'vue';
  import type { OrderEvent } from '@/types/OrderEvent';  // Mirrors backend DTO

  interface Props {
    userId: string;
  }
  const props = defineProps<Props>();
  const orderEvents: Ref<OrderEvent[]> = ref([]);
  
  // Fetch via reactive service
  onMounted(() => {
    fetchOrderStream(props.userId).then(events => orderEvents.value = events);
  });
  </script>
  ```

### Build (Gradle Multimodule)  
- **Integration:** Frontend build outputs to backend's static resources. Use tasks for validation.  
- **Example Task (build.gradle.kts):**  
  ```kotlin
  tasks.register("validateFoundations") {
      dependsOn("backend:checkstyleMain", "frontend:eslint")
      doLast {
          // Custom check for @Semantic/@Vibe presence
          logger.lifecycle("Validating semantic anchors...")
      }
  }
  
  tasks.named("buildFrontend") {
      doLast {
          copy {
              from("$buildDir/frontend/dist")
              into("$projectDir/src/main/resources/static/v1")
          }
      }
  }
  ```

---

## ⚙️ Enforcement Mechanics  
| Rule Type            | Tool/Mechanism                    | Failure Action              |  
|----------------------|-----------------------------------|-----------------------------|  
| Semantic Annotations | Checkstyle + Custom Gradle Plugin | Fail Build / Block PR       |  
| Reactive Naming      | IntelliJ Inspections + ESLint     | Warn / Auto-Format on Save  |  
| DTO Synchronization  | TypeScript Compiler + ErrorProne  | Type Errors / Build Fail    |  
| Static Assets        | Gradle Resource Tasks + Vite      | Exclude Unversioned Files   |  

**CI/CD Integration:**  
- GitHub Actions/PR workflows run `./gradlew validateFoundations` and scan for TODOs/missing annotations.  
- SonarQube for code smells; require 100% coverage of public APIs with `@Vibe` tags.

---

## 📋 Compliance Checklist  
- [ ] Project name, domain, and goals defined (replace all TODOs).  
- [ ] Core DTOs synchronized between Java/TS with `@Semantic` tags.  
- [ ] All public methods use reactive suffixes and `@Vibe` annotations.  
- [ ] Static assets versioned and integrated via Gradle.  
- [ ] Initial Mermaid diagrams linked (see `14-mermaid-diagrams.md`).  
- [ ] Enforcement tasks pass locally (`./gradlew validateFoundations`).

---

## 🔗 Interlocks with Other Rules  
- **Tech Stack:** Versions pinned in `02-technology-stack.md` (e.g., no WebMVC for reactive paths).  
- **Naming & Vibe:** Detailed in `05-vibe-coding.md` and `06-naming-conventions.md` (e.g., `*Dto` suffixes).  
- **Build Sequencing:** `22-gradle-multimodule.md` for frontend-to-backend asset copying.  
- **Reviews:** Check `@dataflow` tags in `16-review-checklist.md` for module ownership.

---

## 📜 Revision History  
| Version | Date       | Author             | Changes Summary                          |  
|---------|------------|--------------------|------------------------------------------|  
| 1.0     | 2026-01-22 | LLM-Generated      | Basic structure and TODO placeholders    |  

---

## Implementation Guide  
1. **Customize TODOs:** Fill in project name, domain context, and goals based on business requirements.  
2. **Branch Setup:** Start from `main` with `feat/foundation-setup-[domain]` (e.g., `feat/foundation-setup-payments`).  
3. **Validate Locally:** Run `./gradlew clean validateFoundations` to check annotations and builds.  
4. **Add Diagrams:** Create initial Mermaid flowcharts for data boundaries (reference `14-mermaid-diagrams.md`).  
5. **Enhance Enforcement:** Integrate custom Checkstyle rules for `@Semantic` detection post-setup.  

**Differentiators:**  
- `@dataflow` extensions for tracing ownership (e.g., `@dataflow(backend:PaymentService → frontend:PaymentView)`).  
- Automated vibe validation in CI to prevent drift from reactive principles.  
- Focus on "3-second clarity test" for all names/annotations to boost team velocity.

> **Note:** This foundation file interlinks with all subsequent rules. Update after major domain shifts.