# 02 – Technology Stack – Foundation

**File Path:** rules/01-project/02-technology-stack.md  
**Domain:** TODO: `<Insert Domain Context>` (Example: "E-commerce payment processing")  
**Last Updated:** 2026-01-22  
**Status:** Active  

<!-- FILE_TITLE: Technology Stack Specification & Enforcement -->  

---

## 🎯 Purpose  
Defines the foundational technology stack, including mandatory versions, tooling constraints, and integration patterns for the Kilo Code monorepo. Ensures reactive-first development with alignment between Java/WebFlux backend, Vue/TypeScript frontend, and supporting infrastructure, while incorporating semantic anchoring for intent signaling and automated compliance.

## 📌 Scope  
- **Applies to:** Backend modules (Java/Spring Boot/WebFlux), frontend components (Vue/TypeScript), CI/CD pipelines, database interactions (MongoDB), and build processes (Gradle/Vite).  
- **Excludes:** Experimental libraries, ad-hoc scripts, third-party non-core tools, and legacy/unversioned integrations.  
- **Dependencies:** Requires adherence to [Folder Structure](03-folder-structure.md) for layout and [Modular Architecture](04-modular-architecture.md) for module boundaries.  

---

## 🧠 Semantic Anchors  
1. **Versioning DNA**: Use `@Semantic("LTS")` for long-term stable components (e.g., Java 21+); `@Semantic("Reactive")` for non-blocking patterns.  
2. **Reactive Intent**: Tag interfaces and methods with `@Vibe("ReactiveBoundary")` to signal Flux/Mono usage and prevent blocking calls.  
3. **Stack Taxonomy**:  
   ```mermaid
   graph TD
     Backend[Java 21+ / Spring Boot 3.5 / WebFlux / MongoDB 7.x] -->|Reactive Streams| Integration[Gradle 8.5+ Unified Build]
     Frontend[TypeScript 5.x / Vue 3.4 / Vite 4.4+] -->|Typed Props/Emits| Integration
     Integration -->|Versioned Assets| Static[ /static/vX/ ]
   ```

---

## 🔧 Core Technology Stack  

### Backend  
| **Component**        | **Version**             | **Semantic Tag**        | **Critical Constraints**                                                                          |  
|----------------------|-------------------------|-------------------------|---------------------------------------------------------------------------------------------------|  
| Java (OpenJDK)       | ≥21 (LTS)               | `@Semantic("LTS")`      | Enforce via `java.toolchain.languageVersion = JavaLanguageVersion.of(21)`; no proprietary JVMs.   |  
| Spring Boot          | 3.5.x                   | `@Semantic("Reactive")` | WebFlux only (`spring-boot-starter-webflux`); ban `spring-boot-starter-web` for blocking MVC.     |  
| MongoDB Driver       | 4.10+ (for MongoDB 7.x) | `@Semantic("Eventual")` | Reactive streams exclusively (`Flux<Document>`, `Mono<Document>`); no synchronous queries.        |  

### Frontend  
| **Component**        | **Version** | **Semantic Tag**          | **Critical Constraints**                                                                                    |  
|----------------------|-------------|---------------------------|-------------------------------------------------------------------------------------------------------------|  
| TypeScript           | 5.x         | `@Semantic("Strict")`     | `strict: true` in `tsconfig.json`; typed interfaces for all props/emits (no `any`).                         |  
| Vue                  | 3.4.x       | `@Semantic("Composable")` | Composition API only (`<script setup>`); no legacy Options API; integrate reactive streams via typed props. |  
| Vite                 | 4.4+        | `@Vibe("Bundler")`        | Hot module replacement for dev; optimized bundles for prod with versioned outputs.                          |  

### Infrastructure  
| **Tool**             | **Version** | **Role**              | **Constraints**                                                 |  
|----------------------|-------------|-----------------------|-----------------------------------------------------------------|  
| Gradle               | 8.5+        | Unified build tool    | Kotlin DSL; multi-module setup for Java/TS sequencing.          |  
| BlockHound           | Latest      | Blocking detection    | Integrate in tests to scan for blocking calls in WebFlux paths. |  

---

## ⚙️ Integration & Enforcement Rules  

1. **Reactive Handshake**:  
   - Backend: Expose endpoints as `Mono<T>` or `Flux<T>` (e.g., `@GetMapping("/users/flux")`).  
   - Frontend: Consume via TypeScript interfaces for live updates:  
     ```typescript  
     interface Props {  
       userStream: Flux<UserDto>; // @Vibe("LiveUpdates") – Mirrors backend Flux<UserDto>  
     }  
     ```  
   - Enforcement: Custom Checkstyle rule flags non-reactive suffixes; ESLint for TS naming.  

2. **Static Assets Management**:  
   - Vue builds output to backend's `/src/main/resources/static/vX/` (X = major version, e.g., `v1`).  
   - Reference in HTML: `<script src="/static/v1/main.js" defer></script>`.  
   - Cache busting: Increment `vX` on major UI changes; use Gradle copy tasks for automation.  

3. **Semantic Enforcement in Build**:  
   ```kotlin  
   // build.gradle.kts (root project)  
   tasks.register("validateSemantics") {  
     doLast {  
       val javaVersion = JavaVersion.current()  
       require(javaVersion >= JavaVersion.VERSION_21) {  
         "@Semantic('LTS') violation: Java 21+ required for Spring Boot 3.5."  
       }  
       // Integrate BlockHound for reactive validation  
       exec { commandLine("./gradlew :backend:blockHoundAgent") }  
     }  
   }  
   tasks.named("build").configure { dependsOn("validateSemantics") }  
   ```  
   - CI/CD: Fail PRs if `./gradlew validateSemantics` or `./gradlew verifyStack` fails.  

4. **Additional Enforcement Mechanics**:  
   | **Check**                  | **Tool**                     | **Action**                     |  
   |----------------------------|------------------------------|--------------------------------|  
   | Java/Stack Versions        | Gradle Toolchain + Plugins   | Fail build if < minima         |  
   | Reactive Naming/Patterns   | Checkstyle + ESLint          | Block commits/PRs on violations|  
   | Vue Prop Typing            | TypeScript Compiler (tsc)    | Compilation errors halt build  |  
   | Blocking Calls             | BlockHound Integration       | Fail tests on detection        |  
   | Asset Versioning           | Gradle Copy/ProcessResources | Warn/enforce versioned paths   |  

---

## 🔗 Interlocks  
- **Naming Conventions**: Follow [06-naming-conventions.md](../02-semantics/06-naming-conventions.md) with suffixes like `*Dto`, `*Flux` for reactive elements.  
- **Modular Build**: Integrates with [22-gradle-multimodule.md](../06-build/22-gradle-multimodule.md) for task dependencies (e.g., `compileJava` before `viteBuild`).  
- **Workflow Visualization**: Use Mermaid diagrams per [14-mermaid-diagrams.md](../04-workflow/14-mermaid-diagrams.md) to depict stack flows.  
- **Architecture Boundaries**: DTOs only for cross-module communication, as per [04-modular-architecture.md](04-modular-architecture.md).  

---

## 📋 Compliance Checklist  
- [ ] All components meet or exceed specified versions (run `./gradlew verifyStack`).  
- [ ] Reactive patterns enforced: No blocking calls in controllers (BlockHound clean).  
- [ ] Frontend DTOs mirror backend via `*.d.ts` files with strict typing.  
- [ ] Static assets placed in versioned paths (`/static/vX/`).  
- [ ] Semantic tags (`@Semantic`, `@Vibe`) applied where required.  
- [ ] Build passes: `./gradlew clean build validateSemantics`.  

---

## 📜 Revision History  
| Version | Date       | Author           | Changes Summary                        |  
|---------|------------|------------------|----------------------------------------|  
| 1.0     | 2026-01-21 | LLM-Generated    | Initial specification with core stack. |  

---

## Implementation Notes  
1. **Customization**: Replace all TODOs with project-specific details (e.g., Domain: "E-commerce order processing"; Owner: "@backend-team").  
2. **Validation Workflow**: Integrate into Git hooks or CI: Run `./gradlew validateSemantics` pre-commit; use `@Vibe("AtomicOperation")` for Mongo transactions (e.g., `@Vibe("AtomicOperation - Payment Rollback") public Mono<Void> processPayment(Order order) { ... }`).  
3. **Key Differentiators**:  
   - **Semantic Tagging**: Facilitates AI-assisted code reviews and intent parsing.  
   - **Reactive-First**: Prevents common pitfalls like blocking I/O in WebFlux.  
   - **Unified Pipeline**: Gradle orchestrates Java/TS builds seamlessly, with Vite for frontend optimization.  
4. **Upgrades**: Monitor for LTS releases (e.g., Java 21); update versions and re-run compliance checks. Ban `@ComponentScan` – prefer explicit imports for modularity.  
5. **Branching Guideline**: Use `feat/[domain]-[stack-update]` (e.g., `feat/payments-update-vue-version`) for stack changes, with PRs including updated Mermaid visuals.