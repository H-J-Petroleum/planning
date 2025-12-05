# Bet: Deployment Pipeline

**Status**: ![Status: In Progress](https://img.shields.io/badge/status%2Fin--progress-1d76db)
**Cycle**: 2
**Owner**: Ops

<!-- LABEL SYSTEM METADATA -->
![Priority: High](https://img.shields.io/badge/priority%2Fhigh-ff9800)
![Type: Automation](https://img.shields.io/badge/type%2Fautomation-1d76db)
![Scope: Multi Repo](https://img.shields.io/badge/scope%2Fmulti--repo-7057ff)
![Effort: 5](https://img.shields.io/badge/effort%2F5-7bc96f)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

**Origin**: [User Request]
**Goal**: Establish a production deployment pipeline for the Timesheet MVP.

---

## 1. Shaping (Planning)

### Context & Problem
We are building services, but we need a way to ship them to production.
- **Problem:** "No where in our plan are we addressing shipping/hosting."
- **Gap:** We have `docker-compose` files, but no defined pipeline for deploying updates to the production environment.
- **Solution:** Formalize the "Docker Production Stack" and create a deployment script/pipeline.

### Scope
- **In Scope**:
    - **Containerization:** Ensure `timesheet-api`, `approval-api`, and `hjps-frontend` have production-ready Dockerfiles.
    - **Orchestration:** Update `docker-compose.production.yml` in `hjps-ops` to include these services.
    - **Pipeline:** Create a `deploy.sh` (or GitHub Action) that builds, pushes, and restarts services on the host.
    - **Secrets:** Standardize secret management (using the existing `init-docker-secrets.js` pattern).
- **Out of Scope**:
    - Kubernetes (Stick to Docker Compose for now).
    - Zero-downtime deployments (Basic restart is fine for MVP).

### Solution Design
1.  **Registry:** Use GitHub Container Registry (GHCR) or local build-on-host.
2.  **Config:** All config via `.env.production` and Docker Secrets.
3.  **Proxy:** Ensure `api-gateway` (Traefik/Nginx) correctly routes to the new services.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Phase 1: Docker Readiness**
    - [ ] Audit Dockerfiles for all 3 services.
    - [ ] optimize build size (multi-stage builds).
- [ ] **Phase 2: Compose Integration**
    - [ ] Add services to `docker-compose.production.yml`.
    - [ ] Configure networking and volumes.
- [ ] **Phase 3: Deployment Script**
    - [ ] Write `scripts/deploy-timesheet-stack.sh`.
    - [ ] Test deployment on a staging environment (or local simulation).

### Acceptance Criteria
- [ ] `docker compose -f docker-compose.production.yml up` starts all services successfully.
- [ ] Services can communicate with each other.
- [ ] Public endpoints are accessible via the Gateway.

### Files/Repos Affected
- `hjps-ops/docker-compose.production.yml`
- `hjps-ops/scripts/`

### References
- [Existing Docker Compose](../../../../hjps-ops/docker-compose.production.yml)

---

## 3. Review (Cool-Down)

### Verification
- [ ] Successful deployment to Staging/Production.
