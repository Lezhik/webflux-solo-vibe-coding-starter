# Kilo Code Starter Project - Reactive Full-Stack Monorepo (Java/Spring Boot WebFlux/JTE + Vue/TypeScript)

![Kilo Code Architecture](https://via.placeholder.com/800x200?text=Reactive+Full-Stack+Architecture+with+AI-Assisted+Workflows)

## 🚀 Overview
This starter project provides a modular monorepo foundation for building reactive full-stack applications, emphasizing task-driven development with LLM-assisted blueprints. It's tailored for domains like fraud detection in payments, using non-blocking services and real-time visualizations. The setup enforces semantic clarity through `@Semantic` and `@Vibe` annotations, immutable feature versioning, and seamless cross-stack integration.

Key benefits:
- **Reactive Architecture**: Spring WebFlux for backend streams + Vue 3 reactivity for frontend.
- **AI-Enhanced Workflow**: Kilo Code CLI generates tasks, blueprints, and scaffolding from LLM prompts.
- **Immutable & Modular**: Versioned features (e.g., v1.0) prevent drift; isolated by domain.
- **Automated CI/CD**: GitHub Actions for build/test/deploy, with Docker orchestration.

## 🛠️ Technology Stack
- **Backend**: Java 21+, Spring Boot 3.2, Spring WebFlux, JTE, Reactive MongoDB (via Spring Data Reactive)
- **Frontend**: Vue 3, TypeScript, Composition API, Vite for development
- **Build System**: Gradle with Kotlin DSL (multimodule for backend/frontend/shared)
- **Database/Infra**: MongoDB Atlas (reactive), Docker Compose, GitHub Actions
- **Tools**: Kilo Code CLI (v3.1) for task management and generation; LLM integration for blueprints

## 🧠 Core Principles
1. **Vibe Coding**: Infuse artifacts with `@Semantic` (structural intent) and `@Vibe` (emotional/user-centric) annotations for clarity and maintainability.
2. **Task-Driven Development**: Start with LLM-generated tasks → scaffold features → implement & test → merge immutably.
3. **Reactive First**: All services use Flux/Mono for non-blocking I/O; Vue components react to streams.
4. **Immutable Features**: Post-merge, features are version-locked (e.g., `/features/fraud-velocity/v1.0/`); no edits.
5. **Cross-Stack Validation**: Auto-mirrored DTOs (Java/TS) ensure API contracts; tests validate end-to-end.

## 📂 Project Structure
```
fraud-system/
├── backend/                  # Spring WebFlux modules
│   ├── src/main/java/com/fraud/
│   │   └── services/         # Reactive services (e.g., VelocityCheckService)
│   └── src/main/resources/   # Configs, static assets
├── frontend/                 # Vue 3 app
│   ├── src/
│   │   ├── features/         # Domain-specific components (e.g., RiskHeatmap.vue)
│   │   └── types/            # TS interfaces (mirrored from shared DTOs)
│   └── package.json          # npm dependencies
├── shared/                   # Cross-stack contracts
│   ├── dto/                  # e.g., FraudAlert.java + FraudAlert.ts
│   └── models/               # Common entities
├── features/                 # Versioned, immutable implementations
│   └── fraud-velocity/
│       ├── v1.0/             # Locked after merge
│       │   ├── semantic-diagram.md  # Mermaid + annotations
│       │   ├── dto/          # Domain DTOs
│       │   ├── services/     # Backend logic (Java/Kotlin)
│       │   ├── components/   # Frontend SFCs (Vue)
│       │   └── tests/        # Test profiles (YAML)
│       └── v1.1/             # Future iterations
├── tasks/                    # Task hub
│   └── domains/
│       ├── payment-fraud/
│       │   ├── incomplete.md # Active backlog
│       │   └── completed.md  # Archived tasks
│       └── user-auth/        # Other domains
├── rules/                    # Governance docs (e.g., folder-structure.md)
├── .github/workflows/        # CI/CD pipelines
├── docker/                   # Compose files
├── gradle/                   # Build configs
└── README.md                 # This file
```

## 🚀 Quick Start: Initialization
### Prerequisites
- Java 21+ (OpenJDK recommended)
- Node.js 20+ and npm/yarn
- Docker & Docker Compose
- Git
- Kilo Code CLI: Install via `npm install -g @kilo-code/cli` (or equivalent; assumes v3.1)

### 1. Clone & Setup Repository
```bash
# Clone template or create new
git clone https://github.com/your-org/fraud-system.git fraud-system
cd fraud-system
git checkout -b develop

# Copy rules and tasks from starter to your project
cp -r ../webflux-solo-vibe-coding-starter/rules .
cp -r ../webflux-solo-vibe-coding-starter/tasks .

# Replace all TODO in md files in rules directory
...

# Initial Commit
git add .
git commit -m "Initial: Setup monorepo with Kilo rules"
git push -u origin develop

# Initialize Kilo Code CLI
kilo-cli init \
  --repo-url https://github.com/your-org/fraud-system.git \
  --name "Fraud Detection System" \
  --domain payments \
  --java-version 21 \
  --spring-boot 3.5.10 \
  --vue-version 3.3.0
```

### 2. Generate Structure
```bash
# Create structure
kilo-cli generate-structure 

# Init backend/frontend
kilo-cli init-backend --webflux --mongodb-reactive
kilo-cli init-frontend --vue --typescript --vite

# Configure project specifics (replaces TODOs)
kilo-cli configure-project \
  --domain payment-fraud \
  --owner "Anti-Fraud Team"
```

### 3. Commit Results
```bash
git add .
git commit -m "Base structure for project development"
git push -u origin develop
```

### 4. Local Development
```bash
# Backend (reactive dev server)
./gradlew :backend:bootRun --args='--spring.profiles.active=dev'

# Full build/test
./gradlew build  # Includes frontend via Node plugin
```

## 🧩 Task Workflow: LLM-Assisted Development
### 1. Create a Task
Use Kilo CLI to generate an LLM-powered task blueprint.

```bash
kilo-cli create-task \
  --domain payment-fraud \
  --id FRAUD-001 \
  --title "Real-Time Velocity Checks" \
  --priority high \
  --prompt "Generate reactive blueprint for payment velocity detection using WebFlux windowing (e.g., Flux.window(Duration.ofSeconds(5))), with Vue heatmap visualization. Include @Semantic boundaries (e.g., 'FraudStream') and @Vibe intent (e.g., 'Instant Risk Alert'). Define input schema: {userId: string, amount: number, ip: string}. Output: RiskScoreDto with riskLevel ('LOW'|'HIGH') and actions array."
```

This outputs to `tasks/domains/payment-fraud/incomplete.md`:
```markdown
# FRAUD-001 - Real-Time Velocity Checks
**Priority**: High  
**Description**: Detect high-frequency payments (≥5 TPS) using reactive streams; block if risky.  
**Input Schema**:  
```json
{"userId": "string", "amount": "number", "ip": "string", "timestamp": "ISO string"}
```  
**Output**: RiskScoreDto { riskLevel: 'LOW'|'MEDIUM'|'HIGH', confidence: number, actions: string[] }  
**Constraints**: <100ms latency; handle 1000+ TPS; atomic DB writes.  
**Feature Link**: /features/fraud-velocity/v1.0/  
**LLM Blueprint**: [Mermaid diagram generated here]  
**Test Profiles**: normal.yaml (low velocity), high-frequency.yaml (block trigger).  
```

### 2. Generate Feature Scaffolding
```bash
kilo-cli generate-feature \
  --task FRAUD-001 \
  --version v1.0 \
  --modules backend,frontend \
  --output features/fraud-velocity/v1.0 \
  --java-package com.fraud.detection \
  --vue-dir src/features/fraud
```

This creates the versioned structure with stubs, DTOs, and a semantic diagram.

### 3. Implement & Test
- **Backend Example** (`features/fraud-velocity/v1.0/services/VelocityCheckService.java`):
```java
package com.fraud.detection.services;

import reactor.core.publisher.Flux;
import org.springframework.stereotype.Service;
import java.time.Duration;

@Service
@Semantic("FraudVelocityProcessor")
public class VelocityCheckService {

    @Vibe("Instant stream analysis for real-time blocking")
    public Flux<RiskScoreDto> monitorTransactions(Flux<PaymentEvent> events) {
        return events
            .window(Duration.ofSeconds(5))  // Sliding window for velocity
            .flatMap(window -> window.count()
                .filter(count -> count >= 5)  // Threshold check
                .map(c -> new RiskScoreDto("HIGH", 0.9, List.of("BLOCK_ACCOUNT"))));
    }
}

// DTO (mirrored in TS)
@Semantic("RiskScore")
public record RiskScoreDto(String riskLevel, double confidence, List<String> actions) {}
```

- **Frontend Example** (`features/fraud-velocity/v1.0/components/RiskHeatmap.vue`):
```vue
<template>
  <div class="heatmap" :style="{ backgroundColor: getColor(props.scores[0]?.riskLevel) }">
    <p v-if="props.scores.length">Risk: {{ props.scores[0]?.riskLevel }}</p>
  </div>
</template>

<script setup lang="ts">
import { RiskScore } from '@/types';  // Mirrored DTO

interface Props { scores: RiskScore[]; }
const props = defineProps<Props>();

const getColor = (level: string) => {
  @Vibe("Visual urgency scaling")
  return level === 'HIGH' ? 'red' : level === 'MEDIUM' ? 'orange' : 'green';
};
</script>

<style scoped>
.heatmap { height: 200px; transition: background-color 0.3s; }
</style>
```

- **Test Profile** (`features/fraud-velocity/v1.0/tests/high-frequency.yaml`):
```yaml
description: "Simulate 6 transactions/second from same IP"
inputs:
  - { userId: "user-123", amount: 150.00, ip: "192.168.1.1", timestamp: "2023-01-01T00:00:00Z" }
  - { userId: "user-123", amount: 200.00, ip: "192.168.1.1", timestamp: "2023-01-01T00:00:01Z" }
  # ... add 4 more for threshold
expected:
  riskLevel: "HIGH"
  actions: ["BLOCK_ACCOUNT", "ALERT_USER"]
```

Run tests: `./gradlew :backend:test` (integrates Reactor test helpers).

### 4. Commit & Complete
```bash
git add features/fraud-velocity/ tasks/domains/payment-fraud/
git commit -m "feat(FRAUD-001): Implement velocity checks with reactive streams and heatmap"
git push origin develop

# Create PR to main; on merge:
kilo-cli complete-task --id FRAUD-001 --version v1.0 --archive-to completed.md
```

## 🔄 CI/CD Pipeline
### GitHub Actions (`.github/workflows/cicd.yml`)
```yaml
name: Kilo CI/CD
on: [push, pull_request]
  branches: [develop, main]

jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
      - name: Setup Node 20
        uses: actions/setup-node@v4
        with: { node-version: '20' }
      - name: Build Backend
        run: ./gradlew :backend:build :backend:test
      - name: Build Frontend
        run: cd frontend && npm ci && npm run build --if-present
      - name: Validate Features
        run: ./gradlew validateSemanticAnnotations  # Custom task for @Semantic/@Vibe checks
      - name: Security Scan
        uses: gradle/wrapper-validation-action@v1

  deploy:
    needs: build-test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build & Push Docker
        run: |
          docker build -t ghcr.io/${{ github.repository }}:latest .
          echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin
          docker push ghcr.io/${{ github.repository }}:latest
      - name: Deploy to Server (via SSH)
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            docker pull ghcr.io/${{ github.repository }}:latest
            docker-compose -f docker-compose.prod.yml down
            docker-compose -f docker-compose.prod.yml up -d
```

### Server Setup (Ubuntu 22.04+)
```bash
# Install Docker & Compose
sudo apt update && sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker $USER  # Re-login after

# Production Docker Compose (docker/docker-compose.prod.yml)
```yaml
version: '3.8'
services:
  backend:
    image: ghcr.io/your-org/fraud-system:latest
    ports: ["8080:8080"]
    environment:
      SPRING_PROFILES_ACTIVE: prod
      MONGO_URI: mongodb://mongo:27017/frauddb
    depends_on: [mongo]
  frontend:
    build: ../frontend
    ports: ["80:80"]
    depends_on: [backend]
  mongo:
    image: mongo:7
    ports: ["27017:27017"]
    volumes: ['./data:/data/db']
```

# Run: docker-compose -f docker/docker-compose.prod.yml up -d

# Log Rotation
sudo tee /etc/logrotate.d/kilo-app > /dev/null <<EOF
/var/log/kilo-app/*.log {
  daily
  rotate 14
  compress
  missingok
  delaycompress
  create 0640 root adm
}
EOF
sudo logrotate -f /etc/logrotate.d/kilo-app
```

### Health Checks
- Backend: `curl http://localhost:8080/actuator/health`
- Frontend: `curl http://localhost:80`
- Logs: `docker logs <container-id>`

## 📝 Coding Standards & Enforcement
- **Annotations**: Mandatory `@Semantic` for APIs/classes; `@Vibe` for user-facing logic.
- **DTO Mirroring**: Use tools like OpenAPI or manual sync; validate in CI.
- **Tests**: 80%+ coverage; use YAML profiles for data-driven tests.
- **Anti-Patterns**: No blocking calls (e.g., avoid Thread.sleep); kebab-case slugs; no edits to merged versions.
- **Rules Enforcement**: Pre-commit hooks scan for TODOs, annotations, and structure (see `/rules/` for details like `03-folder-structure.md`).

Example Rules File Snippet (`rules/01-project/03-folder-structure.md` excerpt):
> **Core Directory Tree**: Enforce monorepo layout with tasks/, features/, backend/, etc. Dependencies: Gradle multimodule (rules/06-build/22-gradle-multimodule.md).

## 🔮 Roadmap & Enhancements
1. **AI Refactoring**: LLM-suggested fixes for CI failures or blocking ops.
2. **Advanced Validation**: Auto-DTO sync + contract testing.
3. **Performance Monitoring**: Integrate Micrometer + Grafana for stream tracing.
4. **Error Recovery**: Auto-generate PRs for common issues.
5. **Multi-Domain Scaling**: Easier onboarding for new domains (e.g., user-auth).

*Generated with Kilo Code CLI v3.1 | Empowering Reactive Monorepos with AI-Driven Workflows*  
For support, check `/rules/` or contribute via PRs. Start by adding a task in `tasks/domains/payment-fraud/incomplete.md`!