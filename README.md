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

Secrets stay in GitHub Environments or on the VPS. They are never committed.
