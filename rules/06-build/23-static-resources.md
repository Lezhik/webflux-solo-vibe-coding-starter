# 23 – Static Resources – Versioned Delivery and Caching

**File Path:** rules/06-build/23-static-resources.md  
**Domain:** TODO: <business-context> (e.g., "Secure e-commerce payment processing with real-time event streaming")
**Last Updated:** 2026-01-25
**Status:** Draft  

<!-- FILE_TITLE: Versioned Static Asset Management -->  
<!-- FILE_PATH: rules/06-build/23-static-resources.md -->

---

## 🎯 Purpose
Establishes standards for delivering static assets (e.g., compiled Vue JS/CSS bundles, images, fonts) via versioned paths in a Spring Boot + WebFlux backend. This ensures effective cache busting, prevents stale resource loading, and maintains clear separation between frontend build outputs and backend serving mechanisms in a reactive, multimodule Gradle setup.

## 📌 Scope
- **Applies to:** All compiled frontend assets (JS/CSS from Vue/Vite, third-party libraries, static images) served through Spring Boot's static resource handlers.
- **Excludes:** Dynamically generated content from controllers, server-side rendered templates, and inline scripts/styles (prohibited per HTML/CSS rules).
- **Dependencies:** 
  - `22-gradle-multimodule.md` (multimodule build sequencing and asset copying)
  - `11-html-css-rules.md` (enforcement against inline assets)
  - `30-semantic-versioning.md` (project version alignment)

---

## 🧠 Semantic Anchors

1. **Versioned Path Principle**
   - Assets must be served exclusively from `/static/v{major.minor}/` (e.g., `/static/v1.2/app.js` or `/static/v1.2/payments/styles.css`).
   - All frontend references (in HTML/templates) use absolute, versioned URLs to enable long-term caching with automatic invalidation on version bumps.
   - Semantic annotation: `@Semantic("VersionedStaticAssets")` on relevant beans or functions.

2. **Asset Organization & Segregation**
   - Structure assets by feature/domain for modularity:
     ```plaintext
     src/main/resources/static/
     ├── v1.2/
     │   ├── payments/
     │   │   ├── app.js
     │   │   └── styles.css
     │   ├── core/
     │   │   └── utils.js
     │   └── assets/
     │       └── logo.png
     └── v1.3/
         └── ...
     ```
   - Vibe Coding: Method/task names reflect intent, e.g., `copyVersionedAssets()`, `serveVersionedStaticResources()`.

3. **Cache & Delivery Intent**
   - Leverage semantic versioning for implicit cache busting: No query params or hashes needed in paths.
   - Anti-Patterns:
     - ❌ Non-versioned paths like `/static/app.js`.
     - ❌ Inline `<script>` or `<style>` tags.
     - ❌ Relative asset references (e.g., `./app.js`).

---

## ⚙️ Enforcement Mechanics

| Rule                     | Tool/Mechanism                    | Action on Failure                  | Example Configuration                                              |
|--------------------------|-----------------------------------|------------------------------------|--------------------------------------------------------------------|
| Versioned Path Compliance| Gradle processResources task      | Fail build if version not injected | `expand(projectVersion: project.version)` in `build.gradle.kts`    |
| Asset Placement & Copy   | Custom Gradle copy task           | Block deployment if paths mismatch | `tasks.register("assembleFrontend") { ... copy { ... } }`          |
| HTML/Template References | ESLint + HTMLHint plugins         | Error/Warn on non-versioned refs   | ESLint rule: `'no-relative-static-path': 'error'`                  |
| Cache Header Validation  | Spring Boot tests (WebTestClient) | Fail test if cache period < 1 year | `WebTestClient` assertions on `/static/v1.2/**` responses          |
| Structure Integrity      | CI script (e.g., grep-based check)| Reject PR if invalid dir structure | `grep -r '/static/' src/ | grep -v '/static/v\d+\.\d+/' && exit 1` |

**CI/CD Integration Example (Gradle Kotlin DSL):**
```kotlin
tasks.register("assembleFrontend") {
    dependsOn(":frontend:build")
    doLast {
        copy {
            from(project(":frontend").file("dist"))
            into(file("src/main/resources/static/v${project.version}"))
        }
    }
}

tasks.processResources {
    filesMatching("**/static/**") {
        expand("projectVersion" to project.version)
    }
}

// Custom validation task
tasks.register("validateStaticAssets") {
    doLast {
        // Shell check or file tree validation
        exec { commandLine("bash", "-c", "grep -r '/static/' frontend/src/ | grep -v '/static/v\\d+\\.\\d+/' && exit 1") }
    }
}
```

---

## 🔧 Technical Specifications

### Backend (Spring Boot + WebFlux)
- **Resource Routing Configuration:**
  ```java
  @Configuration
  @Semantic("VersionedStaticAssets")
  public class StaticResourceConfig {
      @Bean
      public RouterFunction<ServerResponse> staticRoutes() {
          return RouterFunctions.resources(
              "/static/v{version:[0-9.]+}/**", 
              new ClassPathResource("static/")
          );
      }
  }
  ```
- **Cache Control:** Configure long expiry for versioned paths (e.g., 1 year max-age) via `WebFluxConfigurer` or resource handler additions. Example:
  ```java
  @Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
      registry.addResourceHandler("/static/v{version:[0-9.]+}/**")
              .addResourceLocations("classpath:/static/")
              .setCachePeriod(31536000); // 1 year
  }
  ```

### Frontend (Vue/TypeScript + Vite)
- **Build Configuration (vite.config.ts):**
  ```typescript
  import { defineConfig } from 'vite';

  export default defineConfig({
    build: {
      outDir: `dist/v${process.env.VITE_APP_VERSION || '1.0'}` // Dynamically set via env
    }
  });
  ```
- **Asset Referencing in Templates/Components:**
  ```html
  <!-- index.html or Vue template -->
  <script src="/static/v1.2/payments/app.js" defer></script>
  <link rel="stylesheet" href="/static/v1.2/payments/styles.css">
  ```
  ```typescript
  // Helper for dynamic references (optional)
  const assetPath = (file: string, domain?: string) => 
    `/static/v${import.meta.env.VITE_APP_VERSION}/${domain ? `${domain}/` : ''}${file}`;
  ```

### Build Process Flow
1. Frontend build: `./gradlew :frontend:build` → Outputs to `frontend/dist/v{version}/`.
2. Backend assembly: Gradle copies to `backend/src/main/resources/static/v{version}/`.
3. Serving: Boot app exposes via versioned router; tests verify 200 OK and cache headers.
```
plaintext
Frontend Build → dist/v1.2/ → Gradle Copy → backend/src/main/resources/static/v1.2/ → Served as /static/v1.2/**
```

---

## 📋 Compliance Checklist
- [ ] Assets compiled to versioned directories (`v{major.minor}`).
- [ ] All HTML/JS references use absolute versioned paths (e.g., `/static/v1.2/...`).
- [ ] No inline scripts/styles; all external and versioned.
- [ ] Cache headers set (e.g., `Cache-Control: max-age=31536000` for static paths).
- [ ] Build validation passes: `./gradlew validateStaticAssets assembleFrontend`.
- [ ] E2E tests confirm asset loading (per `19-e2e-testing.md`).

---

## 🔗 Interlocks
- **Build Sequencing:** Frontend must build before backend assembly (`22-gradle-multimodule.md`).
- **Version Alignment:** Asset paths sync with project semver (`30-semantic-versioning.md`).
- **Testing:** Include static asset checks in E2E/Cypress flows (`19-e2e-testing.md`).
- **HTML Rules:** No inline assets to avoid security/performance issues (`11-html-css-rules.md`).

**Interlock Matrix:**
| Linked Rule                  | Type             | Verification Step                    |
|------------------------------|------------------|--------------------------------------|
| `22-gradle-multimodule.md`   | Prerequisite     | `:frontend:build` precedes copy task |
| `30-semantic-versioning.md`  | Sync             | Paths use `${project.version}`       |
| `19-e2e-testing.md`          | Post-Deployment  | Test HTTP 200 on `/static/v1.2/**`   |
| `11-html-css-rules.md`       | Reference Style  | ESLint blocks non-versioned refs     |

---

## 📜 Revision History
| Version | Date       | Author          | Changes Summary                        |
|---------|------------|-----------------|----------------------------------------|
| 1.0     | 2026-01-25 | LLM-Generated   | Initial draft combining best practices |

---

## Implementation Guide
1. **Setup Versioning:** Define `version` in `gradle.properties` and propagate to frontend env vars (e.g., `VITE_APP_VERSION=$project.version`).
2. **Configure Builds:** Update `vite.config.ts` for versioned output dir; add Gradle copy task in root `build.gradle.kts`.
3. **Backend Routing:** Implement the `StaticResourceConfig` bean; test with `WebTestClient` for path matching and cache headers.
4. **Validation & Testing:**
   - Local: `./gradlew assembleFrontend bootRun` → Browse `/static/v1.2/app.js` (should serve with cache headers).
   - CI: Integrate `validateStaticAssets` task into PR checks.
5. **Deployment Notes:** On version bump (e.g., 1.2 → 1.3), old `/static/v1.2/` remains cached client-side; no server cleanup needed.
6. **Customization:** Replace TODO placeholders with project-specific details (e.g., domain, team). For dynamic domains, extend paths like `/static/v1.2/{feature}/`.

**Appendix: Quick Validation Script**
```bash
#!/bin/bash
# validate-static.sh
VERSION=$(grep '^version=' gradle.properties | cut -d= -f2)
if [[ ! -d "src/main/resources/static/v$VERSION" ]]; then
  echo "Error: Versioned static dir missing!"
  exit 1
fi
grep -r '/static/' frontend/src/ | grep -v "/static/v$VERSION/" && echo "Warning: Non-versioned refs found!"
```

This guide ensures reliable, performant static asset delivery with minimal maintenance overhead.
