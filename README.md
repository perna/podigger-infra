# Podigger Infrastructure

Deployment manifests and operational configuration for Podigger.

This repository will own Docker Compose, Nginx, backups, environment wiring,
and deployment workflows. Application repositories publish immutable images;
this repository deploys those images without building application source on the
VPS.

## Planned services

- Backend image for Django, Celery worker, and Celery Beat
- Frontend image for Next.js
- PostgreSQL and Redis
- Nginx gateway

## Local full-stack integration

With the three repositories checked out as siblings under the same directory,
start the complete local stack (PostgreSQL, Redis, Django, Celery worker/Beat,
and Next.js) from this repository:

```bash
docker compose -f docker-compose.integration.yml up --build
```

Open `http://localhost:3000`; the API is exposed at `http://localhost:8000`.
If those ports are already in use, override them, for example:

```bash
BACKEND_PORT=18000 FRONTEND_PORT=13000 POSTGRES_PORT=15432 REDIS_PORT=16379 \
  docker compose -f docker-compose.integration.yml up --build
```

The frontend's browser-facing `NEXT_PUBLIC_API_URL` points to the host-published
backend port, while its server routes use `BACKEND_INTERNAL_URL` and Docker's
`backend` service DNS name. To stop the test stack, run
`docker compose -f docker-compose.integration.yml down`; add `-v` only when
intentionally deleting its local PostgreSQL data.

## Staging CI/CD

Pushes to `develop` in the frontend or backend repositories run CI, publish a
SHA-tagged image to GHCR, and request a staging deployment. The infra workflow
updates only `/opt/podigger-staging`; production deployment is not part of this
pipeline.

Configure these values in the infra repository's GitHub `staging` environment:

- Variables: `VPS_HOST`, `VPS_USER`, `VPS_KNOWN_HOSTS`, `GHCR_USERNAME`.
- Secrets: `VPS_SSH_KEY` (staging deploy key) and `GHCR_TOKEN` (read access to
  the private GHCR packages).
- Each application repository needs `INFRA_DISPATCH_TOKEN` with permission to
  dispatch an event to `perna/podigger-infra`.

The Nginx proxy starts after the frontend deploy. Keep the Cloudflare Origin CA
certificate and private key at `/opt/podigger-proxy/certs/cloudflare.crt` and
`/opt/podigger-proxy/certs/cloudflare.key` on the VPS.

Secrets stay in GitHub Environments or on the VPS. They are never committed.
