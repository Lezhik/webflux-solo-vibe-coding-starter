# 15 – Feature Repository Index – Structure & Guidelines

**File Path:** rules/04-workflow/15-feature-repo-index.md  
**Domain:** Payment Fraud Detection  
**Last Updated:** 2026-01-26  
**Status:** Active  

---

## 🎯 Purpose  
This guideline standardizes the organization, naming, versioning, and lifecycle of feature repositories under the `/features/` directory. It promotes atomic, isolated development using LLM-assisted planning and Kilo Code automation, ensuring semantic discoverability, reactive consistency across Java/WebFlux backend and Vue/TypeScript frontend layers, and seamless integration with task tracking. This approach supports mini-sprints, immutable versioning, and automated compliance to maintain scalability in the Payment Fraud Detection domain.

---

## 📌 Scope  
- **Applies to:** All new features developed under `/features/`.  
- **Excludes:** Legacy modules, infrastructure code, third-party integrations, or non-feature directories (e.g., core utilities).  
- **Dependencies:** [06-naming-conventions.md](../02-semantics/06-naming-conventions.md), [13-feature-lifecycle.md](../04-workflow/13-feature-lifecycle.md), [14-mermaid-diagrams.md](../04-workflow/14-mermaid-diagrams.md), [20-test-data-profiles.md](../05-test-strategy/20-test-data-profiles.md), [22-gradle-multimodule.md](../06-build/22-gradle-multimodule.md), [27-dependency-rules.md](../07-modularity/27-dependency-rules.md).  

---

## 🧠 Semantic Architecture  
### Feature Development DNA  
1. **LLM-Assisted Blueprinting**  
   - **Input:** Feature description + current schema (e.g., JSON model of payment events).  
   - **Output:** Mermaid diagrams with `@Semantic` boundaries, initial DTOs, and TODO markers for gaps (e.g., `TODO: Integrate third-party fraud API`).  
   - **Validation:** Ensures all outputs align with vibe-coding principles for emotional and functional intent.  

2. **Kilo Code Execution Flow**  
   ```mermaid
   graph TD
       A[LLM Task Definition & Setup] --> B[Model/DTO Generation]
       B --> C[Test Data Population (YAML Profiles)]
       C --> D[API Stubs & Components (Reactive Stubs)]
       D --> E[Test Suite Implementation (Testcontainers)]
       E --> F[Business Logic Completion (WebFlux/Vue)]
       F --> G[PR Merge & Task Archive to completed.md]
   ```  
   - **Reactive Focus:** All backend methods return `Mono` or `Flux` (e.g., `streamFraudAlertsFlux()`). Frontend props sync via TypeScript interfaces for live updates.  

3. **Task Repository Integration**  
   - **Structure:**  
     ```
     /tasks/
     └── domains/
         └── payment-fraud-detection/
             ├── incomplete.md  # Priority-ordered (High/Med/Low) with feature links
             └── completed.md   # Auto-archived on PR merge via CI hooks
     └── global/
         └── sprint-backlog.md  # Cross-domain overview
     ```  
   - **Workflow:** Features start in `incomplete.md`. Merge triggers migration to `completed.md`, providing an audit trail for fraud detection evolutions.  

### Vibe-Coding Imperatives  
- **Naming Pattern:** `[verb]-[domain]-[action]` in kebab-case (e.g., `detect-fraud-patterns` or `flag-high-risk-payments`). Matches Git branches (e.g., `feat/detect-fraud-patterns`).  
- **Annotations:** `@Semantic` for boundaries (e.g., `@Semantic("FraudStream")` in code/diagrams); `@Vibe` for intent (e.g., `@Vibe "Sub-second alerts for zero-tolerance fraud"` in README.md).  
- **Anti-Patterns:** Avoid blocking I/O in services, inline Vue styles/scripts, or cross-feature dependencies.  

---

## 🔄 Feature Creation Cycle (Mini-Sprint Workflow)  
This cycle integrates LLM planning with Kilo Code execution, ensuring atomic progress from idea to deployment.  

1. **LLM Planning Phase (Task Setup)**  
   - **Input:** Feature description (e.g., "Real-time fraud pattern detection") + schema (e.g., `{"userId": "string", "amount": "double", "ipCountry": "string"}`).  
   - **Output:**  
     - `semantic-diagram.md` with Mermaid and `@Semantic` tags.  
     - Initial task entry in `/tasks/domains/payment-fraud-detection/incomplete.md` (e.g., `## High Priority\n- [ ] Detect Fraud Patterns ➔ /features/detect-fraud-patterns (@Vibe: Instant risk assessment)`).  
     - TODOs for missing data (e.g., `TODO: Define FraudScore thresholds`).  
   - **Script Example:**  
     ```bash
     ./scripts/llm-generate-task.sh \
       --input schema-v2.json \
       --description "Real-time fraud pattern detection" \
       --domain "payment-fraud-detection"
     ```  

2. **Query Formation for Kilo Code (LLM Agent)**  
   - **Prompt Structure:**  
     ```markdown
     ### Request: Detect High-Risk Payments  
     **Scope:** Real-time scoring of payment attempts.  
     **Input Schema:** {"userId": "string", "amount": "double", "ipCountry": "string"}  
     **Output Requirements:**  
     - Reactive service: Mono<FraudScore> for scoring.  
     - Vue component: Risk heatmap with streamed props.  
     **@Vibe:** "Proactive fraud prevention with live updates."  
     ```  

3. **Kilo Code Phases**  
   | Phase                    | Location                                            | Artifacts Generated                          | Key Requirements                                                            |  
   |---------------------------|----------------------------------------------------|----------------------------------------------|-----------------------------------------------------------------------------|  
   | Models/DTOs               | `/features/[slug]/v1.0/dto/`                       | FraudScoreDto.java, FraudScoreDto.ts         | `@Semantic("RiskModel")` annotations; Mirror Java/TS.                       |  
   | Test Data Population      | `/features/[slug]/v1.0/tests/test-data-profiles/`  | high_risk_payment.yaml                       | `@Vibe("Simulate threshold breaches")`; YAML profiles.                      |  
   | API Stubs Creation        | `/features/[slug]/v1.0/services/` & `/components/` | FraudDetectionService.kt, RiskHeatmap.vue    | Reactive signatures (e.g., `Mono<FraudScore>`); `<script setup lang="ts">`. |  
   | Test Suite Implementation | `/features/[slug]/v1.0/tests/`                     | FraudDetectionTest.kt                        | Testcontainers for DB isolation.                                            |  
   | Business Logic Completion | Same as above                                      | Full implementations                         | Resolve all TODOs; Enforce reactive flows.                                  |  

   - **Reactive Example (Service):**  
     ```kotlin
     @Semantic("RiskCalculator")
     fun calculateRiskAsync(attempt: PaymentAttempt): Mono<FraudScore> {
         return riskRepo.findPatterns(attempt.ipCountry)
                        .flatMap { applyRules(it, attempt) }
     }
     ```  

   - **Reactive Example (Vue):**  
     ```typescript
     interface Props {
       riskData: FraudScore[];  // @Vibe "Live risk stream"
     }
     const props = defineProps<Props>();
     ```  

4. **Task Repository Update & Merge**  
   - PR merge auto-archives task to `completed.md` via CI.  
   - **Script Example:**  
     ```bash
     ./gradlew archiveCompletedTask  # Invokes Node.js script to move tasks
     ```  

5. **Validation & Closeout**  
   - Run local checks: `./gradlew verifyFeatureFlow verifySlugFormat verifyVibe`.  
   - Ensure `vX.Y/` is immutable post-merge; increment for iterations (e.g., v1.1 for refinements).  

---

## 🔧 Technical Specifications  
### Versioned Feature Structure  
```
/features/
└── detect-fraud-patterns/          # Kebab-case slug
    ├── v1.0/                       # Semantic versioning; immutable post-merge
    │   ├── semantic-diagram.md     # Mermaid with @Semantic tags
    │   │   <!-- Example Content -->
    │   │   <!-- Version: v1.0 -->
    │   │   <!-- Domain: Payment Fraud Detection -->
    │   │   ```mermaid
    │   │   graph TD
    │   │       A[Payment Event Input] -->|@Semantic("FraudStream")| B[FraudService: processFlux()]
    │   │       B --> C[Vue: Risk Alert Display]
    │   │   ```
    │   ├── dto/                     # Cross-stack models
    │   │   ├── FraudAlertDto.java
    │   │   └── FraudAlertDto.ts
    │   ├── components/              # Vue SFCs
    │   │   └── FraudDashboard.vue
    │   ├── services/                # WebFlux reactors
    │   │   └── FraudPatternService.kt
    │   ├── tests/                   # Testcontainers + YAML
    │   │   └── test-data-profiles/
    │   │       └── fraud_pattern.yaml
    │   └── README.md                # @Vibe, sprint ticket, goals
    │       # Example: @Vibe "Zero-latency fraud blocking for secure payments."
    └── v1.1/                       # For evolutions (e.g., ML integration)
```  

### README.md Template Excerpt  
```markdown
# Feature: Detect Fraud Patterns

**Sprint Ticket:** JIRA-1234  
**Business Goal:** Reduce false positives in fraud detection by 25% via real-time streams.  
**@Vibe:** "Intelligent, adaptive guarding against evolving threats."  
**Version:** v1.0  
**Dependencies:** None (isolated).  
```  

---

## ⚙️ Enforcement Matrix  
| Rule                     | Tool/Mechanism                  | Action on Failure                   |  
|--------------------------|---------------------------------|-------------------------------------|  
| Slug & Naming Format     | Gradle `verifySlugFormat` task  | Fail build; Suggest auto-reformat   |  
| Versioned Structure      | CI Directory Scanner            | Block PR; Enforce vX.Y folders      |  
| @Semantic/@Vibe Presence | Custom Node script + Checkstyle | Reject commits; Auto-insert TODOs   |  
| Reactive Signatures      | ESLint + TS Compiler            | Fail pipeline on blocking code      |  
| TODO Resolution          | Git Pre-commit Hook             | Block commits with unresolved TODOs |  
| Task Tracking Linkage    | Gradle `verifyFeatureFlow`      | Fail if no entry in incomplete.md   |  

**Gradle Enforcement Snippet (build.gradle.kts):**  
```kotlin
tasks.register("verifyFeatureFlow") {
    dependsOn("verifySlugFormat", "verifyVibe")
    doLast {
        val featureDir = File("features/detect-fraud-patterns/v1.0")
        require(featureDir.exists()) { "Missing versioned directory" }
        
        val taskFile = File("tasks/domains/payment-fraud-detection/incomplete.md")
        require(taskFile.readText().contains("detect-fraud-patterns")) { "Missing task entry" }
        
        val todoPattern = Regex("TODO:")
        fileTree(featureDir).forEach { file ->
            if (todoPattern.containsMatchIn(file.readText())) {
                throw GradleException("Unresolved TODO in ${file.path}")
            }
        }
    }
}

tasks.register("archiveCompletedTask") {
    doLast {
        exec { commandLine("node", "scripts/move-task.js", "detect-fraud-patterns") }
    }
}
```  

**CI Pipeline Example (.github/workflows/features.yml):**  
```yaml
name: Feature Validation
on: [pull_request]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: ./gradlew verifyFeatureFlow
      - name: Archive Task on Merge
        if: github.event.pull_request.merged == true
        run: ./gradlew archiveCompletedTask
        env:
          FEATURE: detect-fraud-patterns
```  

---

## 📋 Compliance Checklist  
- [ ] Feature slug: `[verb]-[domain]-[action]` kebab-case (e.g., `detect-fraud-patterns`).  
- [ ] Git branch matches slug (e.g., `feat/detect-fraud-patterns`).  
- [ ] `vX.Y/` directory created with semantic versioning.  
- [ ] `semantic-diagram.md`: Contains Mermaid, `@Semantic` tags, version/domain header.  
- [ ] DTOs: Annotated `@Semantic`; Java/TS mirrors in `/dto/`.  
- [ ] Components: Vue SFCs with `<script setup lang="ts">`; No inline styles/scripts.  
- [ ] Services: Reactive `Mono/Flux` returns with `*Async` or `*Flux` suffixes.  
- [ ] Tests: YAML profiles + Testcontainers; `@Vibe` descriptors.  
- [ ] README.md: Sprint ticket, business goal, `@Vibe`; All TODOs resolved.  
- [ ] Task: Entry in `incomplete.md`; No cross-feature deps (run `dependencyCheck`).  
- [ ] Enforcement: Passes all Gradle verifies; PR triggers task archive.  

---

## 🔗 Interlocks  
- **Naming & Semantics:** [06-naming-conventions.md](../02-semantics/06-naming-conventions.md) for DTOs/responses.  
- **Lifecycle & Tasks:** [13-feature-lifecycle.md](../04-workflow/13-feature-lifecycle.md) for merge processes.  
- **Diagrams:** [14-mermaid-diagrams.md](../04-workflow/14-mermaid-diagrams.md) for syntax.  
- **Testing:** [20-test-data-profiles.md](../05-test-data-profiles.md) for YAML conventions.  
- **Build & Modularity:** [22-gradle-multimodule.md](../06-build/22-gradle-multimodule.md), [27-dependency-rules.md](../07-modularity/27-dependency-rules.md).  
- **Reviews:** [16-review-checklist.md](../04-workflow/16-review-checklist.md) for PR gates.  

---

## 📜 Revision History  
| Version | Date       | Author          | Changes Summary                                                                 |  
|---------|------------|-----------------|---------------------------------------------------------------------------------|  
| 1.0     | 2026-01-26 | LLM-Generated   | Initial structure with semantic anchors and basic enforcement.                  |  

---

## Implementation Guide  
1. **Initialize:** `git checkout -b feat/[slug]`; Create structure:  
   ```bash
   mkdir -p features/[slug]/v1.0/{dto,components,services,tests} && \
   touch features/[slug]/v1.0/{semantic-diagram.md,README.md}
   ```  

2. **LLM Planning:** Run `./scripts/llm-generate-task.sh` to scaffold task and diagram.  

3. **Kilo Code Automation:**  
   ```bash
   ./gradlew generateModels writeTests implementLogic  # Outputs to v1.0/
   ```  
   - Use curl for LLM handoff if needed:  
     ```bash
     curl -X POST https://llm-service/generate-kilo-prompt \
       -H "Content-Type: application/json" \
       -d '{"feature": "[slug]", "schema": "schema-v2.json"}'
     ```  

4. **Develop & Validate:** Implement reactively; `./gradlew verifyFeatureFlow`. Fill TODOs (e.g., thresholds for fraud scoring).  

5. **PR & Deploy:** Submit PR; Merge auto-archives task. Static assets build to `/static/v1.0/`.  

**Key Innovations:**  
- **LLM-Kilo Pipeline:** End-to-end automation from prompt to code, reducing manual overhead.  
- **Immutable Versioning:** Enables safe experimentation (e.g., v2.0 for ML fraud models).  
- **Task Archaeology:** Provides traceable history for compliance in fraud detection.  
- **Reactive Safeguards:** Prevents performance bottlenecks in high-volume payment streams.  

Replace all TODOs before PR. This ensures vibe-aligned, scalable features in the Kilo Code ecosystem.