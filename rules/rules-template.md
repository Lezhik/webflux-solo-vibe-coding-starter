# Kilo Code Project Rules Template

This template provides a standardized, machine-parsable structure for all documentation files in the `rules/` directory of the Kilo Code project. It emphasizes **vibe coding**—where code and documentation inherently communicate intent through semantic naming, annotations, and structure—while supporting the project's Java/Spring Boot/WebFlux backend and Vue/TypeScript frontend stack. The design promotes consistency, AI-assisted analysis (via semantic tags), and automated enforcement to maintain reactive, modular architecture.

Key principles:
- **Semantic Depth**: Mandatory `@Semantic` and `@Vibe` annotations for intent signaling.
- **Reactive Consistency**: Naming patterns enforce non-blocking flows across stacks.
- **Automation-Ready**: Strict sections enable tools like Gradle tasks or LLM parsers to validate compliance.
- **Modularity**: Cross-references to related rules for discoverability.

Apply this template to new files by duplicating it, injecting domain-specific content, and updating metadata. All files MUST follow the file architecture: `rules/[category]-[purpose]/[numbered-filename].md` (e.g., `rules/01-project/01-overview.md`).

---

## File Architecture Pattern
Organize rules hierarchically for scalability:
```
rules/
├── 01-project/          # High-level overviews and foundations
│   └── 01-project-overview.md
├── 02-semantics/        # Vibe coding, naming, annotations
│   └── 05-vibe-coding.md
├── 03-enforcement/      # Tools, validation, CI/CD
│   └── 22-gradle-multimodule.md
├── 04-stack-specific/   # Java/WebFlux, Vue/TypeScript rules
│   ├── 09-java-style.md
│   └── 10-typescript-style.md
└── 05-workflow/         # Development processes, diagrams
    └── 13-feature-lifecycle.md
```

---

## 1. Mandatory Header Block
Every file starts with this metadata for easy parsing and versioning.

```markdown
# [Rule ID] – [Human-Friendly Title]
<!-- e.g., "01 – Project Overview – Foundation" -->

**File Path:** rules/[category]/[filename].md  
**Domain:** [Business Context, e.g., "E-commerce payments"]  
**Last Updated:** YYYY-MM-DD  
**Status:** Draft | Active | Deprecated  
**Owner:** [Team/Dev Name, e.g., "@architecture-team"]  

<!-- FILE_TITLE: [Brief internal title, e.g., "Vibe-Coding Principles"] -->  
<!-- FILE_PATH: rules/02-semantics/05-vibe-coding.md -->  <!-- For templating -->
```

---

## 2. Purpose & Scope
Define the rule's razor-sharp focus and boundaries.

```markdown
## 🎯 Purpose
[1-2 sentences: Single-sentence objective, e.g., "Defines core project boundaries, initialization constraints, and semantic anchors to ensure vibe-consistent reactive architecture."]

## 📌 Scope
- **Applies to:** [e.g., "All Java services using WebFlux Mono/Flux and Vue components."]
- **Excludes:** [e.g., "Legacy blocking code or third-party libraries."]
- **Dependencies:** [Links to related rules, e.g., "See rules/06-naming-conventions.md for base patterns."]
```

---

## 3. Semantic Anchors & Vibe Coding Mandates
Core "vibe-critical" elements: Naming DNA, annotations, and patterns that propagate intent across the codebase. Focus on reactive flows (e.g., `*Flux`/`*Mono` suffixes) and domain signaling.

```markdown
## 🧠 Semantic Anchors
These elements ensure code reads like documentation, enabling LLM-assisted refactoring and audits.

1. **Naming DNA** (Verb-Domain-Context Pattern):
   - **Why:** Signals execution model (reactive vs. blocking) and reduces ambiguity.
   - **How:**
     ```java
     // Backend: Reactive stream for user events
     @Semantic("EventStream")
     public Flux<UserEvent> streamUserPaymentEventsAsync(String userId) { ... }
     
     // Frontend: Typed Vue prop for the stream
     ```typescript
     // Vue SFC: Composition API with semantic prop
     <script setup lang="ts">
     interface Props {
       eventStream: UserEvent[];  // @Vibe "Live payment updates"
     }
     defineProps<Props>();
     </script>
     ```
   - **Validation:** Fails if missing `*Async` or `*Stream` suffixes (e.g., via ESLint/Gradle).

2. **Annotation Taxonomy** (Mandatory for Cross-Layer Intent):
   - **@Semantic**: Marks role (e.g., "AggregateRoot", "DataflowBoundary").
   - **@Vibe**: Describes "emotional" or contextual intent (e.g., "Ensures eventual consistency").
   - **Example:**
     ```java
     @Vibe("Cross-Module Boundary – Prevents tight coupling")
     @Semantic("PaymentGatewayAdapter")
     public class PaymentAdapter {
         public Mono<PaymentResult> processPaymentFlux(PaymentIntent intent) { ... }
     }
     ```

3. **Code Section Ordering** (For Readability):
   1. Imports (grouped: Core Java/Spring, Project, Third-Party).
   2. Constants (prefixed: `PUBLIC_`, `PRIVATE_`).
   3. Domain Models (DTOs/Entities with `@Semantic`).
   4. Reactive Logic (Flux/Mono pipelines).
   5. Frontend Exports (Vue components with typed props).

## 🧠 Vibe Coding Imperatives
- **Emotional Integrity Test:** Names must convey intent in <3 seconds (e.g., `transformOrderToShipmentEvent()` vs. `handleData()`).
- **Reactive Police:** All async methods expose model (e.g., `fetchUsersFlux()` signals streaming).
- **Cross-Stack Sync:** Backend DTOs mirror frontend props (e.g., `PaymentDto` in Java → `PaymentProps` in TS).

**Anti-Patterns:**
- ❌ `public List<User> getUsers() { ... }` (Ambiguous blocking).
- ✅ `public Flux<User> streamActiveUsersAsync() { ... }` (Clear reactive intent).
```

---

## 4. Enforcement Matrix & Tooling
How to verify and automate compliance. Integrate with CI/CD for zero-tolerance.

```markdown
## ⚙️ Enforcement Mechanics
| Rule Type          | Tool/Mechanism              | Failure Action                  | Configuration Example |
|--------------------|-----------------------------|---------------------------------|-----------------------|
| Naming Convention  | ESLint + Custom Rule (JS/TS)| Block PR / Auto-reformat       | `.eslintrc.js`: `{ rules: { 'reactive-naming': 'error' } }` |
| Annotation Check   | Gradle Checkstyle           | Fail Build Pipeline            | `build.gradle.kts`: `tasks.register("validateSemantics") { ... }` |
| Reactive Structure | SonarQube + IDE Formatter   | Warn on Commit / Auto-fix      | `sonar-project.properties`: `sonar.issue.ignore.multicriteria=e1` |
| Vue Prop Typing    | TypeScript Compiler         | Type Errors Block Compilation  | `tsconfig.json`: `strict: true` |
| Diagram Validation | Mermaid CLI (in Git Hooks)  | Reject Invalid Syntax          | `pre-commit: mermaid validate diagram.md` |

**CI/CD Integration:**
- **Gradle Task Example** (for multimodule builds):
  ```kotlin
  // build.gradle.kts (root)
  tasks.register("verifyVibe") {
      dependsOn(":backend:semanticLint", ":frontend:eslint")
      doLast {
          // Custom script: Scan for @Semantic tags and naming patterns
          exec { commandLine("node", "scripts/validate-vibe.js") }
      }
  }
  ```
- **GitHub Actions Workflow:** Run `verifyVibe` on PRs; fail if vibe-score (custom metric) < 90%.
- **IDE Support:** IntelliJ inspections for `@Vibe` JSDoc; VS Code extensions for Vue semantic highlighting.
```

---

## 5. Technical Specifications & Examples
Stack-specific rules with live code, diagrams, and anti-patterns.

```markdown
## 🔧 Technical Specifications
### Java/WebFlux Rules (e.g., for 09-java-style.md)
- **Stream Identification:** Suffix with `Flux`/`Mono`; prefix verbs like `stream*`, `publish*`.
- **Layer Annotations:** Tag boundaries with `@Vibe`.
- **Example:**
  ```java
  @Deprecated(since = "v1.2", forRemoval = true, because = "Blocking violates reactive contract")
  public List<Order> fetchOrders() { ... }  // Legacy – Avoid

  // Current (v1.3+)
  @Semantic("Dataflow")
  @Vibe("Tracks order journey with rollback on failure")
  public Flux<OrderEvent> publishOrderStreamAsync(OrderId id) {
      return orderRepo.findById(id).flatMapMany(this::processEvents);
  }
  ```

### Vue/TypeScript Rules (e.g., for 10-typescript-style.md)
- **Composition API Mandates:** Use `<script setup lang="ts">`; enforce prop types.
- **Example:**
  ```vue
  <template>
    <div v-for="event in eventStream" :key="event.id">
      {{ event.status }}  <!-- Semantic prop drives UI reactivity -->
    </div>
  </template>

  <script setup lang="ts">
  import { defineProps } from 'vue';
  
  interface Props {
    eventStream: OrderEvent[];  // Mirrors Java Flux emission
  }
  
  /**
   * @Semantic("DataVisualizer")
   * @Vibe "Real-time order updates without polling"
   */
  const props = defineProps<Props>();
  </script>
  ```

### Workflow Integration (e.g., for 13-feature-lifecycle.md)
- **Vibe-Centric Flow:** Blueprint with Mermaid before coding.
- **Diagram Example:**
  ```mermaid
  graph TD
      A[User Action: Place Order] --> B[API: publishOrderStreamAsync]
      B --> C[Vue: eventStream Prop]
      C --> D{{Display Live Status}}
      style B fill:#ff9,stroke:#333  %% Semantic highlight for reactive boundary
  ```
- **Code Generation Step:** 
  ```
  /features/order-tracking/
  ├── semantic-diagram.md      # Mermaid blueprint
  ├── dto/OrderEvent.java      # Backend model
  └── components/Tracker.vue   # Frontend viz (typed props)
  ```

**Versioned Examples:**
- **v0.9 (Initial):** Basic naming without annotations.
- **v1.2+ (Current):** Full `@Semantic` + reactive enforcement.
```

---

## 6. Compliance Toolkit & Validation Checklist
Self-audit tools for developers.

```markdown
## 📋 Compliance Checklist
- [ ] All reactive methods use `Flux`/`Mono` suffixes and `@Semantic` tags.
- [ ] Vue props are typed with domain-specific interfaces (no `any`).
- [ ] Names pass 3-second intent test (e.g., `validatePaymentIntentAsync()`).
- [ ] Cross-references to related rules are included.
- [ ] No inline styles/scripts in Vue SFCs (use scoped CSS).
- [ ] Gradle task `verifyVibe` passes locally.

**Resource Placement Rules:**
- Static assets: `/src/main/resources/static/v[version]/` with cache-busting.
- Gradle Snippet for Versioning:
  ```kotlin
  processResources {
      filesMatching("**/static/**") { expand(projectVersion: project.version) }
  }
  ```
```

---

## 7. Cross-Stack Sync & Interlocks
Ensure harmony between backend/frontend.

```markdown
## 🔗 Interlocks with Other Rules
- **Naming:** Aligns with `06-naming-conventions.md` (DTO suffixes like `*Dto`).
- **Build Sequencing:** Depends on `22-gradle-multimodule.md` (e.g., `compileJava` before `compileTypeScript`).
- **Diagrams:** Follow `14-mermaid-diagrams.md` syntax for vibe visualization.
- **Reviews:** Mandatory in `16-review-checklist.md` (e.g., "Verify @Vibe tags").

**Key Innovation:** `@dataflow` tags track ownership (e.g., `@dataflow(backend-to-frontend)` for stream handoffs), enabling automated dependency graphs.
```

---

## 8. Revision History
Track evolution for auditability.

```markdown
## 📜 Revision History
| Version | Date       | Author          | Changes Summary                          | Impact Scope     |
|---------|------------|-----------------|------------------------------------------|------------------|
| 0.1     | 2024-03-15 | @initial-dev    | Initial template with basic structure    | Project-wide     |
| 1.0     | 2024-01-20 | @architecture  | Added vibe coding and enforcement tables | Semantics + Tools|
| 1.2     | 2024-02-10 | @team-lead     | Integrated stack-specific examples; CI hooks | Java/Vue Sync   |
| 1.3     | 2024-03-01 | @vibe-dev      | Emotional integrity checks; Mermaid support | Workflow         |

**Changelog Notes:** v1.3 introduces `@dataflow` for better LLM parsing of cross-module streams.
```

---

## Implementation Guide
1. **Clone & Customize:** `cp TEMPLATE.md rules/[category]/[new-file].md`, then fill sections.
2. **Inject Domain Logic:** Adapt examples to contexts like "payment processing" (e.g., atomic ops with rollback).
3. **Validate Locally:** Run `./gradlew verifyVibe` and check the compliance list.
4. **Branching:** Use `feat/[semantic-verb]-[domain]` (e.g., `feat/stream-payment-events`); attach Mermaid per workflow rules.
5. **Static Assets:** Versioned outputs (e.g., Vue builds to `/static/v1.3/`).

**Differentiators from Standard Templates:**
- **Vibe Propagation:** Semantic tags create AI-friendly anchors beyond types.
- **Reactive Enforcement:** Unified patterns prevent blocking pitfalls.
- **Parsability:** Headers/tables enable scripts to extract/validate rules automatically.
- **Holistic Coverage:** Combines semantics, tooling, and workflows for end-to-end consistency.

This template optimizes for **maintainability**, **scalability**, and **vibe integrity** in Kilo Code's reactive ecosystem. For questions, reference the owner or raise an issue in the repo.