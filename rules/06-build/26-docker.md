# 26 – Docker Containerization & Production Deployment

**File Path:** rules/06-build/26-docker.md  
**Domain:** TODO: <business-context> (e.g., "Secure e-commerce payment processing with real-time event streaming")
**Last Updated:** 2026-01-27
**Status:** Active  

<!-- FILE_TITLE: Docker Containerization with Persistent MongoDB & CI/CD -->  
<!-- FILE_PATH: rules/06-build/26-docker.md -->  

---

## 🎯 Purpose  
Establish a standardized, immutable Docker packaging strategy for the Kilo Code project, encompassing the Spring Boot WebFlux backend, Vue frontend, and persistent MongoDB storage. This ensures reproducible builds, data persistence via named volumes, and automated CI/CD pipelines using GitHub Actions for secure, zero-downtime deployments to production.

## 📌 Scope  
- **Applies to:** Production-grade container builds, orchestration with Docker Compose, and Kubernetes-compatible image tagging.  
- **Excludes:** Ephemeral local development setups (use `docker-compose.dev.yml` for those).  
- **Dependencies:**  
  - 22-gradle-multimodule.md (multi-module build sequencing).  
  - 23-static-resources.md (versioned static asset handling).  
  - 24-artifact-naming.md (image and artifact tagging conventions).  

---

## 🧠 Semantic Anchors & Vibe Coding  

### Naming DNA  
- **Containers:** `<project>-app` (e.g., `kilo-app` for stateless reactive service), `<project>-db` (e.g., `kilo-db` for stateful MongoDB).  
- **Volumes:** `<project>-mongo-data` (persistent storage with retention semantics).  
- **Images:** `<registry>/<project>-app:<semver>-<git-sha>` (immutable, traceable artifacts).  

### Annotation Taxonomy  
- **`@Semantic("PersistentLayer")`**: Applied to MongoDB volumes for data durability.  
- **`@Vibe("ImmutableArtifact")`**: Enforces no `latest` tags; all images must include version/SHA.  
- **`@ContainerScope("ReactiveStack")`**: Highlights WebFlux + Vue integration in a single app container.  

### 3-Second Intent Test Examples  
- ✅ `kilo-app:v1.2.3-sha` → Immediately signals versioned Spring Boot + Vue runtime.  
- ✅ `kilo-db` with `kilo-mongo-data` volume → Clear persistent database role.  
- ❌ `app:latest` → Ambiguous and mutable; violates immutability.  

---

## ⚙️ Enforcement Matrix  
| Rule                   | Tool/Mechanism                   | Failure Action                         | Example Configuration                  |  
|------------------------|----------------------------------|----------------------------------------|----------------------------------------|  
| Volume Persistence     | Docker Compose validation        | Reject deploys without named volumes   | `<project>-mongo-data:/data/db`        |  
| Image Immutability     | GitHub Actions + tagging         | Block push if `latest` only            | `ghcr.io/org/kilo-app:v1.2.3-sha`      |  
| Build Order            | Multi-stage Dockerfile + Actions | Fail if backend builds before frontend | Sequential stages in Dockerfile        |  
| Secret Management      | GitHub Secrets + env vars        | Terminate workflow on exposure         | `MONGO_ROOT_PASSWORD` from secrets     |  
| Health Checks          | Docker HEALTHCHECK + probes      | Auto-rollback after 3 failures         | Mongo ping + app actuator/health       |  
| Non-Root Execution     | Dockerfile USER directive        | Fail vulnerability scans               | `USER 10001` in runtime stage          |  
| Asset Integration      | Gradle + npm in multi-stage      | Fail on static resource mismatch       | Copy `/dist` to `/static/v${VERSION}/` |  

---

## 🔧 Technical Specifications  

### 1. Dockerfile (Multi-Stage Build for App Container)  
This creates a single, optimized image for the reactive app, combining backend JAR and frontend assets. No separate DB Dockerfile needed (use official Mongo image).  

```dockerfile  
# Stage 1: Backend builder (Gradle)  
FROM gradle:8-jdk21 AS backend-builder  
WORKDIR /app  
COPY gradle.properties settings.gradle .  
COPY backend ./backend  
COPY shared ./shared  # If multi-module  
RUN gradle :app:bootJar --no-daemon -x test  

# Stage 2: Frontend builder (Node)  
FROM node:18-bullseye-slim AS frontend-builder  
WORKDIR /app/frontend  
COPY frontend/package*.json .  
RUN npm ci --only=production  
COPY frontend .  
RUN npm run build  

# Stage 3: Runtime image (JRE, non-root)  
FROM eclipse-temurin:21-jre-alpine  
WORKDIR /app  

# Copy artifacts  
COPY --from=backend-builder /app/backend/app/build/libs/app-*.jar ./app.jar  
COPY --from=frontend-builder /app/frontend/dist ./static/  

# Security: Non-root user  
RUN addgroup -g 1001 -S appgroup && \  
    adduser -S appuser -u 1001 -G appgroup && \  
    chown -R appuser:appgroup /app  
USER appuser  

EXPOSE 8080  
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \  
  CMD curl -f http://localhost:8080/actuator/health || exit 1  

ENTRYPOINT ["java", "-jar", "app.jar"]  
```  

### 2. docker-compose.prod.yml (Production Orchestration)  
Defines services with health dependencies, persistent volumes, and secret injection. Use NFS or cloud storage driver for volumes in prod.  

```yaml  
version: '3.9'  
services:  
  app:  
    image: ${REGISTRY}/kilo-app:${VERSION}  
    container_name: kilo-app  
    ports:  
      - "80:8080"  
    depends_on:  
      db:  
        condition: service_healthy  
    environment:  
      SPRING_DATA_MONGODB_URI: mongodb://${MONGO_USER}:${MONGO_PASSWORD}@db:27017/kilo?authSource=admin  
      SPRING_PROFILES_ACTIVE: prod  
    restart: unless-stopped  
    healthcheck:  
      test: ["CMD-SHELL", "curl -f http://localhost:8080/actuator/health || exit 1"]  
      interval: 30s  
      timeout: 10s  
      retries: 3  
      start_period: 40s  

  db:  
    image: mongo:7.0  
    container_name: kilo-db  
    environment:  
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_USER}  
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD}  
    volumes:  
      - kilo-mongo-data:/data/db  
    ports:  
      - "27017:27017"  # Optional for external access  
    restart: unless-stopped  
    healthcheck:  
      test: ["CMD-SHELL", "echo 'db.runCommand(\"ping\").ok' | mongosh localhost:27017/test --quiet"]  
      interval: 10s  
      timeout: 5s  
      retries: 5  
      start_period: 20s  

volumes:  
  kilo-mongo-data:  
    driver: local  # Or 'nfs' with opts: type=nfs,o=addr=your-nfs-server,rw for prod  

# Secrets (injected via .env or GitHub)  
# No hardcoded values; use .env.prod for local testing  
```  

**Notes on Volumes:** The `<project>-mongo-data` volume ensures data persistence across restarts/deploys. In cloud (e.g., Kubernetes), map to PersistentVolumeClaims. Validate with `docker volume inspect <project>-mongo-data` to confirm retention.

### 3. GitHub Actions Workflow (CI/CD Pipeline)  
Triggers on main pushes/PRs; builds, tests, pushes images, and deploys via SSH. Includes vulnerability scanning via Trivy (optional add-on).  

```yaml  
name: Docker Build, Test & Deploy  

on:  
  push:  
    branches: [ main ]  
  pull_request:  
    branches: [ main ]  

env:  
  REGISTRY: ghcr.io/${{ github.repository_owner }}  
  IMAGE_NAME: kilo-app  
  VERSION: ${{ github.sha }}  # Fallback to semver via git describe in steps  

jobs:  
  build-test:  
    runs-on: ubuntu-latest  
    steps:  
      - name: Checkout  
        uses: actions/checkout@v4  
        with:  
          fetch-depth: 0  # For git describe  

      - name: Set version  
        id: version  
        run: |  
          if [[ $GITHUB_REF == 'refs/heads/main' ]]; then  
            VERSION=$(git describe --tags --abbrev=0 2>/dev/null || echo "${{ github.sha }}")  
          else  
            VERSION="dev-${{ github.sha }}"  
          fi  
          echo "VERSION=$VERSION" >> $GITHUB_ENV  

      - name: Set up JDK 21  
        uses: actions/setup-java@v4  
        with:  
          distribution: temurin  
          java-version: 21  
          cache: gradle  

      - name: Build backend  
        run: ./gradlew clean build -x test  

      - name: Build frontend  
        working-directory: ./frontend  
        run: |  
          npm ci  
          npm run build  
          npm run test  

      - name: Log in to GHCR  
        uses: docker/login-action@v3  
        with:  
          registry: ${{ env.REGISTRY }}  
          username: ${{ github.actor }}  
          password: ${{ secrets.GITHUB_TOKEN }}  

      - name: Build & push app image  
        uses: docker/build-push-action@v5  
        with:  
          context: .  
          file: Dockerfile  
          push: ${{ github.ref == 'refs/heads/main' }}  
          tags: |  
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ env.VERSION }}  
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest  # Prod builds only  
          build-args: |  
            VERSION=${{ env.VERSION }}  
          cache-from: type=gha  
          cache-to: type=gha,mode=max  

      - name: Scan image for vulnerabilities (optional)  
        if: success()  
        uses: aquasecurity/trivy-action@master  
        with:  
          image-ref: '${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ env.VERSION }}'  
          format: 'sarif'  
          output: 'trivy-results.sarif'  

  deploy:  
    needs: build-test  
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'  
    runs-on: ubuntu-latest  
    steps:  
      - name: Deploy to production  
        uses: appleboy/ssh-action@v1.0.3  
        env:  
          VERSION: ${{ env.VERSION }}  
        with:  
          host: ${{ secrets.PROD_HOST }}  
          username: ${{ secrets.DEPLOY_USER }}  
          key: ${{ secrets.DEPLOY_SSH_KEY }}  
          script: |  
            set -e  
            cd /opt/kilo-code  
            echo "REGISTRY=${{ env.REGISTRY }}" > .env  
            echo "VERSION=${VERSION}" >> .env  
            echo "MONGO_USER=${{ secrets.MONGO_USER }}" >> .env  
            echo "MONGO_PASSWORD=${{ secrets.MONGO_PASSWORD }}" >> .env  
            docker compose -f docker-compose.prod.yml pull  
            docker compose -f docker-compose.prod.yml up -d  
            docker system prune -f  
            docker compose -f docker-compose.prod.yml ps  # Log status  
```  

### 4. Secret Requirements (GitHub Secrets)  
- `PROD_HOST`: Production server IP/hostname.  
- `DEPLOY_USER`: SSH username (e.g., `deploy`).  
- `DEPLOY_SSH_KEY`: Private SSH key for authentication.  
- `MONGO_USER`: MongoDB root username (default: `root`).  
- `MONGO_PASSWORD`: MongoDB root password (rotate regularly).  
- `GITHUB_TOKEN`: Auto-provided for GHCR login.  

**Security Note:** Never commit `.env` files. Use ephemeral env injection in Actions; for compose, load from secure sources.

---

## 📋 Compliance Checklist  
- [ ] Multi-stage Dockerfile builds backend (Gradle) and frontend (npm) sequentially.  
- [ ] App image copies Vue `/dist` to `/static/` for serving.  
- [ ] MongoDB uses named volume `kilo-mongo-data` with optional NFS driver.  
- [ ] Health checks defined for both services; app depends on DB health.  
- [ ] Images tagged immutably (semver + SHA); no `latest` in non-prod.  
- [ ] CI/CD workflow tests builds, pushes to GHCR, and deploys via SSH.  
- [ ] Non-root user (UID 1001) in runtime image.  
- [ ] Secrets injected via GitHub Secrets; no hardcoding.  
- [ ] Volume persistence verified post-deploy (e.g., data survives restarts).  
- [ ] Static assets versioned per rule 23 (e.g., `/static/v1.2.3/`).  

---

## 🔗 Interlocks with Other Rules  
- **Build Sequencing:** Relies on 22-gradle-multimodule.md for JAR production.  
- **Artifact Naming:** Image tags align with 24-artifact-naming.md (e.g., `<semver>-<sha>`).  
- **Static Resources:** Frontend build outputs to versioned paths in 23-static-resources.md.  
- **Data Layer:** Mongo init scripts follow data migration patterns in related rules (e.g., 25-data-migration.md).  

---

## 📜 Revision History  
| Version | Date       | Author             | Changes Summary                           |  
|---------|------------|--------------------|-------------------------------------------|  
| 1.0     | 2026-01-27 | LLM-Generated      | Initial MongoDB persistence and CI basics |  

---

## Implementation Guide  
1. **Local Setup:**  
   - Create `.env` from `.env.example` with test secrets.  
   - Run `docker compose -f docker-compose.prod.yml up -d` and verify: `docker compose logs -f`.  
   - Test persistence: Insert data, restart, query via mongosh.  

2. **CI/CD Activation:**  
   - Add secrets to GitHub repo settings.  
   - Push to main to trigger build/deploy. Monitor in Actions tab.  

3. **Production Deployment:**  
   - Ensure server has Docker Compose installed.  
   - Use `./gradlew dockerPrepare` (custom task) pre-CI for local image builds.  
   - Monitor: `docker compose ps` and app logs for health.  

4. **Troubleshooting:**  
   - Build failures: Check Gradle/npm caches in Actions.  
   - Deploy issues: SSH debug with `ssh -v user@host`.  
   - Volume growth: Prune unused with `docker volume prune`; monitor with `df -h`.  

**Differentiators:**  
- Single app container for simplicity (WebFlux serves Vue assets reactively).  
- Resilient startup (DB health gates app).  
- Secure & auditable (immutable tags, scanned images, secret rotation).  
This blueprint upholds Kilo Code's reactive, immutable infrastructure principles while enabling scalable deployments. For Kubernetes migration, adapt compose to Helm manifests.