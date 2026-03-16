# Container & Docker Security Rules

**Project:** VibeShield
**Stack:** Docker & Container Security
**Version:** 0.1.0

## Overview

This document defines imperative security rules for containerized applications using Docker. AI code generators frequently introduce insecure Dockerfile patterns, expose secrets, and violate container security best practices.

---

## Base Image Security

### Image Tags

Never use `latest` tag for base images. Tags are mutable and create unpredictable builds. [V-02]

Always use specific version tags with explicit variants. [V-02]

Always specify the full tag including version and variant (`node:18.20.2-alpine3.19`, not `node:18`). [V-02]

Never use unversioned or ambiguous tags in production Dockerfiles. [V-02]

NEVER:
```dockerfile
FROM node:latest
FROM python:3
FROM ubuntu
```

ALWAYS:
```dockerfile
FROM node:18.20.2-alpine3.19
FROM python:3.11.8-slim-bookworm
FROM ubuntu:22.04
```

### Minimal Images

Always prefer minimal base images to reduce attack surface. [V-02]

Always use `-slim`, `-alpine`, or `distroless` variants when available. [V-02]

Never use full OS images (debian, ubuntu, centos) unless absolutely necessary. [V-02]

Prefer distroless images for production when possible. [V-02]

NEVER: `FROM node:18` (includes full Debian, build tools, package manager) [V-02]

ALWAYS: `FROM node:18.20.2-alpine3.19` or `FROM gcr.io/distroless/nodejs18-debian12` [V-02]

### Image Provenance

Always pull images from official registries (Docker Hub official images, gcr.io, ECR). [V-12]

Always verify image checksums for critical base images. [V-12]

Never use unverified third-party images in production. [V-12]

---

## User & Permissions

### Non-Root User

Never run application processes as root. [V-02]

Always create a dedicated non-root user and switch to it before CMD/ENTRYPOINT. [V-02]

Always run application processes as a non-root user with minimal privileges. [V-02]

NEVER:
```dockerfile
FROM node:18-alpine
COPY . /app
CMD ["node", "server.js"]
# Runs as root (UID 0)
```

ALWAYS for Alpine:
```dockerfile
FROM node:18.20.2-alpine3.19

# Create non-root user
RUN addgroup -S app && adduser -S app -G app

WORKDIR /app
COPY --chown=app:app . .

USER app
CMD ["node", "server.js"]
```

ALWAYS for Debian/Ubuntu:
```dockerfile
FROM node:18.20.2-slim-bookworm

# Create non-root user
RUN groupadd -r app && useradd -r -g app app

WORKDIR /app
COPY --chown=app:app . .

USER app
CMD ["node", "server.js"]
```

### File Permissions

Always set proper ownership with `COPY --chown=user:group`. [V-02]

Always ensure the non-root user has only necessary file permissions. [V-02]

Never set world-writable permissions (777, 666) on application files. [V-02]

---

## Multi-Stage Builds

### Build Separation

Always use multi-stage builds to exclude build tools from production images. [V-02]

Always separate build dependencies from runtime dependencies. [V-02]

Never include compilers, development headers, or build tools in final production images. [V-02]

Never include source code, tests, or development files in production images. [V-02]

NEVER (single-stage with build tools in production):
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "server.js"]
# Includes node_modules with dev dependencies
```

ALWAYS (multi-stage):
```dockerfile
# Build stage
FROM node:18.20.2-alpine3.19 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:18.20.2-alpine3.19
RUN addgroup -S app && adduser -S app -G app
WORKDIR /app
COPY --from=builder --chown=app:app /app/node_modules ./node_modules
COPY --chown=app:app . .
USER app
CMD ["node", "server.js"]
```

ALWAYS for compiled languages (Go, Rust):
```dockerfile
# Build stage
FROM golang:1.22.1-alpine AS builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/server

# Production stage
FROM gcr.io/distroless/static-debian12
COPY --from=builder /app/server /server
USER nonroot:nonroot
ENTRYPOINT ["/server"]
```

---

## Secrets Management

### Secret Exposure

Never use `ARG` or `ENV` for secrets. They persist in image layers and are visible in `docker history`. [V-02, V-09]

Never copy `.env` files into images. [V-02, V-09]

Never hardcode passwords, API keys, tokens, or credentials in Dockerfiles. [V-02, V-09]

Always use Docker secrets, runtime environment variables, or secret management systems. [V-02]

NEVER:
```dockerfile
ENV DATABASE_PASSWORD=mypassword
ARG API_KEY=sk-1234567890
COPY .env /app/.env
RUN echo "SECRET_KEY=abc123" > /app/config
```

ALWAYS use runtime secrets:
```dockerfile
# No secrets in Dockerfile
# Pass at runtime:
# docker run -e DATABASE_PASSWORD="$DB_PASS" myapp
# Or use Docker secrets (Swarm/Kubernetes)
```

### BuildKit Secret Mounts

Always use BuildKit secret mounts for secrets needed during build (private registry auth, SSH keys). [V-02]

Never copy secrets into intermediate layers. [V-02]

ALWAYS with BuildKit:
```dockerfile
# syntax=docker/dockerfile:1.4
FROM node:18.20.2-alpine3.19

# Mount secret during build, doesn't persist in image
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm ci --only=production
```

Build with: `docker build --secret id=npmrc,src=.npmrc .` [V-02]

---

## File Operations

### Dockerignore

Always create a `.dockerignore` file. [V-09, V-16]

Always exclude `.env`, `.git`, `node_modules`, `__pycache__`, `.pytest_cache`, and sensitive files. [V-09, V-16]

Never use `COPY . .` without a comprehensive `.dockerignore`. [V-09, V-16]

ALWAYS create `.dockerignore`:
```
.env
.env.*
.git
.gitignore
*.md
node_modules
__pycache__
*.pyc
.pytest_cache
.coverage
.vscode
.idea
*.log
secrets/
*.key
*.pem
```

### Specific Copy Instructions

Always use specific `COPY` instructions for required files only. [V-16]

Never copy entire directories without filtering via `.dockerignore`. [V-16]

Prefer explicit file lists over glob patterns where practical. [V-16]

NEVER: `COPY . /app` without `.dockerignore` [V-16]

ALWAYS: Combine specific COPY with `.dockerignore`:
```dockerfile
COPY package*.json ./
COPY src/ ./src/
COPY public/ ./public/
# .dockerignore prevents .env and .git from being copied
```

---

## Dependency Management

### Version Pinning

Always pin dependency versions in package manifests. [V-12]

Never use `^` or `~` version ranges in production dependency files. [V-12]

Always generate and commit lock files (`package-lock.json`, `poetry.lock`, `Gemfile.lock`). [V-12]

ALWAYS:
```dockerfile
# Node.js - use npm ci with package-lock.json
COPY package*.json ./
RUN npm ci --only=production

# Python - use pinned requirements
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Ruby - use bundle install with Gemfile.lock
COPY Gemfile Gemfile.lock ./
RUN bundle install --deployment --without development test
```

### Cache Optimization

Never use `pip install --no-cache-dir` as a security measure alone. It's for space optimization. [V-02]

Always install production dependencies only. [V-02]

Always copy dependency manifests before application code to leverage layer caching. [V-02]

ALWAYS:
```dockerfile
# Copy dependency files first (cached if unchanged)
COPY package*.json ./
RUN npm ci --only=production

# Copy application code (invalidates cache when changed)
COPY . .
```

---

## Network & Ports

### Port Exposure

Never expose debug ports in production images. [V-02, V-14]

Never expose database ports (5432, 3306, 27017) from application containers. [V-02]

Only expose application service ports. [V-02]

Always document why each port is exposed. [V-02]

NEVER:
```dockerfile
EXPOSE 3000 5005 9229 5432
# 5005 = Java debug, 9229 = Node debug, 5432 = PostgreSQL
```

ALWAYS:
```dockerfile
EXPOSE 3000
# Application HTTP server only
```

### Network Isolation

Always run containers in isolated networks (user-defined bridge networks, not default bridge). [V-02]

Never use host network mode unless absolutely necessary. [V-02]

Always use internal networks for database-to-app communication. [V-02]

---

## Health Checks

### Health Check Implementation

Always add `HEALTHCHECK` instructions to production images. [V-02]

Always implement health checks that verify application readiness, not just process existence. [V-02]

Never expose sensitive system information in health check responses. [V-14]

ALWAYS:
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node healthcheck.js || exit 1
```

Or for simple HTTP checks:
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1
```

Never expose stack traces or internal errors in health endpoints. [V-14]

---

## Image Metadata

### Labels

Always add metadata labels for maintainability. [V-02]

Always include version, description, and maintainer labels. [V-02]

ALWAYS:
```dockerfile
LABEL org.opencontainers.image.version="1.0.0"
LABEL org.opencontainers.image.description="Application name and purpose"
LABEL org.opencontainers.image.source="https://github.com/org/repo"
```

---

## Security Scanning

### Pre-Deployment Scanning

Always scan images for vulnerabilities before deployment. [V-12]

Always use tools like `docker scout`, `trivy`, or `grype` in CI/CD pipelines. [V-12]

Always address critical and high-severity vulnerabilities before production deployment. [V-12]

Never deploy images with known critical vulnerabilities. [V-12]

ALWAYS scan before deployment:
```bash
# Docker Scout
docker scout cve myapp:latest

# Trivy
trivy image myapp:latest

# Grype
grype myapp:latest
```

Set up automated scanning in CI/CD:
```bash
# Fail build on critical vulnerabilities
docker scout cve --exit-code --only-severity critical,high myapp:latest
```

---

## Complete Secure Dockerfile Example

### Node.js Application

```dockerfile
# syntax=docker/dockerfile:1.4

# Build stage
FROM node:18.20.2-alpine3.19 AS builder

WORKDIR /app

# Install dependencies (cached layer)
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:18.20.2-alpine3.19

# Create non-root user
RUN addgroup -S app && adduser -S app -G app

WORKDIR /app

# Copy dependencies from builder
COPY --from=builder --chown=app:app /app/node_modules ./node_modules

# Copy application code
COPY --chown=app:app package*.json ./
COPY --chown=app:app src/ ./src/

# Switch to non-root user
USER app

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

# Expose application port only
EXPOSE 3000

# Metadata
LABEL org.opencontainers.image.version="1.0.0"
LABEL org.opencontainers.image.description="Secure Node.js application"

# Start application
CMD ["node", "src/server.js"]
```

### Python Application

```dockerfile
# syntax=docker/dockerfile:1.4

# Build stage
FROM python:3.11.8-slim-bookworm AS builder

WORKDIR /app

# Install dependencies
COPY requirements.txt ./
RUN pip install --user --no-cache-dir -r requirements.txt

# Production stage
FROM python:3.11.8-slim-bookworm

# Create non-root user
RUN groupadd -r app && useradd -r -g app app

WORKDIR /app

# Copy dependencies from builder
COPY --from=builder --chown=app:app /root/.local /home/app/.local

# Copy application code
COPY --chown=app:app . .

# Update PATH for user-installed packages
ENV PATH=/home/app/.local/bin:$PATH

# Switch to non-root user
USER app

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')" || exit 1

# Expose application port only
EXPOSE 8000

# Metadata
LABEL org.opencontainers.image.version="1.0.0"
LABEL org.opencontainers.image.description="Secure Python application"

# Start application
CMD ["python", "app.py"]
```

---

## Implementation Checklist

Before deploying containers, verify:

- [ ] Base image uses specific version tag with variant
- [ ] Application runs as non-root user
- [ ] Multi-stage build excludes build tools from production image
- [ ] No secrets in Dockerfile, image layers, or committed files
- [ ] Comprehensive `.dockerignore` file exists
- [ ] Only necessary ports exposed (no debug or database ports)
- [ ] Health check implemented
- [ ] Dependencies pinned and installed from lock files
- [ ] Image scanned for vulnerabilities
- [ ] Production dependencies only (no dev dependencies)

---

**End of Container & Docker Security Rules v0.1.0**
