# 04 – Modular Architecture – Boundaries & Domain Isolation

**File Path:** rules/01-project/04-modular-architecture.md  
**Domain:** TODO: <business-context> (e.g., "Secure e-commerce payment processing with real-time event streaming")  
**Last Updated:** 2026-01-22
**Status:** Active  

<!-- FILE_TITLE: Modular Architecture Foundations -->  
<!-- FILE_PATH: rules/01-project/04-modular-architecture.md -->  

---

## 🎯 Purpose  
Establishes a strict hierarchical module structure with enforced boundaries, mandatory facade APIs, reactive communication protocols, and comprehensive build-time validation. This ensures domain isolation, loose coupling, and maintainability across Java/Spring Boot/WebFlux backend modules, while aligning frontend Vue components with backend contracts for seamless integration.  

The approach prevents dependency entanglement, promotes reactive-first design, and scales for complex systems like e-commerce payment processing.  

## 📌 Scope  
- **Applies to:** All backend modules (e.g., under `backend/modules/`), inter-module dependencies, API facades, DTO contracts, and Vue frontend type syncing.  
- **Excludes:** Third-party libraries, generated client code, infrastructure adapters for external systems.  
- **Dependencies:** Integrates with `05-vibe-coding.md` (semantic annotations), `06-naming-conventions.md` (naming/DTO patterns), `15-testing-strategy.md` (ArchUnit validation), and `22-gradle-multimodule.md` (build configuration).  

---

## 🧠 Semantic Anchors (Core Concepts)  

### 1. Hierarchical Module Structure  
Modules are organized in a rooted tree topology to enforce clear parent-child-sibling relationships and prevent sprawling dependencies.  

- **Levels Defined:**  
  - **Level 0 (Root):** Core aggregator (e.g., `core-module` – shared utilities and domain primitives).  
  - **Level 1:** Primary feature modules (e.g., `payment-module`, `order-module`).  
  - **Level 2+:** Sub-feature/child modules (e.g., `fraud-module` under `payment-module`; access restricted beyond direct parents).  

**Visual Tree Example:**  
```text  
core-module (Level 0)  
├── payment-module (Level 1)  
│   └── fraud-module (Level 2 – child)  
└── order-module (Level 1 – sibling)  
    └── shipping-module (Level 2 – child)  
```  

**Access Rules:**  
- ✅ **Allowed:**  
  - Same-level siblings (via public APIs only).  
  - Parents to direct children (via child APIs).  
  - Children to parents (via parent facades for reverse dependencies).  
- ❌ **Forbidden:**  
  - Direct access to grandchildren or deeper (e.g., `core-module` cannot directly import `fraud-module`).  
  - Circular dependencies at any level.  
  - Internal entity access across modules (use DTOs exclusively).  

**Code Enforcement Example:**  
```java  
// ❌ Forbidden: Direct grandchild import  
// import com.project.payment.fraud.domain.FraudCheck;  

// ✅ Allowed: Via parent facade  
import com.project.payment.api.PaymentFacade;  
PaymentFacade facade = ...;  
facade.processPaymentWithFraudCheck(request);  
```  

### 2. Facade API Enforcement  
Every module **must** expose a dedicated public API layer to abstract internals and define contracts.  

- **Structure:**  
  ```
  <module-name>/  
  ├── api/                    # Public facade (interfaces only, no impls)  
  │   ├── <ModuleName>Facade.java  
  │   └── dto/                # Immutable DTOs and events  
  ├── domain/                 # Internal aggregates/entities (@Semantic("Internal"))  
  ├── application/            # Use cases/services  
  └── infrastructure/         # Repos/adapters  
  ```  

- **Mandatory Annotations:**  
  ```java  
  @Semantic("ModuleFacade")  
  @Vibe("PublicAPI")  
  public interface PaymentFacade {  
      @Semantic("ReactiveBoundary")  
      @Vibe("AtomicPaymentStream")  
      Flux<PaymentEvent> streamPaymentsForOrder(String orderId);  
  }  
  ```  

- **DTO Guidelines:**  
  - Immutable records or final classes implementing `Serializable`.  
  - Annotated with `@Semantic("Contract")`.  
  - No business logic; purely for data transfer.  
  ```java  
  @Semantic("Contract")  
  public record PaymentEvent(  
      @Semantic("PaymentID") UUID id,  
      BigDecimal amount,  
      Instant processedAt,  
      PaymentStatus status  
  ) implements Serializable {}  
  ```  

### 3. Dependency Governance & Validation Matrix  
Strict rules govern flows to maintain isolation.  

| Relationship          | Allowed? | Method/Example                                | Enforcement Tool                  |  
|-----------------------|----------|-----------------------------------------------|-----------------------------------|  
| Parent → Child        | ✅       | Via child's `api/` (e.g., `fraud-module/api`) | ArchUnit `LayeredArchitecture`    |  
| Sibling → Sibling     | ✅       | Via sibling's facade/DTOs only                | Gradle dependency analysis        |  
| Child → Parent        | ✅       | Via parent's facade                           | Static analysis (e.g., SonarQube) |  
| Module → Grandchild   | ❌       | Forbidden (route via parent chain)            | Custom `NoGrandchildAccessRule`   |  
| Any → Internal Entity | ❌       | Use DTOs; no domain model exposure            | ArchUnit `ApiOnlyImportsRule`     |  

**Reactive Mandate:** All cross-module methods return `Mono<T>` or `Flux<T>`; blocking calls trigger build failure via `@BlockingGuard`.  

### 4. Hierarchical Communication Protocol  
- **DTO-Only Exchange:** Modules communicate via facades and events; entities stay internal.  
- **Frontend Alignment:** Vue props mirror backend DTOs (e.g., Java `UUID` → TS `string`, `BigDecimal` → `number`).  
  ```typescript  
  // frontend/src/models/PaymentEvent.ts (auto-generated)  
  interface PaymentEvent {  
      id: string;  
      amount: number;  
      processedAt: Date;  
      status: 'PENDING' | 'COMPLETED';  
  }  
  ```  
- **Event-Driven Flows:** Use `Flux` for streams; ensure no direct method calls bypass facades.  

**Mermaid Diagram (Dependency Graph):**  
```mermaid  
graph TD  
    Core[Core Module (L0)] --> Payment[Payment Module (L1)]  
    Core --> Order[Order Module (L1)]  
    Payment --> Fraud[Fraud Module (L2)]  
    Order --> Shipping[Shipping Module (L2)]  
    Payment -.->|API Call| Order  
    Core -.->|✅ Sibling/Child| Payment  
    Core -.X.->|❌ Forbidden| Fraud  
    style Fraud stroke:#ff0000,stroke-width:2px  
```  

---

## 🔧 Technical Specifications  

### Cross-Module Interaction Examples  
**Allowed Sibling Call (Reactive):**  
```java  
// In order-module/application/OrderService.java  
@Service  
public class OrderService {  
    private final PaymentFacade paymentFacade;  

    public Mono<Order> processOrder(OrderRequest request) {  
        return paymentFacade.streamPaymentsForOrder(request.getOrderId())  
            .filter(event -> event.status() == PaymentStatus.COMPLETED)  
            .collectList()  
            .flatMap(events -> createOrderWithPayments(request, events));  
    }  
}  
```  

**Forbidden Grandchild Access:**  
```java  
// ❌ Violation  
// import com.project.payment.fraud.domain.FraudCheckResult;  

// ✅ Remediation: Use parent facade  
Mono<PaymentEvent> result = paymentFacade.processWithFraudCheck(request);  
```  

### Frontend-Backend Sync  
- **Generation Script:** `./gradlew generateTypescriptTypes` (from OpenAPI schema or DTOs).  
- **Validation:** CI compares JSON schemas vs. TS interfaces; mismatches fail build.  

### Build-Time Validation Stack  
- **Gradle Task (`validateModuleHierarchy`):**  
  ```kotlin  
  tasks.register("validateModuleHierarchy") {  
      dependsOn("compileJava", "compileTestJava")  
      doLast {  
          // ArchUnit execution  
          javaexec {  
              mainClass.set("com.project.archunit.ModuleHierarchyTest")  
              classpath = sourceSets.main.get().runtimeClasspath  
              args("--rules=NoGrandchildAccess,NoApiOnlyImports")  
          }  
          // Custom AST scan for imports  
          val forbiddenPaths = setOf("**/fraud/**", "**/internal/**")  
          if (hasInvalidImports(forbiddenPaths)) {  
              throw GradleException("Hierarchy violation: Direct grandchild access detected!")  
          }  
          // DTO sync check  
          exec { commandLine("./scripts/validate-dto-sync.sh") }  
      }  
  }  
  ```  

- **ArchUnit Rules Example:**  
  ```java  
  @ArchTest  
  static final ArchRule NO_GRANDCHILD_DEPENDENCIES =  
      noClasses().that().resideInAPackage("..level0..", "..level1..")  
                 .should().accessClassesThat().resideInAPackage("..level2..");  

  @ArchTest  
  static final ArchRule API_ONLY_ACCESS =  
      classes().that().resideOutsideOfAPackage("..api..")  
               .should().onlyAccessClassesThat()  
               .resideInAPackage("..api..")  
               .orShould().beInterfaces();  
  ```  

- **CI/CD Pipeline:**  
  ```
  build → validateModuleHierarchy → ArchUnitTests → DTO Frontend Match → SonarQube Scan → deploy  
  ```  

---

## ⚙️ Enforcement Mechanics  

| Violation Type          | Detection Method                  | Auto-Remediation/Action                   |  
|-------------------------|-----------------------------------|-------------------------------------------|  
| Grandchild Access       | ArchUnit + Gradle AST Parser      | Suggest facade routing; auto-comment code |  
| Missing Facade/API      | Package Structure Scanner         | Generate scaffold template                |  
| Blocking Cross-Module   | @BlockingGuard + Reactor Agent    | Convert to Mono/Flux; fail test           |  
| DTO Mismatch (BE/FE)    | Schema Comparator (JSON ↔ TS)     | Auto-sync TS interfaces                   |  
| Circular Dependencies   | Gradle Dependency Insight         | Report graph; block merge                 |  

- **Additional Tools:**  
  - `@Semantic("ForbidGrandchildAccess")` for static tagging.  
  - `./gradlew exportModuleGraph` to generate Mermaid/PNG diagrams in CI.  
  - Use `module-info.java` for JVM-level package visibility.  

---

## 📋 Compliance Checklist  
- [ ] Modules follow tree hierarchy with `<domain>-module` naming.  
- [ ] Every module has an `api/` package with `@Semantic("ModuleFacade")` interfaces.  
- [ ] No direct grandchild dependencies or internal entity imports.  
- [ ] All cross-module calls are reactive (`Mono`/`Flux`) with `@Semantic("ReactiveBoundary")`.  
- [ ] Vue TS interfaces sync with Java DTOs via generation script.  
- [ ] `validateModuleHierarchy` task integrated in CI; passes without errors.  
- [ ] ArchUnit rules cover all boundary checks.  

---

## 🔗 Interlocks with Other Rules  
- **Vibe Coding (`05-vibe-coding.md`):** Leverages `@Semantic` and `@Vibe` for annotations.  
- **Naming Conventions (`06-naming-conventions.md`):** DTOs end in `*Dto` or `*Event`.  
- **Testing Strategy (`15-testing-strategy.md`):** Includes ArchUnit in unit/integration suites.  
- **Gradle Multimodule (`22-gradle-multimodule.md`):** Manages hierarchical dependencies.  

---

## 📜 Revision History  
| Version | Date       | Author          | Changes Summary                                      |  
|---------|------------|-----------------|------------------------------------------------------|  
| 1.0     | 2026-01-22 | LLM-Generated   | Basic hierarchy and facade rules                     |  

---

**Key Innovations:**  
1. **Tree Topology with Strict Rules:** Prevents "spaghetti dependencies" via forbidden grandchild access.  
2. **Facade-First Design:** Abstracts internals, enabling easy refactoring.  
3. **Reactive Isolation:** Ensures non-blocking flows for high-throughput systems like payments.  
4. **Automated Validation:** Build/CI fails early; generates reports and fixes.  
5. **Full-Stack Alignment:** DTOs bridge backend/frontend without manual overhead.  

**Implementation Notes:**  
1. Pilot on core modules (e.g., `payment-module`); use `@Vibe("StrictHierarchy")` for gradual rollout.  
2. Reference `/reference/arch-diagrams/` for sample graphs and templates.  
3. For violations, consult generated `module-hierarchy-issues.md` artifact.  
4. Extend with OpenAPI for facade docs; ensure Serializable DTOs for potential serialization needs.