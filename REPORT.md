# Report: Deploying a Cloud-Native Guestbook Application on Red Hat OpenShift

## 1. Introduction
This repository documents our group project for deploying a distributed, cloud-native Guestbook application on Red Hat OpenShift. The goal was to learn the full workflow of containerized application delivery, not only to make the app run.

During the project, we worked in different schedules and locations. Some team members were sick during parts of the assignment period, so coordination was not always easy. Even with these challenges, we continued contributing and completed the deployment. This process helped us learn practical teamwork in a setup similar to real industry collaboration.

The key learning area was configuration quality: correct YAML structure, valid environment variables, and reliable service-to-service communication.

## 2. Background
The Guestbook is a multi-tier application with clear responsibility separation:

- Frontend: NGINX container serving static UI
- Backend: Go API for guestbook operations
- Database: PostgreSQL for persistent entry storage
- Cache: Redis for performance improvement

The frontend communicates with the backend service. The backend communicates with PostgreSQL and Redis through internal OpenShift services.

## 3. Architecture Overview

### Application Components
- `frontend/`: static UI (`index.html`) and NGINX config (`nginx.conf`)
- `backend/`: Go API (`main.go`, `go.mod`)
- `openshift/apps/`: frontend and backend Deployments
- `openshift/infrastructure/`: PostgreSQL, Redis, PVC, Services, Route

### Runtime Flow
1. User accesses the frontend through OpenShift Route.
2. Frontend sends API requests to backend (`/api/*`).
3. Backend writes and reads entries from PostgreSQL.
4. Backend uses Redis cache for faster reads.

## 4. Method and Implementation

### 4.1 Database and Cache Layer
We deployed PostgreSQL and Redis first because backend startup depends on them.

- PostgreSQL deployment uses a PersistentVolumeClaim (`postgresql-pvc`) for data retention.
- PostgreSQL credentials are read from `postgresql-secret`.
- Redis is configured with password protection from `redis-secret`.
- Both components are exposed internally through ClusterIP services:
  - `postgresql:5432`
  - `redis:6379`

### 4.2 Backend Service
The backend is written in Go and containerized with a multi-stage Docker build:

- Build stage: `golang:1.24-alpine`
- Runtime stage: `alpine:latest`
- Static build: `CGO_ENABLED=0`, `GOOS=linux`

Backend deployment (`openshift/apps/backend.yml`) defines:
- Database and Redis environment variables
- Liveness and readiness probes on `/health`
- Internal port `8080`

API endpoints:
- `GET /health`
- `GET /api/entries`
- `POST /api/entries`
- `GET /api/stats`

### 4.3 Frontend Service
The frontend is packaged with NGINX:

- `frontend/Dockerfile` copies `index.html` and `nginx.conf` into the image
- NGINX serves on port `8080`
- Requests under `/api/` are proxied to backend service

Frontend is exposed externally through OpenShift Route (`guestbook-frontend`).

### 4.4 Service Discovery
OpenShift service DNS names are used for internal communication:

- Backend uses `postgresql` as DB host
- Backend uses `redis` as cache host
- Frontend proxies API requests to `backend`

This removed the need to manage pod IP addresses manually.

### 4.5 YAML-First Deployment
We managed deployment mainly via YAML files instead of only UI operations. This improved our understanding of:

- `metadata` and `spec` structure
- environment variable mappings
- secrets integration
- probes and resource limits
- volume mounts and persistent storage

## 5. CI/CD Workflow
This repository includes GitHub Actions workflows:

- `.github/workflows/infra-deployment.yml`
  - Manually deploys infrastructure manifests to OpenShift.
- `.github/workflows/apps-deployment.yml`
  - Builds frontend and backend images.
  - Pushes images to GHCR.
  - Deploys application manifests to OpenShift.

Required secrets in GitHub:
- `GHCR_USERNAME`
- `GHCR_TOKEN`
- `OPENSHIFT_URL`
- `OPENSHIFT_PASSWORD`

## 6. Deployment Steps

### 6.1 Deploy Infrastructure
Apply manifests in `openshift/infrastructure/`:

- PostgreSQL deployment and service
- Redis deployment and service
- PostgreSQL PVC
- Shared services and frontend route

### 6.2 Deploy Applications
Apply manifests in `openshift/apps/`:

- backend deployment
- frontend deployment

### 6.3 Verify
- Check pod status (`Running`)
- Open frontend route
- Create a guestbook entry
- Verify backend, database, and cache logs

## 7. Results
Final outcome:

- Full multi-tier app deployed on OpenShift
- Persistent database storage configured
- Redis cache integrated
- Frontend route accessible externally
- Internal service communication stable
- End-to-end guestbook flow tested successfully

## 8. Challenges

### 8.1 Team Coordination
Different schedules, remote participation, and sickness periods made quick coordination difficult.

### 8.2 Configuration Errors
YAML indentation and variable mismatches caused pod failures during deployment.

### 8.3 Environment Setup Complexity
Matching all components (Go, NGINX, PostgreSQL, Redis, OpenShift) required repeated testing.

### 8.4 Debugging Effort
We spent significant time reading logs, checking events, and validating service endpoints.

## 9. Reflection
This project improved our practical understanding of cloud-native deployment and DevOps workflows. We learned that small configuration details can break complete systems, and that systematic debugging is essential.

It also improved our teamwork under real constraints such as remote collaboration and limited availability.

## 10. Conclusion
The project successfully demonstrated deployment of a cloud-native Guestbook application on Red Hat OpenShift. We applied multi-stage builds, secret management, persistent storage, service discovery, routing, and debugging in a realistic environment. The experience prepared us better for real-world container and platform operations.
