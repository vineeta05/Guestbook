# Guestbook App on OpenShift

A cloud-native guestbook project built as part of an internship/school assignment. The app demonstrates a multi-service deployment on Red Hat OpenShift using a frontend, backend API, database, and cache.

## Tech Stack
- Frontend: NGINX + static HTML
- Backend: Go (Gorilla Mux)
- Database: PostgreSQL
- Cache: Redis
- Platform: Red Hat OpenShift
- CI/CD: GitHub Actions + GHCR

## Architecture
- `frontend` service (public via OpenShift Route)
- `backend` service (internal API)
- `postgresql` service (persistent data)
- `redis` service (cache)

## Repository Layout
```text
.
├── backend/
├── frontend/
├── openshift/
│   ├── apps/
│   └── infrastructure/
└── .github/workflows/
```

## Deployment (OpenShift)
1. Deploy infrastructure manifests from `openshift/infrastructure/`.
2. Deploy application manifests from `openshift/apps/`.
3. Open the frontend route and test entry creation.

## CI/CD
- `.github/workflows/infra-deployment.yml`: deploys infrastructure
- `.github/workflows/apps-deployment.yml`: builds/pushes images and deploys apps

Required GitHub secrets:
- `GHCR_USERNAME`
- `GHCR_TOKEN`
- `OPENSHIFT_URL`
- `OPENSHIFT_PASSWORD`

## Full Project Report
For the detailed academic report (introduction, implementation, results, challenges, reflection, and conclusion), see [REPORT.md](REPORT.md).
