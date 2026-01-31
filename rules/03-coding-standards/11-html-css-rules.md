# 11 - HTML/CSS Rules - JTE Server-Side Rendering + Vue Hydration

**File Path:** rules/03-coding-standards/11-html-css-rules.md  
**Domain:** Server-Side Rendering (JTE) + Client-Side Hydration (Vue)  
**Last Updated:** 2026-01-23
**Status:** Active  

<!-- FILE_TITLE: HTML/CSS Standards for JTE + Vue Hydration -->  
<!-- FILE_PATH: rules/03-coding-standards/11-html-css-rules.md -->

---

## 🎯 Purpose
Establish unified standards for HTML/CSS implementation in server-generated JTE templates and Vue-powered client interactions, ensuring:
- Semantic server-rendered base structure with accessibility and security (e.g., CSRF, CSP)
- Safe hydration boundaries for Vue components to prevent flickering or conflicts
- Version-controlled static assets for caching and maintainability
- Cross-stack compatibility with LLM-assisted development and vibe annotations
- Strict separation of server (JTE) and client (Vue) responsibilities for progressive enhancement

## 📌 Scope
- **Applies to:** 
  - JTE templates in `/src/main/resources/templates/`
  - Vue Single File Components (SFCs) in `/src/main/resources/static/js/`
  - Global CSS/SCSS in versioned static directories (`/src/main/resources/static/vX/css/`)
  - Data exchange via DTOs between Java controllers and Vue props
- **Excludes:** 
  - Legacy templates (Thymeleaf/JSP)
  - Inline CSS/JS outside nonce-protected CSP exceptions
- **Dependencies:** 
  - [10-typescript-style.md] - Vue component and TypeScript conventions
  - [23-static-resources.md] - Asset versioning and bundling strategy
  - [09-java-style.md] - DTO patterns for JTE-Vue data exchange

---

## 🧠 Architectural Imperatives

### 1. JTE Template Standards (Server-Side)
**Semantic Foundations and Security:**
- Prioritize semantic HTML5 elements for accessibility (e.g., `<header>`, `<main role="main">`).
- Include CSRF protection and CSP headers.
- Use Gradle-expanded variables for versioned assets (e.g., `${staticVersion}`).
- Avoid Vue directives (e.g., `v-if`, `v-for`) in JTE to prevent hydration mismatches.

**Example JTE Template:**
```html
@param com.example.dto.PageShell shell
<!DOCTYPE html>
<html lang="${shell.locale}" 
      data-vibe="server-rendered-container">
<head>
    <meta charset="UTF-8">
    <meta name="csrf-token" content="${_csrf.token}">
    <meta name="_csrf_header" content="${_csrf.headerName}">
    <meta http-equiv="Content-Security-Policy" 
          content="default-src 'self'; script-src 'self' 'nonce-${shell.nonce}'">
    <title>${shell.title}</title>
    <!-- Versioned global styles -->
    <link rel="stylesheet" 
          href="/static/${staticVersion}/css/core.css">
</head>
<body>
    <header aria-labelledby="main-header">
        <h1 id="main-header">${shell.headerText}</h1>
    </header>

    <!-- Vue hydration anchor with initial state -->
    <main id="vue-root" 
          role="main"
          data-initial-state='${shell.toJson()}'
          data-vibe="client-hydration-point">
        <!-- Progressive enhancement: skeleton for no-JS fallback -->
        <div class="skeleton-loader"></div>
        <noscript>
            <div class="nojs-warning">
                Enable JavaScript for interactive features
            </div>
        </noscript>
    </main>

    <!-- Non-blocking, nonce-protected scripts -->
    <script src="/static/${staticVersion}/js/chunk-vendors.js" 
            nonce="${shell.nonce}"
            defer></script>
    <script type="module" src="/static/${staticVersion}/js/app.js" 
            nonce="${shell.nonce}"
            defer></script>
</body>
</html>
```

**Prohibitions:**
- No inline styles/scripts (except CSP-nonce exceptions).
- No direct DOM manipulation or Vue logic in server templates.
- Ensure hydration container has a unique ID (e.g., `#vue-root`) for safe mounting.

### 2. Vue Hydration Protocol (Client-Side)
**Mounting and Data Flow:**
- Mount Vue exclusively to the JTE-defined container (e.g., `#vue-root`).
- Read initial state from `data-initial-state` attribute to hydrate without re-rendering server HTML.
- Use TypeScript props for DTO alignment; validate with JSON Schema.

**Example Vue Entry Point (main.ts):**
```typescript
// /src/main/resources/static/vX/js/main.ts
import { createApp } from 'vue';
import App from './App.vue';

const mountElement = document.getElementById('vue-root');
if (mountElement?.dataset.initialState) {
    const initialState = JSON.parse(mountElement.dataset.initialState);
    const app = createApp(App, {
        initialState: initialState as PageShellDto // Align with Java DTO
    });
    app.mount('#vue-root');
}
```

**Vue SFC Structure Example:**
```vue
<template>
  <div class="data-grid" 
       role="grid"
       aria-labelledby="grid-header"
       data-vibe="reactive-data-visualization">
    <header class="data-grid__header">
      <h2 id="grid-header" v-text="gridTitle"></h2>
    </header>
    <slot :items="processedItems" />
  </div>
</template>

<style scoped lang="scss">
.data-grid {
  --grid-gap: 1rem;
  display: grid;
  gap: var(--grid-gap);
  transition: opacity var(--transition-duration, 250ms);
  
  &__header {
    @apply bg-neutral-100; /* If using Tailwind */
  }
}
</style>

<script setup lang="ts">
/** @Vibe "Reactive data visualization grid" */
import type { PurchaseOrderDto } from '@/types'; // Align with Java DTO

defineProps<{
  initialState: PageShellDto;
}>();

const processedItems = ref<PurchaseOrderDto[]>([]);
</script>
```

**Hydration Safety:**
- Avoid global DOM changes on mount to prevent FOUC (Flash of Unstyled Content).
- Use lifecycle hooks like `onMounted` for post-hydration updates.
- Test with Vue Test Utils to enforce mounting boundaries.

### 3. CSS Layering and Isolation Strategy
Separate layers to prevent leakage between server and client:

| Layer            | Location                       | Scope              | Technique                   | Example                                      |
|------------------|--------------------------------|--------------------|-----------------------------|----------------------------------------------|
| **Reset/Global** | `/static/vX/css/core.css`      | Global (body-wide) | Normalize.css + CSS vars    | `:root { --color-primary: #2c3e50; }`        |
| **Semantic**     | JTE `<style>` (rare) or global | Server-scoped      | Attribute selectors         | `[data-vibe="server-rendered-container"] {}` |
| **Component**    | Vue SFC `<style scoped>`       | Local to component | Scoped + BEM                | `.data-grid__item {}`                        |
| **Utility**      | `/static/vX/css/utilities.css` | Global utilities   | Tailwind (purged) or custom | `.margin-auto { margin: 0 auto; }`           |

**Standards:**
- Prefer BEM naming for globals; scoped styles in Vue.
- Use SCSS variables/CSS custom properties for theming.
- Anti-patterns: Overly generic selectors (e.g., `.container div`); use semantic/precise ones instead.
- Version assets to enable cache busting.

---

## ⚙️ Enforcement Matrix

| Rule                            | Tools                             | CI Action           | Configuration Snippet                              |
|---------------------------------|-----------------------------------|---------------------|----------------------------------------------------|
| JTE Hygiene (No Vue Directives) | jte-lint + HTMLHint               | Block PR            | `disallowVueDirectives: true`                      |
| CSS Scoping & Isolation         | Stylelint + Vite/Vue-loader       | Fail build          | `"scoped-only": true; "no-global-selectors": warn` |
| Asset Versioning & Paths        | Gradle Regex + ESLint             | Reject PR           | `versionPattern: /static\/v\d+\//`                 |
| Hydration Safety                | Vue Test Utils + Jest             | Warning on mismatch | `allowedRootIds: ['vue-root']`                     |
| Accessibility (a11y)            | Axe Core + Cypress                | Grade (≥95%)        | `impactLevel: 'critical'; threshold: 95`           |
| Security (CSRF/CSP)             | Spring Security Audit + OWASP ZAP | Block on failure    | `requireCsrfMeta: true`                            |
| HTML Validity                   | Nu HTML Validator                 | Zero errors         | Integrated in Gradle `validateHtml` task           |

**Gradle Integration:**
```kotlin
plugins {
    id("gg.jte.gradle") version "3.1.0"
}

tasks.named<ProcessResources>("processResources") {
    filesMatching("**/*.jte") {
        expand(
            "staticVersion" to project.property("static.version") ?: "v1",
            "csrfEnabled" to true
        )
    }
}

tasks.named("jtePrecompile") {
    inputs.property("staticVersion", project.version)
    outputs.dir(layout.buildDirectory.dir("generated-sources/jte"))
    
    doFirst {
        rootProject.extensions.configure<JavaPluginExtension> {
            sourceCompatibility = JavaVersion.VERSION_21
        }
    }
}
```

**Additional Tooling:**
- ESLint plugin for inline detection.
- Stylelint for CSS anti-patterns.
- Axe in CI for server-rendered HTML.

---

## 📋 Compliance Verification Checklist
- [ ] JTE templates include `<meta name="csrf-token">` and CSP headers
- [ ] All scripts/styles use versioned paths (`/static/${staticVersion}/...`)
- [ ] Vue mounts only to `#vue-root` (or equivalent JTE-defined ID)
- [ ] Zero Vue directives (`v-*`) in `.jte` files
- [ ] Initial state passed via `data-initial-state` (JSON Schema validated)
- [ ] All Vue CSS uses `<style scoped>`; globals follow BEM
- [ ] Axe accessibility score ≥ 95% on key server/client renders
- [ ] No inline JS/CSS; all assets nonce-protected where applicable
- [ ] Progressive enhancement: `<noscript>` fallback present

---

## 🔗 Cross-Stack Integration
- **DTO Alignment:** Java `PageShell` DTO serializes to JSON for Vue `initialState` prop; use TypeScript interfaces matching Java types.
- **Build Sequencing:** JTE precompile → TypeScript/Vue build → Static asset bundling (Gradle orchestrates).
- **Security:** Nonce-protected scripts via Spring Security CSP; CSRF tokens in forms/meta.
- **Analytics & Vibe:** Preserve `data-vibe` attributes in production for LLM/tooling analysis.
- **Interlocks:** 
  - [22-gradle-multimodule.md]: Ensures build order for assets.
  - [10-typescript-style.md]: Prop validation aligns with DTOs.

---

## 📜 Revision History

| Version | Date       | Author        | Changes Summary                          |  
|---------|------------|---------------|------------------------------------------|  
| 1.0     | 2026-01-23 | LLM-Generated | Initial spec for hybrid enhancement arch |  

---

## Implementation Roadmap

1. **Toolchain Setup**
   ```bash
   ./gradlew addJteTemplatesTask  # Enables JTE plugin
   npm install eslint-plugin-vue-hydration stylelint-config-standard-scss axe-core
   ```

2. **Hydration Safety Configuration**
   ```typescript
   // jest-vue.config.ts
   export default {
       testEnvironment: 'jsdom',
       globals: {
           __VUE_HYDRATION__: JSON.stringify({
               allowedRootIds: ['vue-root']
           })
       }
   };
   ```

3. **Progressive Enhancement & Validation**
   - Add skeleton loaders in JTE for initial render.
   - Run CI commands:
     ```bash
     ./gradlew validateHtml processResources  # Checks templates and expands vars
     npm run lint:css test:hydration          # Stylelint + Vue tests
     npm run accessibility:ci                 # Axe on rendered HTML
     ```

**Key Differentiators:**
- Strict JTE/Vue separation with nonce-based CSP for security.
- Gradle-managed versioning and precompilation for reliable builds.
- Double-layer a11y checks (server HTML + client hydration).
- Schema-validated data flow prevents hydration mismatches.
- Vibe annotations (`data-vibe`, `@Vibe`) for enhanced tooling/LLM compatibility.

This consolidated rule maintains original numbering, integrates best practices from prior versions, and adds refinements for security (nonce/CSRF), testing (Vue Utils/Axe), and build reliability (Gradle tasks). It is optimized for the Kilo Code project's Spring Boot + Vue stack.