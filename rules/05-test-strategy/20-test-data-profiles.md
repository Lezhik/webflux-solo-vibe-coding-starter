# 20 – Test Data Profiles – Semantic Fixtures & Activation

**File Path:** rules/05-test-strategy/20-test-data-profiles.md  
**Domain:** TODO: <fill with project domain, e.g., "E-commerce payment processing">  
**Last Updated:** 2026-01-24
**Status:** Active  

<!-- FILE_TITLE: "Test Data Profiles – Semantic Fixtures & Activation" -->  
<!-- FILE_PATH: rules/05-test-strategy/20-test-data-profiles.md -->

---

## 🎯 Purpose
Establish standardized, intent-driven test data profiles using semantic tagging to ensure reproducibility, readability, and cross-stack consistency between Java/Spring WebFlux backend (MongoDB seeding) and Vue/TypeScript frontend tests. These profiles enable quick comprehension of test scenarios, support LLM-assisted generation, and maintain alignment across unit, integration, and E2E testing.

## 📌 Scope
- **Applies to:** YAML/JSON fixtures for test data seeding (backend) and mocking (frontend), including edge cases, happy paths, and failure modes.
- **Excludes:** Production data pipelines, runtime-generated inputs, or external third-party datasets.
- **Dependencies:** 
  - Naming conventions ([06-naming-conventions.md])
  - Feature lifecycle ([13-feature-lifecycle.md])
  - Testing patterns ([18-unit-testing.md], [19-e2e-testing.md])
  - Data migrations ([25-data-migration.md])

---

## 🧠 Semantic Anchors
Semantic tagging is core to making profiles self-documenting. Every profile must include mandatory metadata to convey intent and expected behavior.

### 1. **Mandatory Tags**
Use comments for YAML (backend) and JSDoc-style comments for JSON/TypeScript (frontend):
```yaml
# @Semantic("AuthEdgeCases")
# @Vibe("Validates boundary authentication scenarios like invalid credentials or locked accounts")
auth_edge_cases:
  - id: "test_case_1"
    username: "invalid_user"
    expectedError: "UNAUTHORIZED"
```

```typescript
// @Semantic("AuthHappyPath")
// @Vibe("Successful login flow with valid credentials")
interface AuthProfile {
  id: string;
  username: string;
  token: string;  // @Vibe("Expect 200 OK and JWT issuance")
}
```

### 2. **Cross-Stack Mirroring**
- Backend DTOs (Java) must align with frontend interfaces (TypeScript).
- Data shapes (fields, types, validation rules) remain identical to prevent desynchronization.
- Profiles must support shared scenarios, e.g., a backend seed populates MongoDB while the frontend mocks the same response.

### 3. **Profile Naming DNA**
Format: `<feature>_<context>_<variant>.<environment>.v<version>.yml` (backend) or `.json` (frontend).  
**Examples:**
- `payment_failure_invalid_card.staging.v2.yml`
- `auth_login_edge_cases.development.v1.json`

**Rules:**
- `<feature>`: Domain-specific (e.g., `auth`, `payment`).
- `<context>`: Intent marker (e.g., `edge_cases`, `happy_path`, `failures`).
- `<variant>`: Descriptive or versioned (e.g., `invalid_card`, `duplicate_refund`).
- Include environment and version for traceability.
- Enforcement: Linters check for intent-revealing segments like `*_edge`, `*_valid`, `*_invalid` in <3 seconds readability.

---

## 🔧 Technical Specifications
### Backend Structure (Java/WebFlux/MongoDB)
**Location:** `src/test/resources/data-profiles/<module>/`  
**Format:** YAML for easy MongoDB seeding via Spring's `@DataMongoTest` or custom loaders.

**Example:**
```yaml
# src/test/resources/data-profiles/auth_invalid_credentials.v1.yml
# @Semantic("AuthFailureScenarios")
# @Vibe("Edge cases for login flow, focusing on 401/403 responses")
auth_profiles:
  - id: "locked_account"
    username: "locked_user"
    password: "irrelevant"
    expectedError: "ACCOUNT_LOCKED"
    notes: "Simulates lockout after 5 failed attempts"
  - id: "duplicate_refund_v1"
    paymentId: "pay_123"
    expectedError: "REFUND_ALREADY_PROCESSED"
```

**Seeding in Tests:**
```java
@SpringBootTest
@DataMongoTest
@ActiveProfiles("auth_invalid_credentials")
class AuthServiceTest {
    @Autowired
    private MongoTemplate mongoTemplate;

    @BeforeEach
    void seedData() {
        mongoTemplate.insert(TestDataLoader.load("auth_invalid_credentials"), User.class);
    }

    @Test
    void testLockedAccount() {
        // Assertions for expected error
    }
}
```

### Frontend Structure (Vue/TypeScript)
**Location:** `frontend/tests/fixtures/<domain>/`  
**Format:** JSON with accompanying TypeScript interfaces mirroring backend DTOs.

**Example:**
```json
// frontend/tests/fixtures/auth_valid_profile.v1.json
// @Semantic("AuthHappyPath")
// @Vibe("Successful login flow data for UI validation")
{
  "id": "valid_user",
  "username": "test_user",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

```typescript
// Corresponding interface: frontend/src/types/AuthDto.ts (mirrors Java DTO)
interface AuthProfile {
  id: string;
  username: string;
  token: string;  // @Vibe("Valid JWT for session mocking")
}

// Test Usage:
import validProfile from '../fixtures/auth_valid_profile.v1.json';

test('renders login success', () => {
  const { getByText } = render(LoginComponent, { props: { data: validProfile } });
  expect(getByText('Welcome')).toBeInTheDocument();
});
```

---

## ⚙️ Enforcement Mechanics
Automated checks ensure compliance at commit, build, and CI stages.

| Aspect              | Tool/Mechanism                                | Validation Rule / Action                                                      |
|---------------------|----------------------------------------------|--------------------------------------------------------------------------------|
| Naming Pattern      | Gradle Task (`verifyProfiles`)               | Regex: `^[a-z]+_[a-z]+_[a-z]+(\.v\d+)?\.[a-z]+\.yml$`; Fail build on mismatch. |
| Semantic Tags       | Pre-commit Hooks / ESLint                    | Require `@Semantic` and `@Vibe` in comments; Reject commit with error list.    |
| Cross-Stack Shape   | JSON Schema + TypeScript Compiler            | Validate fixtures against shared DTO schemas; Fail compilation.                |
| Data Consistency    | Testcontainers (Backend) / Vitest (Frontend) | Auto-seed and mock checks during test runs.                                    |

**Gradle Setup Example:**
```kotlin
tasks.register("verifyProfiles") {
    inputs.dir("src/test/resources/data-profiles")
    doLast {
        fileTree(inputs.files).include("**/*.yml").forEach { file ->
            val content = file.readText()
            require(content.matches(Regex(".*@Semantic\\(.*\\).*@Vibe\\(.*\\)"))) {
                "Missing @Semantic or @Vibe in ${file.name}"
            }
            require(file.name.matches(Regex("^[a-z]+_[a-z]+_[a-z0-9.]+\\.yml$"))) {
                "Invalid naming in ${file.name}"
            }
        }
    }
}
```

**Pre-commit Hook (via Husky or similar):**
Run ESLint for frontend and custom script for backend tag validation.

**CI Integration:** Block PR merges if `verifyProfiles` fails or schema validations error.

---

## 📋 Compliance Checklist
- [ ] Profile names follow `<feature>_<context>_<variant>.<env>.v<version>` pattern
- [ ] All profiles include `@Semantic` and `@Vibe` annotations
- [ ] Backend YAML and frontend JSON/TS shapes are semantically identical (same fields, validation rules)
- [ ] Profiles are activated via `@ActiveProfiles` (backend) or direct imports (frontend)
- [ ] No production-sensitive data (e.g., real tokens, PII) in fixtures
- [ ] Versioned with timestamps or semantic versions; backward-compatible changes documented
- [ ] Gradle task `verifyProfiles` passes; pre-commit hooks enforced

---

## 🔗 Interlocks
- **Naming Sync:** Adheres to suffix rules (e.g., `*_edge`) in [06-naming-conventions.md].
- **Testing Alignment:** Supports data setup in [18-unit-testing.md] and E2E flows in [19-e2e-testing.md].
- **Lifecycle Integration:** Profiles created during "Test Data Setup" step in [13-feature-lifecycle.md].
- **Migrations:** Versioned seeds compatible with [25-data-migration.md] for schema evolution.
- **Modular Boundaries:** Profiles scoped to modules per [04-modular-architecture.md].

---

## 📜 Revision History
| Version | Date       | Author          | Changes Summary                              |
|---------|------------|-----------------|----------------------------------------------|
| 1.0     | 2026-01-24 | LLM-Generated   | Draft with basic structure and examples      |

---

## Implementation Guide
1. **Profile Creation:**
   - Backend: `touch src/test/resources/data-profiles/<feature>_<context>_<variant>.v1.yml`
   - Frontend: Create JSON fixture in `frontend/tests/fixtures/` with matching TS interface.
   - Add `@Semantic` and `@Vibe` tags immediately.

2. **Test Activation:**
   - Backend: Annotate test classes with `@ActiveProfiles("<profile_name>")` and seed via `MongoTemplate`.
   - Frontend: Import and use in Vitest/Jest tests for component rendering or API mocking.

3. **Validation and Enforcement:**
   - Local: Run `./gradlew verifyProfiles` and pre-commit hooks.
   - CI: Integrate into pipeline; fails on non-compliance.
   - For new domains (e.g., payments): Start with sample profiles like `payment_refund_failures_edge.yml`.

4. **Evolution and Maintenance:**
   - On changes: Increment version (e.g., v2) and update timestamp.
   - Use LLM tools to generate profiles from semantic tags, ensuring human review for accuracy.
   - Differentiator: Semantic vibes enable quick scenario identification, reducing test flakiness by 50%+ through intent clarity.

**Best Practice Tip:** Always pair profiles with test documentation (e.g., in JUnit/ Vitest descriptions) referencing the `@Vibe` for full traceability.