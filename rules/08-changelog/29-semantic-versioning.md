# 29 – Semantic Versioning Policy
<!-- FILE_TITLE: Semantic Versioning Guidelines for Kilo Code -->
<!-- FILE_PATH: rules/08-changelog/29-semantic-versioning.md -->

**File Path:** rules/08-changelog/29-semantic-versioning.md  
**Domain:** TODO: <Project Domain, e.g., "Modular E-commerce Payments Platform">  
**Last Updated:** 2026-01-25
**Status:** Active  

---

## 🎯 Purpose
Define clear, deterministic rules for semantic versioning (SemVer) across all Kilo Code artifacts to clearly communicate change impacts, ensure backward compatibility where possible, and align with reactive architecture, vibe coding principles, and automated release pipelines.

## 📌 Scope
- **Applies to:** JAR artifacts, Vue bundles, shared DTOs/data contracts, Docker images, and public APIs
- **Excludes:** Internal development snapshots (e.g., SNAPSHOT builds) and non-public refactors
- **Dependencies:** [01-project-overview.md](rules/01-project/01-project-overview.md), [06-naming-conventions.md](rules/02-semantics/06-naming-conventions.md), [22-gradle-multimodule.md](rules/06-build/22-gradle-multimodule.md), [14-mermaid-diagrams.md](rules/04-workflow/14-mermaid-diagrams.md)

---

## 🧠 Semantic Anchors
### Version Format: `MAJOR.MINOR.PATCH`
Adopt strict SemVer with vibe-driven annotations to signal intent and impact:

1. **MAJOR (X.0.0):** Breaking changes to public contracts, APIs, or reactive streams
   - Required: `@Semantic("BreakingChange")` annotation on affected elements
   - Vibe Check: Ensures users can anticipate disruptions; deprecate prior versions with `@Deprecated(since = "X.0.0", forRemoval = true)`
   - Example (Java/WebFlux):
     ```java
     @Semantic("BreakingChange")
     @Deprecated(since = "2.0.0", forRemoval = true)
     public Mono<PaymentResponse> legacyPaymentEndpoint(@RequestBody PaymentRequestV1 req) {
         // Deprecated in favor of V2 schema
     }
     ```

2. **MINOR (0.X.0):** Backward-compatible additions, such as new features or reactive enhancements
   - Required: `@Semantic("FeatureAddition")` to highlight scope
   - Vibe Check: Maintains trust in existing integrations while adding value
   - Example (TypeScript/Vue):
     ```typescript
     // @Semantic("FeatureAddition")
     // @Vibe("Real-time payment stream integration")
     export class PaymentStreamComponent extends Vue {
       props: { stream: Flux<PaymentEvent> };
       // New reactive binding for live updates
     }
     ```

3. **PATCH (0.0.X):** Backward-compatible bug fixes, performance tweaks, or documentation updates
   - Optional: `@Semantic("Maintenance")` for clarity
   - Vibe Check: Reinforces stability without fanfare

### Cross-Layer Syncing
- Backend version (from `gradle.properties`) must sync with frontend (`package.json` or `version.ts`).
- Reactive contracts (e.g., Flux/Mono types) trigger MAJOR if signatures change.
- Odd MINOR versions signal exploratory/unstable features; even MINOR indicate production readiness.

---

## ⚙️ Enforcement Mechanics
Automated guards to prevent version drift and ensure compliance:

| Aspect                    | Tool/Mechanism                     | Rule/Failure Action                                         |
|---------------------------|------------------------------------|-------------------------------------------------------------|
| Version Format Validation | Gradle Version Plugin + Regex      | Block builds if not `^\\d+\\.\\d+\\.\\d+$`; throw exception |
| Semantic Tag Presence     | Git Hooks + Static Analysis        | Reject commits without required `@Semantic` annotations     |
| Bump Rationale            | GitHub PR Template/Checklist       | Require "Version Impact" section with Mermaid diagram link  |
| Artifact Sync             | CI Pipeline (e.g., GitHub Actions) | Fail deployment if frontend/backend versions mismatch       |
| Changelog Integration     | PR Validation Script               | Mandate entry with SemVer bump type and diagram reference   |

**Gradle Enforcement Snippet (build.gradle.kts):**
```kotlin
tasks.register("validateSemVer") {
    doLast {
        val version = project.version.toString()
        if (!version.matches(Regex("^\\d+\\.\\d+\\.\\d+$"))) {
            throw GradleException("Version '$version' violates SemVer format")
        }
        // Additional check: Scan for missing @Semantic tags in breaking changes
        // (Integrate with custom annotation processor)
    }
}

// Hook into build pipeline
tasks.named("build").configure { dependsOn("validateSemVer") }
```

**Git Tag Example:**
```bash
git tag -a v2.1.0 -m "MINOR: Added real-time payment streams; see changelog for details"
```

---

## 🔧 Technical Specifications
### Backend (Java/WebFlux)
```properties
# gradle.properties
version=2.1.0  # MAJOR=2 (API evolution), MINOR=1 (new feature), PATCH=0
app.version=${version}  # Inject into application.properties
```

### Frontend (Vue/TypeScript)
```typescript
// src/version.ts
export const API_VERSION = process.env.BACKEND_VERSION || '2.1.0';  // Synced via build env
// Usage in components: Ensure API calls respect versioned endpoints
```

### Artifact Naming Patterns
| Artifact Type | Naming Convention                            | Example                               |
|---------------|----------------------------------------------|---------------------------------------|
| JAR           | <project name>-<module>-<version>.jar        | <project name>-payments-2.1.0.jar     |
| Docker Image  | <registry>/<project-name>/<module>:<version> | ghcr.io/<project name>/payments:2.1.0 |
| Static Assets | /static/v<version>/<asset>                   | /static/v2.1.0/bundle.js              |
| DTO Contracts | <DtoName>V<version>                          | PaymentRequestV2                      |

### Release Workflow (Mermaid Diagram)
```mermaid
graph TD
    A[Code Change Submitted] --> B{Impact Assessment}
    B -->|Breaking| C[MAJOR Bump + @Semantic('BreakingChange') + Deprecate Old]
    B -->|Feature| D[MINOR Bump + @Semantic('FeatureAddition')]
    B -->|Fix| E[PATCH Bump + @Semantic('Maintenance') optional]
    C --> F[Update Changelog + Mermaid Diagram]
    D --> F
    E --> F
    F --> G[PR Review: Validate Tags & Rationale]
    G -->|Approved| H[Merge + Auto-Tag + Publish Artifacts]
    H --> I[Sync Frontend/Backend Versions]
```

---

## 📋 Compliance Checklist
- [ ] Version adheres to `MAJOR.MINOR.PATCH` and is updated in `gradle.properties`/`package.json`
- [ ] All breaking changes include `@Semantic("BreakingChange")` and deprecation warnings
- [ ] PR description includes bump rationale (e.g., "[MINOR] Added reactive stream support") and links to Mermaid diagram
- [ ] Changelog entry references the version bump and affected artifacts
- [ ] `validateSemVer` Gradle task passes; no version mismatches across layers
- [ ] DTOs and APIs use versioned naming (e.g., V2 suffixes for evolutions)
- [ ] Docker tags and static assets follow versioned paths

---

## 🔗 Interlocks
- **[06-naming-conventions.md](rules/02-semantics/06-naming-conventions.md):** Version suffixes for DTOs and modules
- **[22-gradle-multimodule.md](rules/06-build/22-gradle-multimodule.md):** Multi-module build sequencing and version propagation
- **[14-mermaid-diagrams.md](rules/04-workflow/14-mermaid-diagrams.md):** Required for visualizing change impacts in PRs/changelogs
- **[24-artifact-naming.md](rules/06-build/24-artifact-naming.md):** Consistent patterns for published artifacts (if applicable)

---

## 📜 Revision History
| Version | Date       | Author        | Changes Summary                              |
|---------|------------|---------------|----------------------------------------------|
| 1.0     | 2026-01-25 | LLM-Generated | Initial draft with core structure.           |

---

## Implementation Guide
1. **Setup Placeholders:** Replace all `TODO:` items with project-specific details (e.g., domain, team, current date).
2. **Integrate Tools:** Add the Gradle task to your root `build.gradle.kts` and configure GitHub Actions for PR checks.
3. **Initial Rollout:** 
   - Audit current artifacts for version alignment.
   - Create a baseline changelog in `/changelog/` with Mermaid diagrams for recent changes.
   - Tag the next release following this policy (e.g., `git tag -a v1.0.0`).
4. **Vibe Tip:** Versions should evoke stability—use them to build user confidence, like a "v2.0.0" signaling a mature evolution.
5. **Common Pitfalls to Avoid:** Prevent module version drift by enforcing root-level versioning; always test reactive streams post-bump.

This policy enhances automation, reduces integration errors, and maintains a cohesive "vibe" across the Kilo Code stack. For customizations, reference interlocks or consult the architecture team.
