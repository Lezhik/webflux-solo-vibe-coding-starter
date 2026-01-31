# 25 – Data Migration – Versioned Seeding
**File Path:** rules/06-build/25-data-migration.md  
**Domain:** TODO: <business-context> (e.g., "Secure e-commerce payment processing with real-time event streaming")
**Last Updated:** 2026-01-25
**Status:** Active  

<!-- FILE_TITLE: Data Migration Rules -->  
<!-- FILE_PATH: rules/06-build/25-data-migration.md -->  

---

## 🎯 Purpose
This rule outlines a version-controlled, reactive strategy for seeding and migrating data in MongoDB environments. It ensures reproducible setups for development, testing, and staging while maintaining semantic alignment between backend entities (Java/WebFlux), frontend models (Vue/TypeScript), and fixture data. The approach emphasizes idempotency, rollback capabilities, and integration with reactive architectures to prevent data inconsistencies during builds and deployments.

---

## 📌 Scope
- **Applies to:** MongoDB initialization, fixture-based seeding for test/staging environments, and non-production data migrations.
- **Excludes:** Production database operations (managed via dedicated Ops tools like Flyway or Liquibase); blocking (non-reactive) data access patterns.
- **Dependencies:** 
  - `rules/05-test-strategy/20-test-data-profiles.md` (for test data management and profiles).
  - `rules/06-build/22-gradle-multimodule.md` (for build sequencing and multi-module integration).

---

## 🧠 Semantic Anchors
Semantic anchors provide declarative metadata to enforce intent, versioning, and cross-stack consistency. Use these annotations to tag migrations and fixtures, enabling automated validation and generation.

### 1. Annotation Taxonomy
- **`@DataMigration`**: Custom annotation for marking seeding/migration classes or methods.
- **`@Versioned("vX.Y")`**: Specifies the data version (e.g., `@Versioned("v1.2")`).
- **`@Semantic("<domain-context>")`**: Defines the business or technical domain (e.g., `@Semantic("UserAggregateMigration")`).
- **`@Vibe("<intent-description>")`**: Captures the migration's behavioral vibe (e.g., `@Vibe("Idempotent upserts with atomic rollback")`).

**Java Example (Reactive Migration):**
```java
import reactor.core.publisher.Flux;

@DataMigration
@Versioned("v1.3")
@Semantic("PaymentGatewaySeed")
@Vibe("Atomic upsert of baseline currency rates for staging; supports rollback via version diff")
public Flux<CurrencyRate> seedCurrencyRatesAsync() {
    return mongoTemplate.upsert(Flux.fromIterable(loadFixture("currency_rates_v1.3.yml")));
}
```

### 2. Fixture Metadata Structure
Fixtures use YAML for declarative data with embedded semantic headers. Paths follow a versioned hierarchy: `/src/main/resources/mongo-init/v<major>_<minor>/<domain>_<purpose>.yml`.

**YAML Fixture Example:**
```yaml
# /mongo-init/v1.2/auth_users.yml
# @Semantic: UserAuthAggregate
# @Vibe: Baseline idempotent seed for authentication tests; edge cases included
# @Versioned: v1.2
collections:
  users:
    - _id: "user-001"
      email: "admin@example.com"
      roles: ["ADMIN"]
      status: "ACTIVE"
indexes:
  users:
    keys: { email: 1 }
    options: { unique: true }
```

### 3. Versioning & Naming Patterns
- **Paths:** `/<module>/src/main/resources/mongo-init/vX.Y/` (e.g., `v1.0/auth_seed.yml`).
- **Methods:** `<action>Seed<Domain>Async()` (e.g., `loadUserPermissionsSeedAsync()`).
- **Files:** `<domain>_<purpose>_vX.Y.yml` (e.g., `payments_edgecases_v1.1.yml`).

---

## ⚙️ Enforcement Mechanics
Enforce compliance through integrated tools in the build pipeline, CI/CD, and pre-commit hooks. This prevents invalid migrations from propagating.

| Rule Category          | Tool/Mechanism                        | Failure Action                  |
|------------------------|---------------------------------------|---------------------------------|
| Version Validation     | Custom Gradle Task                    | Fail Build / Block PR           |
| Semantic Headers       | YAML Linter (e.g., yamllint)          | Reject Commit (pre-commit hook) |
| Reactive Compliance    | Checkstyle / SonarQube                | Fail Compilation / Warn in CI   |
| Idempotency Checks     | Static Analysis (e.g., custom script) | Block Merge on Non-Upsert Ops   |
| Cross-Stack Alignment  | TypeScript Generator Task             | TypeScript Compilation Errors   |

**Gradle Enforcement Example:**
```kotlin
// build.gradle.kts
tasks.register("validateMigrations") {
    dependsOn(":backend:compileJava", ":frontend:compileTypeScript")
    doLast {
        // Validate fixture paths and headers
        fileTree("$projectDir/src/main/resources/mongo-init").visit { file ->
            if (!file.name.matches(Regex("v\\d+\\.\\d+_.*\\.yml"))) {
                throw GradleException("Invalid fixture versioning: ${file.name}")
            }
            // Check for required YAML headers
            val content = file.readText()
            if (!content.contains("@Semantic:") || !content.contains("@Vibe:")) {
                throw GradleException("Missing semantic headers in ${file.name}")
            }
        }
        // Run YAML linter
        exec { commandLine("yamllint", "$projectDir/src/main/resources/mongo-init") }
    }
}

// Integrate into main build
tasks.named("build") { dependsOn("validateMigrations") }
```

---

## 🔧 Technical Specifications
### Backend (Java/Spring WebFlux with MongoDB)
- **Operations:** Mandate reactive types (`Flux<T>` or `Mono<Void>`) and idempotent methods (e.g., `upsert` over `insert` to avoid duplicates).
- **Error Handling:** Include rollback logic via transaction boundaries or version diffs.
- **Loading Fixtures:** Use libraries like Jackson or SnakeYAML for deserialization into entity streams.

**Extended Java Snippet:**
```java
@Versioned("v1.0")
public Mono<Void> migrateUserRolesAsync() {
    return loadYamlFixture("roles_v1.0.yml")
        .flatMapMany(Flux::fromIterable)
        .flatMap(mongoTemplate::save)
        .then();
}
```

### Frontend Synchronization (Vue/TypeScript)
- **Generation:** Automate TypeScript interface creation from YAML fixtures via a Gradle/Node task to ensure prop-field alignment.
- **Example Generated Interface:**
```typescript
// auto-generated from users_v1.2.yml
export interface UserSeed {
  _id: string;
  email: string;
  roles: string[];
  status: 'ACTIVE' | 'LOCKED' | 'INACTIVE';
}
```

### File Organization
```
src/
├── main/
│   └── resources/
│       └── mongo-init/
│           ├── v1.0/                # Stable baseline
│           │   ├── users.yml
│           │   └── payments.yml
│           └── v1.1/                # Incremental updates
│               └── auth_edge.yml
└── test/
    └── resources/
        └── profiles/                 # Test-specific overrides
            └── staging-fixtures.yml
```

---

## 📋 Compliance Checklist
- [ ] Fixtures are stored in versioned paths (`/vX.Y/`) with required `@Semantic` and `@Vibe` headers.
- [ ] All migration methods are annotated with `@DataMigration` and end in `*Async`.
- [ ] Operations use reactive streams (`Flux`/`Mono`) and idempotent patterns (e.g., `upsert`).
- [ ] Gradle task `validateMigrations` executes without errors.
- [ ] Frontend interfaces are generated and aligned with fixture schemas.
- [ ] Test profiles reference versioned fixtures (per `20-test-data-profiles.md`).

---

## 🔗 Interlocks
- **Versioning Alignment:** Follows semantic versioning from `rules/02-semantics/30-semantic-versioning.md`.
- **Build Pipeline:** Executes post-compilation in multi-module setups (`22-gradle-multimodule.md`).
- **Testing Integration:** Fixtures feed into boundary tests (`rules/05-test-strategy/21-boundary-scenarios.md`).
- **Semantics:** Adheres to naming conventions in `rules/02-semantics/06-naming-conventions.md`.

---

## 📜 Revision History
| Version | Date       | Author        | Changes Summary                              |
|---------|------------|---------------|----------------------------------------------|
| 1.0     | 2026-01-25 | LLM-Generated | Initial draft with core structure.           |

---

## 🚀 Implementation Guide
1. **Initialize Structure:**
   ```bash
   mkdir -p src/main/resources/mongo-init/v1.0
   # Add initial fixture: users_v1.0.yml with semantic headers
   ```

2. **Develop Migration Logic:**
   - Annotate Java classes/methods with `@DataMigration`, `@Versioned`, etc.
   - Implement reactive seeding (e.g., `Flux.fromIterable(fixtures).flatMap(mongoTemplate::upsert)`).

3. **Validate Locally:**
   ```bash
   ./gradlew validateMigrations
   # Ensure no errors; run yamllint for YAML syntax
   ```

4. **Sync and Test:**
   - Generate TS interfaces: `./gradlew generateTsFromFixtures`.
   - Reference in tests: `@ActiveProfiles("v1.0-users")` or load via `mongoTemplate` in integration tests.

5. **CI/CD Integration:** Add `validateMigrations` to PR gates; use it in staging deployment hooks.

**Anti-Patterns to Avoid:**
- ❌ Blocking calls: `mongoTemplate.insert(users)` (use reactive equivalents).
- ❌ Unversioned fixtures: `data.yml` (always include `vX.Y`).
- ❌ Missing semantics: Fixtures without `@Semantic` (blocks automated discovery).

**Key Innovation:** Embed `@dataflow` directives in YAML for generating dependency graphs (e.g., "users → payments"), enabling automated seed ordering in pipelines.

This rule promotes a declarative, zero-downtime approach to data evolution, reducing environment drift across teams.
