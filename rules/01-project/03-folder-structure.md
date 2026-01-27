# 03 – Folder Structure – Monorepo Layout, Task-Driven Features & Lifecycle Automation

**File Path:** rules/01-project/03-folder-structure.md  
**Domain:** Payment Fraud Detection  
**Last Updated:** 2026-01-22  
**Status:** Active  

---

## 🎯 Purpose  
This document defines the mandatory monorepo organization for the Kilo Code project, integrating Java/WebFlux backend, Vue frontend, shared modules, and a task-driven development workflow. It emphasizes domain-driven feature lifecycle management, atomic versioning of immutable features, LLM-assisted blueprinting, and automated task promotion through CI/CD pipelines. The structure ensures traceability from prioritized tasks to deployed artifacts, preserving reactive boundaries, semantic annotations, and vibe-aligned development (e.g., zero-latency fraud blocking).

---

## 📌 Scope  
- **Applies to:** Core directories, task repositories, versioned features, static resources, and cross-stack contracts (DTOs, tests).  
- **Excludes:** Ephemeral files (e.g., IDE configs like `.idea/` or temporary build outputs).  
- **Dependencies:**  
  - [04-modular-architecture.md](04-modular-architecture.md) for domain modules.  
  - [22-gradle-multimodule.md](../06-build/22-gradle-multimodule.md) for multi-module builds.  
  - [05-feature-lifecycle.md](../05-workflow/05-feature-lifecycle.md) for workflow integration.  
  - [06-naming-conventions.md](../02-semantics/06-naming-conventions.md) for task/feature naming.  
  - [10-shared-models.md](../04-stack/10-shared-models.md) for shared DTO alignment.  
  - [30-ci-cd.md](../03-enforcement/30-ci-cd.md) for automation hooks.  

---

## 🧠 Semantic Anchors & Feature Lifecycle  
The lifecycle integrates LLM-driven automation to streamline solo or team development:  

1. **Task Creation & Prioritization**  
   - Define tasks in `tasks/domains/{domain}/incomplete.md` using priority labels (High/Medium/Low).  
   - Include metadata like ID, title, schemas, constraints, and feature links.  
   - LLM Prompt Template: "Generate a semantic blueprint for [Title] in [Domain], focusing on [Input Schema] with @Semantic tags for reactive flows."  

2. **LLM Blueprinting**  
   - Generate `features/{kebab-slug}/v{version}/semantic-diagram.md` with Mermaid diagrams and @Semantic/@Vibe tags.  
   - Auto-create mirrored DTOs in `dto/` (Java/TS) and test profiles in `tests/test-data-profiles/`.  

3. **Implementation Wave (TDD)**  
   - Stubs → Failing Tests → Passing Logic → Vue Components.  
   - Ensure cross-stack consistency (e.g., WebFlux reactors in backend, SFC in frontend).  

4. **Completion & Immutability**  
   - PR Merge Triggers:  
     - Move task to `completed.md`.  
     - Lock feature version (no edits; evolve to v1.1).  
     - Update `global/sprint-backlog.md` for cross-domain tracking.  

**Vibe Mandate:** Every artifact must include @Vibe annotations (e.g., "@Vibe 'Instant fraud blocking with sub-second latency'") to align with project ethos.  

**Anti-Patterns:**  
❌ Editing merged versions (branch for new versions).  
❌ Unversioned or unlinked tasks.  
❌ Missing @Semantic tags in diagrams.  

---

## 📁 Core Directory Tree  
```
kilo-code/  
├── backend/  
│   └── modules/{domain}-module/       # e.g., payment-fraud-module/src/main/java/... (WebFlux services)  
├── frontend/  
│   └── src/features/{verb}-{domain}/  # e.g., detect-fraud/src/components/... (Vue SFCs)  
├── shared/  
│   └── models/                        # Cross-stack contracts (base DTOs)  
├── tasks/                             # Task-driven hub (NEW: Centralized backlog management)  
│   ├── domains/  
│   │   └── {domain}/                  # e.g., payment-fraud-detection/  
│   │       ├── incomplete.md          # Priority-sorted backlog (High→Med→Low); LLM-populated  
│   │       └── completed.md           # Auto-archived on PR merge via CI  
│   └── global/  
│       ├── sprint-backlog.md          # Cross-domain sprint overview & progress  
│       └── sprint-planning/  
│           └── sprint-YYYY-MM.md      # Detailed sprint artifacts (e.g., goals, impediments)  
├── features/                          # Immutable, versioned feature artifacts (NEW: Lifecycle core)  
│   └── {kebab-slug}/                  # e.g., detect-fraud-patterns/  
│       └── v{version}/                # Semantic versioning (e.g., v1.0, v1.1)  
│           ├── semantic-diagram.md    # Mermaid graph with @Semantic/@Vibe tags  
│           ├── dto/                   # Mirrored contracts  
│           │   ├── {Feature}Dto.java  # Java (e.g., FraudAlertDto)  
│           │   └── {Feature}Dto.ts    # TypeScript mirror  
│           ├── services/              # Backend: WebFlux reactors (e.g., FraudPatternService.kt)  
│           ├── components/            # Frontend: Reusable Vue SFCs (e.g., FraudDashboard.vue)  
│           ├── tests/  
│           │   └── test-data-profiles/ # YAML/JSON scenarios (e.g., high-velocity.yaml)  
│           └── README.md              # @Vibe summary, sprint goals, task links  
└── [other roots]                      # docs/, gradle/, static/ (versioned assets in backend/static/v{X}.{Y}/)  
```

### Example: Task in `incomplete.md`  
```markdown
# Payment Fraud Detection - Incomplete Backlog  

## Priority: High  
- [ ] **ID:** FRAUD-001  
  **Title:** Real-Time Velocity Check Stream  
  **Description:** Detect high-velocity fraud using Flux windowing on payment events.  
  **Input Schema:** PaymentEventDto, UserAccountDto (from shared/models/)  
  **Expected Output:** RiskScoreDto with @Semantic("FraudSignal")  
  **Constraints:** Biometric gating for blocks > $1K; sub-second latency.  
  **Feature Link:** features/detect-fraud-patterns/v1.0/  
  **LLM Prompt:** Generate blueprint for velocity checks with reactive streams.  
  **Test Data Profile:** High-velocity scenario (150 tx/min from high-risk IP).  
```

### Example: `semantic-diagram.md` (v1.0)  
```markdown
# Fraud Pattern Detection Flow - v1.0  

```mermaid  
graph TD  
    A[PaymentEventDto] -->|@Semantic("FraudStream")| B[FraudService.processFlux()]  
    B --> C[RiskScoreDto]  
    C -->|@Vibe("InstantBlock")| D[VueAlertComponent]  
    style B fill:#ff9,stroke:#333  
```  

@Vibe "Zero-latency fraud signals with reactive alerting."  
```

### Example: DTO Mirroring  
**Java (FraudAlertDto.java):**  
```java
/**
 * @Semantic("DataTransfer")
 * @Vibe "Immutable fraud payload for cross-stack sync"
 */
public record FraudAlertDto(
    String alertId,
    String transactionId,
    RiskLevel riskLevel,  // Enum: LOW, MEDIUM, HIGH, CRITICAL
    Instant detectedAt
) {}
```

**TypeScript (FraudAlertDto.ts):**  
```typescript
// Mirrors Java exactly for type safety  
export interface FraudAlertDto {  
    alertId: string;  
    transactionId: string;  
    riskLevel: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';  
    detectedAt: Date;  
}  
```

### Example: Test Data Profile (`high-velocity.yaml`)  
```yaml
id: "high-velocity-test"  
description: "Multiple rapid transactions from same IP"  
inputs:  
  transactions:  
    - amount: 100.00  
      timestamp: "2023-01-01T00:00:00Z"  
      ip: "192.168.1.1"  
    - amount: 150.00  
      timestamp: "2023-01-01T00:00:05Z"  
      ip: "192.168.1.1"  
expected:  
  riskLevel: "CRITICAL"  
  action: "BLOCK"  
```

---

## ⚙️ Enforcement Mechanics  
Automated checks ensure compliance and prevent drift.  

| Rule Category          | Tool/Mechanism                     | Failure Action                        |  
|------------------------|------------------------------------|---------------------------------------|  
| Task Prioritization    | Script: `verify-task-order.js`     | Block PR if unsorted (High first)     |  
| Semantic Diagram Tags  | Gradle Task: `check-diagram`       | Fail build on missing @Semantic/@Vibe |  
| Feature Immutability   | Git Hook/CI: `prevent-edit-merged` | Reject pushes to locked vX.Y dirs     |  
| DTO Mirroring          | Gradle Task: `validate-dto-mirror` | Compile error on field mismatches     |  
| Task Archiving         | CI Script: `archive-task.sh`       | Warn if PR closes without move        |  

**Sample CI Pipeline (GitHub Actions):**  
```yaml  
name: Feature & Task Validation  
on: [pull_request]  

jobs:  
  validate:  
    runs-on: ubuntu-latest  
    steps:  
      - uses: actions/checkout@v3  
      - name: Run Checks  
        run: ./gradlew check-diagram validate-dto-mirror verify-task-order  
      - name: Archive on Merge  
        if: github.event.pull_request.merged == true  
        run: |  
          scripts/archive-task.sh ${{ github.event.pull_request.number }}  
          # Moves task ID from incomplete.md to completed.md  
        env:  
          TASK_ID: ${{ github.event.pull_request.title.split(' ')[0] }}  # e.g., FRAUD-001  
          DOMAIN: payment-fraud-detection  
```

**Sample Gradle Task (build.gradle.kts):**  
```kotlin  
tasks.register("archiveCompletedTask") {  
    doLast {  
        val taskId = System.getenv("TASK_ID") ?: throw GradleException("TASK_ID required")  
        val domain = System.getenv("DOMAIN") ?: throw GradleException("DOMAIN required")  
        // Logic to append task to completed.md and remove from incomplete.md  
        // e.g., exec { commandLine("node", "scripts/move-task.js", taskId, domain) }  
        println("Archived task $taskId for $domain")  
    }  
    onlyIf { System.getenv("CI_MERGE_REQUEST_ID") != null }  
}  
```

---

## 📋 Compliance Checklist  
- [ ] Tasks in `incomplete.md` include ID, Title, Schemas, Constraints, Feature Link, and Priority.  
- [ ] Features use versioned dirs (`vX.Y`) with no post-merge edits.  
- [ ] `semantic-diagram.md` has Mermaid + @Semantic/@Vibe tags.  
- [ ] DTOs mirrored (Java/TS) with identical fields; aligned to shared/models/.  
- [ ] Tests include `test-data-profiles/` with YAML/JSON scenarios.  
- [ ] README.md links tasks and defines @Vibe context.  
- [ ] PRs reference task IDs; CI validates structure.  

---

## 💡 Implementation Notes  
- **Onboarding:** Start with a pilot domain (e.g., payment-fraud-detection) and one feature to validate the flow.  
- **Scaling:** Use LLM tools (e.g., GitHub Copilot or custom prompts) for blueprint generation; integrate with existing ADRs in `docs/architecture/`.  
- **Versioning Static Resources:** Built assets (e.g., JS/CSS) land in `backend/.../static/v{X}.{Y}/` for immutable deploys.  
- **Team Alignment:** Review `completed.md` quarterly to refine prompts and anti-patterns.  

This structure empowers AI-assisted, vibe-coded development with full traceability, reducing manual overhead and ensuring atomic, evolvable features.  

---

## 📜 Revision History  
| Version | Date       | Author        | Changes Summary                              |  
|---------|------------|---------------|----------------------------------------------|  
| 1.0     | 2026-01-22 | LLM-Generated | Base monorepo layout                         |  
