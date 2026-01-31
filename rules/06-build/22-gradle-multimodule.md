# 22 – Gradle Multimodule Build Configuration

**File Path:** rules/06-build/22-gradle-multimodule.md  
**Domain:** TODO: [Specify business domain, e.g., "Secure E-commerce Payment Processing"]  
**Last Updated:** 2026-01-25
**Status:** Active  

<!-- FILE_TITLE: Gradle Multimodule Build Orchestration for Reactive Full-Stack -->  

---

## 🎯 Purpose  
This rule establishes a Gradle-based multimodule build configuration for a modular monorepo, enabling unified compilation, testing, and deployment of a Java/Spring Boot/WebFlux backend alongside a Vue/TypeScript frontend. It enforces reactive architecture principles, semantic task naming, versioned static resource handling, and cross-stack synchronization to maintain vibe-coding standards and prevent build inconsistencies.

---

## 📌 Scope  
- **Applies to:** Root `build.gradle.kts`, `settings.gradle.kts`, and subprojects (`backend/`, `frontend/`, `common/`).  
- **Excludes:** Non-Gradle build tools (e.g., Maven, npm standalone), third-party plugin internals.  
- **Dependencies:**  
  - [03-folder-structure.md](../01-project/03-folder-structure.md) for directory layout.  
  - [23-static-resources.md](23-static-resources.md) for asset versioning.  
  - [17-test-pyramid.md](../05-test-strategy/17-test-pyramid.md) for test sequencing.  
  - [24-artifact-naming.md](24-artifact-naming.md) for naming conventions.  

---

## 🧠 Semantic Anchors  

### 1. Task Nomenclature (Verb-Domain-Context Pattern)  
- **Why:** Promotes intent-driven builds with clear dependencies, reducing errors in reactive flows.  
- **Vibe:** "Orchestrates non-blocking, cross-stack compilation while isolating module boundaries."  
- **Example:**  
  ```kotlin
  @Semantic("BuildOrchestrator")
  @Vibe("Ensures sequential compilation to align backend DTOs with frontend contracts")
  tasks.register("buildFullStackReactive") {
      dependsOn(":backend:compileJava", ":frontend:compileTypeScript")
      finalizedBy(":backend:assemble", ":frontend:assembleStaticAssets", "validateCrossStackSync")
  }
  ```  

### 2. Module Boundaries and Inclusions  
- **Why:** Defines clear separation for backend (reactive core), frontend (view layer), and shared logic.  
- **Vibe:** "Modular isolation prevents tight coupling in reactive event streams."  
- **Example:**  
  ```kotlin
  // settings.gradle.kts
  rootProject.name = "kilo-code"   // TODO: Update to project name
  include(":backend")              // @Semantic("ReactiveCore")
  include(":frontend")             // @Semantic("ViewLayer")
  include(":common")               // @Semantic("SharedContracts")
  ```  

### 3. Cross-Stack Validation  
- **Why:** Verifies DTO/schema alignment between Java and TypeScript for reactive data flows.  
- **Vibe:** "Guards against desync in backend-frontend contracts during builds."  
- **Example:**  
  ```kotlin
  @Semantic("ComplianceValidator")
  tasks.register("validateCrossStackSync") {
      doLast {
          // TODO: Integrate tool like 'gradle-dto-validator' or custom script for Java/TS schema check
          exec { commandLine("./scripts/validate-dtos.sh") }
          println("Cross-stack contract alignment verified")
      }
  }
  ```  

---

## ⚙️ Enforcement Mechanics  

| Rule Type              | Tool/Mechanism                  | Failure Action             | Example Configuration                          |
|------------------------|---------------------------------|----------------------------|------------------------------------------------|
| Circular Dependencies  | Gradle `:dependencies` insight  | Fail build & notify CI     | `gradle dependencyInsight --dependency common` |
| Task Sequencing        | Task Graph Dependencies         | Block parallel execution   | `dependsOn` in root tasks                      |
| Static Resource Sync   | Copy Task with Versioning       | Invalidate cache & retry   | `/static/v${version}/` path validation         |
| Module Isolation       | Project Dependencies Check      | Reject merge in PR         | No direct backend → frontend deps              |

**CI/CD Integration Snippet (GitHub Actions):**  
```yaml
name: Multimodule Build CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
      - name: Setup Node.js 20
        uses: actions/setup-node@v4
        with:
          node-version: '20.x'
      - name: Validate Modules
        run: ./gradlew validateCrossStackSync
      - name: Full Stack Build
        run: ./gradlew buildFullStackReactive -Pversion=${{ github.run_number }}.0-SNAPSHOT
      - name: Run Tests
        run: ./gradlew test
```

---

## 🔧 Technical Specifications  

### A. Root Configuration  
```kotlin
// settings.gradle.kts
pluginManagement {
    repositories {
        gradlePluginPortal()
        mavenCentral()
    }
    plugins {
        id("org.springframework.boot") version "3.5.10"
        id("io.spring.dependency-management") version "1.1.4"
        id("com.github.node-gradle.node") version "7.0.2"  // For frontend Node integration
    }
}

rootProject.name = "kilo-code"  // TODO: Set project name
include(":backend", ":frontend", ":common")

// build.gradle.kts (root)
allprojects {
    group = "com.kilocode"  // TODO: Set organization
    version = "1.0.0-SNAPSHOT"  // TODO: Align with 24-artifact-naming.md
}

subprojects {
    apply(plugin = "java")
    repositories {
        mavenCentral()
    }
    java {
        toolchain {
            languageVersion = JavaLanguageVersion.of(21)
        }
    }
}
```

### B. Backend Module (Java/Spring Boot/WebFlux)  
```kotlin
// backend/build.gradle.kts
plugins {
    id("org.springframework.boot")
    id("io.spring.dependency-management")
    id("java")
}

dependencies {
    implementation(project(":common"))
    implementation("org.springframework.boot:spring-boot-starter-webflux")
    implementation("org.springframework.boot:spring-boot-starter-data-mongodb-reactive")  // TODO: Adjust for domain
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testImplementation("io.projectreactor:reactor-test")
}

tasks.named<JavaCompile>("compileJava") {
    options.compilerArgs.addAll(listOf("-Xlint:unchecked", "-Xlint:deprecation", "--enable-preview"))
}

tasks.named<Test>("test") {
    useJUnitPlatform()
    systemProperty("spring.profiles.active", "test")
}

tasks.named<ProcessResources>("processResources") {
    filesMatching("**/static/**") {
        expand(projectVersion: project.version)
    }
}
```

### C. Frontend Module (Vue/TypeScript)  
```kotlin
// frontend/build.gradle.kts
plugins {
    id("com.github.node-gradle.node")
}

node {
    version = "20.10.0"
    npmVersion = "10.2.3"
    download = true
    workDir = file("${projectDir}/src/main/frontend")
}

tasks.register("npmInstall") {
    dependsOn("clean")
    doLast {
        exec {
            workingDir = file("src/main/frontend")
            commandLine("npm", "ci")  // Use ci for reproducible builds
        }
    }
}

@Semantic("FrontendCompiler")
tasks.register("compileTypeScript") {
    dependsOn("npmInstall")
    inputs.dir("src/main/frontend/src")
    outputs.dir("${buildDir}/dist")
    doLast {
        exec {
            workingDir = file("src/main/frontend")
            commandLine("npm", "run", "build")
        }
    }
}

@Semantic("AssetCopier")
tasks.register("assembleStaticAssets") {
    dependsOn("compileTypeScript")
    doLast {
        copy {
            from("${projectDir}/src/main/frontend/dist")
            into("${rootProject.buildDir}/resources/main/static/v${project.version}")
        }
    }
}

tasks.named<Test>("test") {
    dependsOn("npmInstall")
    doLast {
        exec {
            workingDir = file("src/main/frontend")
            commandLine("npm", "run", "test:unit")
        }
    }
}
```

### D. Cross-Module Orchestration  
```kotlin
// Root build.gradle.kts (additional orchestrator)
tasks.register("buildFullStackReactive") {
    dependsOn(":backend:compileJava", ":frontend:compileTypeScript")
    finalizedBy(":backend:assemble", ":frontend:assembleStaticAssets", "validateCrossStackSync")
}

@Vibe("Ensures vibe-aligned test pyramid execution")
tasks.register("runFullTests") {
    dependsOn(":backend:test", ":frontend:test")
    finalizedBy("integrationTests")  // TODO: Link to 17-test-pyramid.md
}
```

### E. Build Flow Diagram  
```mermaid
graph TD
    Root[Root: buildFullStackReactive] --> BackendCompile[Backend: compileJava & test]
    Root --> FrontendCompile[Frontend: compileTypeScript & test]
    BackendCompile --> BackendAssemble[Assemble JAR]
    FrontendCompile --> CopyAssets[Copy to /static/v{version}/]
    BackendAssemble --> Validate[validateCrossStackSync]
    CopyAssets --> Validate
    Validate --> Success[Build Success]
    style Root fill:#e1f5fe
    style Validate fill:#fff3e0
```

---

## 📋 Compliance Checklist  
- [ ] All submodules included in `settings.gradle.kts` without circular deps.  
- [ ] Root orchestrator task (`buildFullStackReactive`) depends on compile/test tasks.  
- [ ] Frontend assets copied to versioned `/static/v${version}/` path.  
- [ ] Backend uses WebFlux for reactive endpoints; no blocking I/O.  
- [ ] Cross-stack validation task (`validateCrossStackSync`) implemented and wired.  
- [ ] Task names follow `verbDomainContext` pattern with `@Semantic`/`@Vibe` annotations.  
- [ ] CI/CD pipeline executes full build on push/PR (verify with `--dry-run`).  
- [ ] No direct dependencies between backend and frontend (use `:common` only).  

---

## 🔗 Interlocks with Other Rules  
1. **Folder Structure:** Subprojects align with [03-folder-structure.md](../01-project/03-folder-structure.md).  
2. **Static Resources:** Versioning per [23-static-resources.md](23-static-resources.md).  
3. **Testing Strategy:** Task sequencing matches [17-test-pyramid.md](../05-test-strategy/17-test-pyramid.md).  
4. **Annotations:** `@Semantic`/`@Vibe` follow [12-annotations.md](../03-coding-standards/12-annotations.md).  
5. **Naming:** Artifacts comply with [24-artifact-naming.md](24-artifact-naming.md).  
6. **Diagrams:** Mermaid usage per [14-mermaid-diagrams.md](../04-documentation/14-mermaid-diagrams.md).  

---

## 📜 Revision History  
| Version | Date       | Author        | Changes Summary                                                 |  
|---------|------------|---------------|-----------------------------------------------------------------|  
| 1.0     | 2026-01-25 | LLM-Generated | Initial creation: Combined semantics, CI/CD, and reactive focus |  

---

## Implementation Guide  
1. **Setup Basics:**  
   - Replace all `TODO` placeholders with project-specific values (e.g., domain, versions).  
   - Run `./gradlew tasks --all` to list and verify new tasks.  

2. **Local Validation:**  
   ```bash
   ./gradlew clean
   ./gradlew validateCrossStackSync  # Implement script if needed
   ./gradlew buildFullStackReactive --dry-run  # Check task graph
   ./gradlew buildFullStackReactive  # Full build
   ```  

3. **Customization Steps:**  
   - Add domain-specific submodules (e.g., `:backend:payments`) per [04-modular-architecture.md](../02-architecture/04-modular-architecture.md).  
   - Integrate DTO validation tool (e.g., OpenAPI or custom JSON schema checker).  
   - Configure versioning strategy in `gradle.properties` (e.g., dynamic via CI).  
   - For frontend: Ensure `package.json` in `src/main/frontend` defines `build` and `test:unit` scripts.  

4. **Key Differentiators:**  
   - **Vibe-Infused Builds:** Semantic annotations enable AI-assisted debugging and tracing.  
   - **Reactive-First:** Prioritizes non-blocking tasks for WebFlux/Vue harmony.  
   - **Holistic Orchestration:** Combines compilation, testing, and validation in one flow.  
   - **Scalability:** Supports easy addition of modules without breaking existing builds.  

This configuration ensures a robust, maintainable build pipeline tailored for reactive full-stack applications. For questions, reference the owner or interlocked rules.