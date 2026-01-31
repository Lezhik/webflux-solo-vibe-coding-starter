# 07 - Semantic Comments - Intent-Driven Documentation and Vibe Storytelling

**File Path:** rules/02-semantics/07-semantic-comments.md  
**Domain:** TODO: <business-context> (e.g., "Secure e-commerce payment processing with real-time event streaming")  
**Last Updated:** 2026-01-22  
**Status:** Active  

<!-- FILE_TITLE: Semantic Comments – Intent-Driven Documentation and Vibe Storytelling -->  
<!-- FILE_PATH: rules/02-semantics/07-semantic-comments.md -->  

---

## 🎯 Purpose
Establish machine-parsable comment standards that document technical and business rationale, encode semantic intent, and enable LLM-assisted analysis, automated tooling, and cross-stack consistency. Comments evolve from static notes to dynamic artifacts that convey *why* code exists, using structured tags for "vibe storytelling" – capturing emotional, operational, and contextual nuances in reactive systems.

## 📌 Scope
- **Applies to:** All Java (Javadoc), TypeScript (JSDoc), and Vue (script blocks) source files, focusing on non-trivial logic like reactive flows and domain boundaries.
- **Excludes:** Auto-generated code, trivial utilities (e.g., simple getters/setters), configuration files, and third-party dependencies.
- **Dependencies:** [05-vibe-coding.md](./05-vibe-coding.md) for naming/vibe principles, [06-naming-conventions.md](./06-naming-conventions.md) for suffixes, [12-annotations.md](../03-coding-standards/12-annotations.md) for annotation extensions.

---

## 🧠 Semantic Anchors

### 1. Comment Taxonomy and Mandatory Tags
Use these tags to make comments human-readable and machine-parsable. All complex methods (e.g., those returning `Flux`/`Mono` or handling streams) require at least `@Semantic` and `// Why:`.

| Tag              | Scope          | Purpose                                                                     | Enforcement Level             | Example Usage                                    |
|------------------|----------------|-----------------------------------------------------------------------------|-------------------------------|--------------------------------------------------|
| `@Semantic`      | Cross-stack    | Defines the code's domain role (e.g., "PaymentBoundary", "DataRenderer").   | Required for public methods   | `@Semantic("PaymentGatewayAdapter")`             |
| `@Vibe`          | Cross-stack    | Conveys contextual/emotional intent (e.g., "Ensures eventual consistency"). | Recommended for reactive code | `@Vibe("Error-resilient but latency-sensitive")` |
| `@Constraint`    | Method/Class   | Documents hard limits (e.g., timeouts, rate limits).                        | Required for boundaries       | `@Constraint: Retries backoff up to 30s`         |
| `@Flow`          | Reactive paths | Maps execution flows (e.g., "read → validate → process → event").           | Required for streams          | `@Flow: (kafka→ui via SSE)`                      |
| `@EdgeCase`      | Inline/Method  | Highlights exceptions (e.g., "Handles null sessions").                      | Recommended                   | `@EdgeCase: Redirects to login on null`          |
| `@Guard`         | TypeScript/Vue | Signals defensive logic (e.g., "Prevents race conditions").                 | Conditional (validation)      | `@Guard: Validates props before render`          |

### 2. Structured Comment Format
Follow this ordered template for clarity and parsability. Limit "FUNCTIONAL INTENT" or summaries to ≤120 characters.

**Java (Javadoc) Example:**
```java
/**
 * @Semantic("PaymentBoundary")  // Domain role
 * @Vibe("Ensures eventual consistency across processors")
 * @Constraint: Transaction <2s timeout
 * @Flow: (read → validate → process → save → event)
 *
 * FUNCTIONAL INTENT: Streams payment events with failure isolation
 * @param batch PaymentBatch input
 * @return Flux<PaymentResult>
 *
 * // Why: Avoids 3rd-party API rate limits via staggered retries (complies with policy XYZ)
 * // Why: Aligns backend with frontend stream visualization
 * // Caveat: Exponential backoff; requires SSE polyfill for legacy browsers
 * // EdgeCase: Skips invalid records to prevent full batch failure
 */
public Flux<PaymentResult> processPaymentsFlux(PaymentBatch batch) {
    return paymentProcessor.transform(batch)
        .retryWhen(Retry.backoff(3, Duration.ofSeconds(1)))
        .onErrorResume(e -> logAndSkip(e));
}
```

**TypeScript/Vue (JSDoc) Equivalent:**
```typescript
/**
 * @Semantic("DataVisualizer")
 * @Vibe("Real-time stream without polling")
 * @Guard "Client-side filtering for high-volume data"
 * @Flow: (backend EventStream → UI render)
 *
 * FUNCTIONAL INTENT: Visualizes payment status changes in <200ms
 * @param events PaymentEvent[] - Stream from backend
 * @returns void
 *
 * // Why: Reduces backend load by 40% vs. polling; syncs with RxJS timestamps
 * // Why: Improves UX with live updates during checkout
 * // Caveat: Degrades >1k items; use virtual scrolling
 * // EdgeCase: Handles connection loss with auto-reconnect
 */
function renderPaymentStream(events: PaymentEvent[]): void {
    // Implementation...
}
```

**Vue Component Script Example:**
```vue
<script setup lang="ts">
/**
 * @Semantic("DataVisualizer")
 * @Vibe("Client-side filtering for streams")
 * @Constraint: Compatible with Chrome/Firefox only
 *
 * FUNCTIONAL INTENT: Renders real-time payment events
 *
 * // Why: Offloads common filters from backend
 * // Why: Matches backend DTO types for consistency
 * // Caveat: Initial load pulls full dataset
 */
interface Props {
  events: PaymentEvent[];  // @DataFlow: backend→frontend
}
const props = defineProps<Props>();
const filterResults = (criteria: Filter) => { /* ... */ };
</script>
```

### 3. Inline Comment Standards
For fields or simple logic, use concise `// Why:` or `// Caveat:` prefixes.
```java
// Why: MongoDB TTL index auto-expires after 30d (data retention policy XYZ)
// Caveat: Non-atomic; use transactions for critical paths
@Indexed(expireAfterSeconds = 2_592_000)
private Instant createdAt;
```

### 4. Vibe Storytelling Guidelines
- Convey intent in <3 seconds: Focus on "why this exists" (business impact) and "how it feels" (e.g., "latency-sensitive").
- Avoid ambiguity: Bad – "// Updates status"; Good – "// Why: Prevents blocking in 10k+ concurrent sessions".
- Progressive depth: Trivial code needs none; reactive/complex needs full taxonomy.

---

## ⚙️ Enforcement Mechanics

| Rule Type            | Tool/Mechanism                  | Failure Action             | Configuration Example                                           |
|----------------------|---------------------------------|----------------------------|-----------------------------------------------------------------|
| Tag Presence         | Checkstyle (Java), ESLint (TS)  | Block PR merge             | `.eslintrc.js: { 'semantic-comments': 'error' }`                |
| `// Why:` Coverage   | Custom Gradle Semantic Scan     | Fail Build / Warn in CI    | `build.gradle.kts: tasks.register("validateComments") { ... }`  |
| Vibe/Intent Clarity  | SonarQube + IDE Linter          | Auto-reformat / Type Errors| `tsconfig.json: { strict: true }`                               |
| Reactive Structure   | TypeScript Compiler + Checkstyle| Block Compilation          | `sonar-project.properties: sonar.issue.ignore.multicriteria=e1` |

**CI/CD Integration (Gradle Example):**
```kotlin
tasks.register("validateComments") {
    dependsOn("checkstyleMain", "eslint")
    inputs.files(layout.files("**/*.java", "**/*.ts", "**/*.vue"))
    doLast {
        exec { commandLine("node", "scripts/semantic_audit.js", "python", "scripts/semantic_audit.py") }
    }
}

tasks.named("check") {
    dependsOn("validateComments")
}
```

**IDE Support:**
- IntelliJ/VS Code: Live templates/snippets (e.g., `kcmt` expands to full structure).
- Auto-validation: Link tags to quick-help docs referencing this file.

---

## 🔧 Technical Specifications

### Java Reactive Rules (WebFlux Focus)
- Mandatory for `Flux`/`Mono`: Include `@Flow` and `@Constraint`.
- Avoid blocking: Comments must flag non-reactive pitfalls.
```java
// ❌ Ambiguous
List<User> getUsersBlocking() { ... }

/**
 * @Semantic("UserStream")
 * @Vibe("Permission-sensitive non-blocking stream")
 * @Flow: (ACL filter → cache → DB query)
 *
 * FUNCTIONAL INTENT: Reactive users with ACL for dashboards
 * // Why: Handles 10k+ sessions without backpressure
 * // Caveat: Retry max 3x via reactor-extra
 */
public Flux<User> streamFilteredUsersAsync() {
    return userRepo.findAll().filter(acl::permit);
}
```

### TypeScript/Vue Rules (Composition API)
- Typed props/interfaces: Require `@Guard` for validation.
- Streams: Align with backend (e.g., WebSocket/SSE).
```vue
<script setup lang="ts">
/**
 * @Semantic("PaymentEventHandler")
 * @Vibe("UI sync with backend events")
 * @Guard "Validates event timestamps"
 *
 * FUNCTIONAL INTENT: Handles real-time payment updates
 * // Why: Syncs RxJS with EventStream API
 * // EdgeCase: Polyfill SSE for IE
 */
const { events } = defineProps<Props & { events: PaymentEvent[] }>();
</script>
```

---

## 📋 Compliance Checklist
- [ ] Non-trivial methods include `@Semantic`, `@Vibe`, and `// Why:` (≤120 chars for intent).
- [ ] Reactive flows use `@Flow` and pass 3-second readability test.
- [ ] Cross-stack tags match (e.g., backend `@Semantic` ↔ frontend props).
- [ ] Inline comments cover constraints/edges; no unused/orphaned notes.
- [ ] Enforcement tasks (`validateComments`) run clean locally (<5% warnings).
- [ ] Examples adapted to domain (e.g., payment atomicity with rollbacks).

---

## 🔗 Interlocks
- **Naming:** Aligns with [06-naming-conventions.md](./06-naming-conventions.md) (e.g., `*Async` suffixes for reactive).
- **Annotations:** Extends [12-annotations.md](../03-coding-standards/12-annotations.md) with semantic tags.
- **Workflow:** Links to [14-mermaid-diagrams.md](../04-workflow/14-mermaid-diagrams.md) for flow visuals; [16-review-checklist.md](../04-workflow/16-review-checklist.md) mandates tag verification.
- **Testing:** `// Why:` ties to boundary scenarios in [21-boundary-scenarios.md](../05-testing/21-boundary-scenarios.md).
- **Build:** Integrates with [22-gradle-multimodule.md](../04-workflow/22-gradle-multimodule.md) sequencing.

---

## 📜 Revision History
| Version | Date       | Author           | Changes Summary                                                               |
|---------|------------|------------------|-------------------------------------------------------------------------------|
| 1.0     | 2026-01-22 | LLM-Generated    | Combined variants: Added taxonomy table, unified format, enhanced enforcement |

---

## Implementation Guide
1. **Template Setup:** Use IDE snippets or `kilocode new-comment --type java-reactive` to generate boilerplate.
2. **Customization:** Fill TODOs (e.g., domain specifics like "atomic rollbacks for Stripe").
3. **Validation:** Run `./gradlew validateComments verifySemantics` locally; fix via auto-reformat.
4. **Branching/PRs:** Use `feat/semantic-[domain]` (e.g., `feat/semantic-payment-stream`); include Mermaid flows.
5. **Progressive Rollout:** Start with warnings in CI, escalate to blocks; audit existing code quarterly.
6. **Differentiators:** Enables AI extraction (e.g., scripts parse `@Vibe` for reports); promotes reactive best practices beyond types.

**Quick Start Command:**
```bash
# Generate and validate
cp templates/semantic-comment.md rules/02-semantics/07-semantic-comments.md
./gradlew validateComments
```