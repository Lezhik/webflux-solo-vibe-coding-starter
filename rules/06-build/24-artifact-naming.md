# 24 – Artifact Naming & Versioning Standards

**File Path:** rules/06-build/24-artifact-naming.md  
**Domain:** TODO: <Project Name> (e.g., "E-commerce Payment Gateway")  
**Last Updated:** 2026-01-25
**Status:** Active  

<!-- FILE_TITLE: Artifact Naming & Versioning Standards -->  
<!-- FILE_PATH: rules/06-build/24-artifact-naming.md -->

---

## 🎯 Purpose
Establish deterministic, verifiable naming conventions for build artifacts to ensure traceability, efficient cache management, and synchronized versioning across backend, frontend, and test components in a multi-module reactive architecture. This prevents deployment issues, supports automated CI/CD pipelines, and aligns with semantic versioning practices.

## 📌 Scope
- **Applies to:** Java JARs (Spring Boot/WebFlux modules), Vue/TypeScript bundles, test artifacts (JUnit/Cypress), and static resources.
- **Excludes:** Source files, transient build intermediates, IDE-generated artifacts, and third-party binaries.
- **Dependencies:** 
  - [22-gradle-multimodule.md](22-gradle-multimodule.md) (build sequencing).
  - [23-static-resources.md](23-static-resources.md) (asset bundling).
  - [29-semantic-versioning.md](/rules/08-changelog/29-semantic-versioning.md) (version format: MAJOR.MINOR.PATCH).

---

## 🧠 Semantic Anchors

1. **Core Naming Principles**:
   - All artifacts prefix with TODO: `project-name-` for project identity.
   - Use semantic versioning (e.g., `1.2.0`) from a single source (`gradle.properties` root).
   - Module names are slugified (lowercase, hyphens) and descriptive (e.g., `payment-service`).
   - Static assets use versioned paths to enable cache invalidation.

2. **Artifact Patterns**:
   - **Backend JAR**: `project-name-<module>-<version>.jar`
   - **Frontend Bundle**: `project-name-frontend-<version>.[js|css]`
   - **Test Artifact**: `project-name-<module>-test-<profile>-<version>.jar` (e.g., `<profile>` = `integration`, `perf`).
   - **Static Path**: `/static/v<version>/` (e.g., `/static/v1.2.0/`).
   - **TODO**: Replace all "project-name-" with your project name

   Example configurations:
   ```kotlin
   // Backend (module build.gradle.kts)
   tasks.jar {
       archiveBaseName.set("project-name-${project.name}")
       archiveVersion.set(project.version.toString())  // Syncs from root gradle.properties
   }
   ```

   ```javascript
   // Frontend (vite.config.js or build script)
   export default defineConfig({
     build: {
       outDir: `../src/main/resources/static/v${process.env.npm_package_version}`,
       rollupOptions: {
         output: {
           entryFileNames: 'project-name-frontend-${version}.js',
           chunkFileNames: 'project-name-frontend-${version}-[hash].js'
         }
       }
     }
   });
   ```

   HTML usage example:
   ```html
   <!-- Production -->
   <script src="/static/v1.2.0/project-name-frontend-1.2.0.js" defer></script>
   <link rel="stylesheet" href="/static/v1.2.0/project-name-frontend-1.2.0.css">
   
   <!-- Development -->
   <script src="/static/dev/project-name-frontend-dev.js" defer></script>
   ```

---

## ⚙️ Enforcement Matrix

| Rule Type              | Tool/Mechanism                     | Failure Action            | Example Command                  |
|------------------------|------------------------------------|---------------------------|----------------------------------|
| JAR Naming Validation  | Gradle `verifyArtifactNaming` Task | Fail build                | `./gradlew verifyArtifactNaming` |
| Version Consistency    | Git pre-push hook                  | Block commit              | Custom script in `.git/hooks/`   |
| Frontend Asset Paths   | ESLint/Webpack plugin              | Warning + Auto-fix        | `npm run lint`                   |
| Test Artifact Naming   | Checkstyle/JUnit extension         | Report in CI              | Integrated in `./gradlew test`   |

**Root Gradle Task for Validation**:
```kotlin
// root build.gradle.kts
tasks.register("verifyArtifactNaming") {
    doLast {
        subprojects.forEach { sp ->
            val jarTask = sp.tasks.findByName("jar") as? Jar
            jarTask?.let {
                val baseName = it.archiveBaseName.get()
                if (!baseName.matches(Regex("^project-name-[a-z0-9-]+$"))) {
                    throw GradleException("Invalid JAR base name in ${sp.name}: $baseName (must start with 'project-name-' and use hyphens)")
                }
                if (it.archiveVersion.get() != project.version.toString()) {
                    throw GradleException("Version mismatch in ${sp.name}: ${it.archiveVersion.get()} != ${project.version}")
                }
            }
        }
        // Frontend check (assuming dist exists)
        val frontendJs = file("$buildDir/dist/project-name-frontend-${project.version}.js")
        if (!frontendJs.exists()) {
            throw GradleException("Missing frontend bundle: ${frontendJs.absolutePath}")
        }
    }
}
```

**CI/CD Integration Example (GitHub Actions)**:
```yaml
# .github/workflows/build.yml
name: Build and Validate
on: [push]
jobs:
  validate-artifacts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK
        uses: actions/setup-java@v4
        with: { java-version: '17' }
      - name: Build and Verify
        run: |
          ./gradlew clean build verifyArtifactNaming
      - name: Validate Frontend
        run: npm ci && npm run build && npm run lint  # Includes asset naming checks
```

---

## 🔧 Technical Specifications

### Version Synchronization
- **Single Source of Truth**: Define `version=1.2.0` in root `gradle.properties`.
- **Frontend Sync**: Use environment injection or Gradle task to propagate to `package.json`:
  ```bash
  # Build script hook
  echo "VERSION=$GRADLE_VERSION" > .env
  npm run build:versioned
  ```
- **Profiles for Tests**: Append via Gradle properties (e.g., `-PtestProfile=integration`) to generate targeted artifacts.

### Full Examples
- **Backend Output**: `project-name-user-auth-1.2.0.jar`
- **Frontend Output** (in `src/main/resources/static/v1.2.0/`): `project-name-frontend-1.2.0.js`, `project-name-frontend-1.2.0.css`
- **Test Output**: `project-name-payment-service-test-integration-1.2.0.jar`
- **Dev Fallback**: Non-versioned paths for local dev (e.g., `/static/dev/`).

Avoid hardcoded versions; always reference `project.version` or equivalents.

---

## 📋 Compliance Checklist
- [ ] All backend JARs follow `project-name-<module>-<version>.jar` pattern.
- [ ] Frontend bundles are named `project-name-frontend-<version>.[js|css]` and placed in `/static/v<version>/`.
- [ ] Test artifacts include a `<profile>` segment (e.g., `-test-integration-`).
- [ ] Versions are synchronized across stacks (no hardcoding).
- [ ] `./gradlew verifyArtifactNaming` runs without errors locally and in CI.
- [ ] Static paths prevent cache issues (versioned in prod, dev fallback in local).

---

## 🔗 Interlocks
- **Build Structure**: Builds on multi-module setup from [22-gradle-multimodule.md](22-gradle-multimodule.md).
- **Asset Management**: Integrates with bundling rules in [23-static-resources.md](23-static-resources.md).
- **Versioning**: Strictly follows [29-semantic-versioning.md](/rules/08-changelog/29-semantic-versioning.md) for release increments.
- **Naming Conventions**: Consistent with general rules in [06-naming-conventions.md](/rules/01-general/06-naming-conventions.md).

---

## 📜 Revision History
| Version | Date       | Author          | Changes Summary                                 |
|---------|------------|-----------------|-------------------------------------------------|
| 1.0     | 2026-01-25 | LLM-Generated   | Initial standards with enforcement and examples |
