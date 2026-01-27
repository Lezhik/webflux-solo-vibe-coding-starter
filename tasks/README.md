# Task Repository - Atomic Feature Development Tracker

**File Path:** `/tasks/README.md`  
**Domain:** Cross-Project Task Management  
**Last Updated:** 2026-01-27
**Status:** Active  

---

## 🎯 Purpose  
This repository centralizes atomic feature development tracking across domains (e.g., payment-fraud-detection) in a Java/Spring WebFlux + Vue/TypeScript stack. It enforces LLM-assisted planning, GitOps-style automation for task lifecycles, PR-triggered archival, and semantic consistency. Key goals include traceability from ideation to deployment, vibe-driven development, and audit-ready reporting to maintain velocity in financial-grade systems.

---

## 📂 Repository Structure  

```
/tasks/
├── domains/                      # Domain-specific backlogs (e.g., payment-fraud-detection)
│   └── {domain}/
│       ├── incomplete.md         # Prioritized active tasks
│       └── completed.md          # Immutable archive of merged features
├── global/
│   └── sprint-backlog.md         # Cross-domain sprint overview with progress metrics
├── hooks/                        # Automation scripts
│   ├── move-task.js              # Migrates tasks from incomplete to completed on PR merge
│   ├── verify-task-tracking.js   # Validates format, links, and compliance
│   └── generate-sprint-report.js # Generates reports (CLI-integrated)
└── README.md                     # This file: guidelines and quick start
```

---

## 📋 Task Entry Format  

### `incomplete.md` (Active Backlog)  
Prioritize tasks by domain sprint. Use consistent Markdown tables for readability.

```markdown
## {Domain Name} Sprint - YYYY-MM-DD

| Priority | Task ID       | Description                          | Feature Path                     | Est. Effort | @Vibe (Optional)          |
|----------|---------------|--------------------------------------|----------------------------------|-------------|---------------------------|
| High     | F-26-02-15-00 | Real-time fraud scoring via WebFlux  | /features/detect-fraud-patterns  | 2 days      | "Instant risk assessment" |
| Med      | F-26-02-15-01 | Vue dashboard for fraud alerts       | /features/fraud-dashboard        | 1.5 days    | "Live risk visualization" |
| Low      | F-26-02-16-00 | Audit log integration                | /features/audit-logs             | 1 day       | ""                        |
```

**Field Requirements:**  
- **Priority:** High/Med/Low (sorted descending).  
- **Task ID:** `F-{YY}-{MM}-{DD}-{XX}` (e.g., F-26-02-15-00; maps to branch `feat/F-26-02-15-00-{kebab-description}`).  
- **Description:** Concise, actionable summary tied to stack (e.g., WebFlux streams, Vue components).  
- **Feature Path:** Absolute path to `/features/{slug}/` directory (e.g., `/features/detect-fraud-patterns`).  
- **Est. Effort:** Effort in days (e.g., 2, 1.5; used for burndown charts).  
- **@Vibe:** Optional semantic tag from feature README (propagated on archival).  

### `completed.md` (Immutable Archive)  
Append-only; no manual edits post-merge. Enforced via protected branches.

```markdown
## Completed Features - {Domain} (YYYY-MM-DD)

| Task ID       | Feature Path                     | Merged PR | Date       | @Vibe (Optional)          |
|---------------|----------------------------------|-----------|------------|---------------------------|
| F-26-01-30-00 | /features/detect-fraud-patterns  | #451      | 2026-02-14 | "Sub-second fraud detection" |
| F-26-02-10-00 | /features/user-auth-flow         | #452      | 2026-02-12 | "Seamless secure login"    |
```

---

## 🔄 Lifecycle Workflow  

### Phase 1: Initialization  
Use CLI for LLM-assisted task creation (integrates with Kilo Code tools for stubs/DTOs).

```bash
kilo-cli init-task \
  --domain payment-fraud-detection \
  --id F-26-02-15-00 \
  --desc "Real-time fraud scoring via WebFlux" \
  --feature-path /features/detect-fraud-patterns \
  --priority high \
  --effort 2 \
  --vibe "Instant risk assessment"
```
- This auto-appends to `incomplete.md` and generates branch suggestion, feature stubs (e.g., DTOs in Java, Vue components).

### Phase 2: Development  
1. Branch: `git checkout -b feat/F-26-02-15-00-real-time-fraud-scoring`.  
2. Implement per [Feature Lifecycle](../rules/04-workflow/13-feature-lifecycle.md):  
   - Backend: Reactive WebFlux (`Mono`/`Flux`), `@Semantic` annotations.  
   - Frontend: Vue 3 with `<script setup lang="ts">`, Vitest tests.  
   - LLM assistance: Generate diagrams, test profiles (`/test-profiles/{domain}/`).  
3. Validate locally: `./gradlew verifyTaskTracking && node hooks/verify-task-tracking.js --strict`.  
4. Ensure 80%+ coverage (JaCoCo for Java, Vitest for TS).

### Phase 3: Merge & Archival  
- PR Template: Include `Resolves #F-26-02-15-00` in body/title.  
- On merge: CI triggers migration.  
- Auto-actions:  
  - Remove from `incomplete.md`.  
  - Append to `completed.md` with PR #, date, @Vibe (from feature README).  
  - Update `global/sprint-backlog.md` with progress (% complete, burndown).

---

## ⚙️ Automation & Enforcement  

### 1. Pre-Commit Hooks  
Enforce via Husky (`.husky/pre-commit`):  
```bash
#!/bin/sh
node hooks/verify-task-tracking.js --strict && ./gradlew verifyTaskTracking && ./gradlew verifyFeatureLinks
```
- Checks: Format, feature path existence, unresolved TODOs, TODO resolution in code.

### 2. CI/CD Pipeline (GitHub Actions: `.github/workflows/tasks.yml`)  
```yaml
name: Task Lifecycle Management
on:
  pull_request:
    types: [opened, synchronize, closed]
  push:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: node hooks/verify-task-tracking.js --strict
      - run: ./gradlew verifyTaskTracking verifyFeatureLinks
    if: github.event_name != 'pull_request' || github.event.action != 'closed'

  archive:
    if: github.event.pull_request.merged == true
    needs: validate
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Extract Task ID
        id: task
        run: echo "TASK_ID=$(echo '${{ github.event.pull_request.body }}' | grep -o 'Resolves #F-[0-9]\+-[0-9]\+-[0-9]\+-[0-9]\+' | cut -c10-)" >> $GITHUB_OUTPUT
      - name: Migrate Task
        run: node hooks/move-task.js --task ${{ steps.task.outputs.TASK_ID }} --domain payment-fraud-detection
      - name: Generate Sprint Report
        run: node hooks/generate-sprint-report.js --domains payment-fraud-detection,user-authentication --output global/sprint-backlog.md
```

### 3. Gradle Integration (build.gradle.kts)  
```kotlin
tasks.register<Exec>("verifyTaskTracking") {
    commandLine("node", "hooks/verify-task-tracking.js")
    dependsOn("compileKotlin")
}

tasks.register<Exec>("verifyFeatureLinks") {
    commandLine("./gradlew", "scanFeaturePaths")  // Custom task to check /features/ dirs
}
```
- Run: `./gradlew verifyTaskTracking` to block invalid commits/PRs.

### 4. CLI Toolkit (kilo-cli)  
- `kilo-cli generate-sprint-report --domains payment-fraud-detection --output global/sprint-backlog.md`: Outputs progress charts, vibe analysis.  
- `kilo-cli validate-domain --domain payment-fraud-detection`: Scans for orphaned tasks.

---

## 🛡️ Compliance Matrix  

| Rule/Check                  | Mechanism/Tool                  | Failure Action                  |
|-----------------------------|---------------------------------|---------------------------------|
| Task ID/Format              | `verify-task-tracking.js`       | Block commit/PR                 |
| Feature Path Validity       | Gradle `verifyFeatureLinks`     | Fail build                      |
| @Vibe Propagation           | `move-task.js` (from README)    | Auto-insert default if missing  |
| Immutability (completed.md) | GitHub Protected Branches       | Reject force-pushes             |
| Cross-Stack Parity          | CI semantic scan                | Warn/Block on mismatch          |
| Effort/Burndown Accuracy    | Report generation script        | Notify on discrepancies         |

**Anti-Patterns to Avoid:**  
- Manual edits to `completed.md` (use new tasks for iterations).  
- Orphaned tasks (no feature link).  
- Cross-domain mixing in per-domain files.  
- Missing `Resolves #TASK_ID` in PRs.

---

## 📊 Reporting & Analytics  
- **Sprint Overview (`global/sprint-backlog.md`):** Auto-generated with:  
  - Progress: % complete per domain.  
  - Burndown: Remaining effort chart (Markdown ASCII or SVG via CLI).  
  - Vibe Trends: Distribution of semantic tags.  
- Command: `kilo-cli generate-sprint-report --all-domains`.  
- Metrics: Velocity (tasks/day), coverage from JaCoCo/Vitest.

---

## 🚀 Getting Started  

1. **Setup Domain:**  
   ```bash
   mkdir -p tasks/domains/payment-fraud-detection
   touch tasks/domains/payment-fraud-detection/{incomplete,completed}.md
   ```

2. **Create First Task:**  
   ```bash
   kilo-cli init-task \
     --domain payment-fraud-detection \
     --id F-26-02-15-00 \
     --desc "Real-time fraud scoring" \
     --feature-path /features/detect-fraud-patterns \
     --priority high
   ```

3. **Develop:**  
   ```bash
   git checkout -b feat/F-26-02-15-00-real-time-scoring
   # Implement, test, commit with pre-hooks passing
   ```

4. **Merge & Verify:**  
   - PR: "feat: real-time fraud scoring (Resolves #F-26-02-15-00)".  
   - Post-merge: Check `completed.md` and run `kilo-cli generate-sprint-report`.

---

## 📜 Revision History  
| Version | Date       | Author          | Changes Summary                                          |  
|---------|------------|-----------------|----------------------------------------------------------|  
| 1.0     | 2026-01-27 | LLM-Generated   | Initial structure with semantic anchors and enforcement. |  

---

> **Key Innovation:** Seamlessly blends LLM planning with automated GitOps, ensuring vibe-infused, traceable development across reactive stacks. For questions, reference [Vibe Coding Guidelines](../rules/01-principles/05-vibe-coding.md) or [Feature Lifecycle](../rules/04-workflow/13-feature-lifecycle.md).
