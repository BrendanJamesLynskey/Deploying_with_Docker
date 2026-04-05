# Deploying with Docker & Containers

**Build, Ship, and Run Anywhere**

> From local development to production infrastructure -- packaging applications into portable, reproducible containers.

`Dockerfile` -> `Build` -> `Image` -> `Registry` -> `Deploy`

Build - Ship - Run - Scale

---

## Table of Contents

1. [Topics](#topics)
2. [The Problem Containers Solve](#the-problem-containers-solve)
3. [What Is a Container?](#what-is-a-container)
4. [Docker Architecture](#docker-architecture)
5. [Writing a Dockerfile](#writing-a-dockerfile)
6. [Multi-Stage Builds](#multi-stage-builds)
7. [Docker Commands Cheat Sheet](#docker-commands-cheat-sheet)
8. [Docker Compose](#docker-compose)
9. [Container Registries](#container-registries)
10. [Pushing & Pulling Images](#pushing--pulling-images)
11. [Docker Networking](#docker-networking)
12. [Volumes & Persistent Data](#volumes--persistent-data)
13. [Container Security Best Practices](#container-security-best-practices)
14. [Container Orchestration -- Why You Need It](#container-orchestration--why-you-need-it)
15. [Kubernetes Basics](#kubernetes-basics)
16. [Deploying Containers to Cloud](#deploying-containers-to-cloud)
17. [Free & Cheap Container Hosting](#free--cheap-container-hosting)
18. [Common Docker Pitfalls & Debugging](#common-docker-pitfalls--debugging)
19. [Summary & Further Reading](#summary--further-reading)

---

## Topics

### Container Fundamentals

- The problem containers solve
- Containers vs virtual machines
- Docker architecture
- Writing Dockerfiles

### Building & Running

- Multi-stage builds
- Docker commands cheat sheet
- Docker Compose
- Networking & volumes

### Registries & Security

- Container registries
- Pushing & pulling images
- Security best practices
- Common pitfalls & debugging

### Orchestration & Deployment

- Container orchestration intro
- Kubernetes basics
- Deploying to cloud providers
- Free & cheap hosting options

---

## The Problem Containers Solve

Every developer has heard **"But it works on my machine!"** -- the classic symptom of environment inconsistency between development, staging, and production.

### Dependency Hell

- App needs Python 3.11, server has 3.8
- Library A requires OpenSSL 1.1, Library B needs 3.0
- Dev uses macOS, CI runs Ubuntu, prod is Amazon Linux
- Global package installs conflict across projects
- "Just install these 47 things first..."

### Without Containers

- Manual server provisioning (hours/days)
- Snowflake servers -- no two are alike
- Configuration drift over time
- Painful rollbacks
- "Don't touch that server, nobody knows how it works"

### With Containers

- Identical environment everywhere
- Dependencies bundled with the app
- Spin up in seconds, not hours
- Version-controlled infrastructure
- Instant rollback -- just run the old image

---

## What Is a Container?

A container is a **lightweight, isolated process** that shares the host OS kernel but has its own filesystem, network, and process space.

### Virtual Machines vs Containers

**Virtual Machines:**

```
App A        App B        App C
Bins/Libs    Bins/Libs    Bins/Libs
Guest OS     Guest OS     Guest OS
         Hypervisor
           Host OS
        Infrastructure
```

**Containers:**

```
App A        App B        App C
Bins/Libs    Bins/Libs    Bins/Libs
       Container Engine
           Host OS
        Infrastructure
```

### Linux Kernel Features

- **Namespaces** -- isolate PID, network, mount, user, IPC
- **cgroups** -- limit CPU, memory, disk I/O
- **Union filesystems** -- layered, copy-on-write storage

### Key Differences

- VMs: full OS per instance (~GB), boot in minutes
- Containers: shared kernel (~MB), start in milliseconds
- Containers are *not* lightweight VMs -- they're isolated processes

---

## Docker Architecture

```
Docker CLI          Docker Daemon (dockerd)              Registry
+-----------+       +---------------------------+        +------------------+
| docker    | REST  |  +--------+ +--------+    | pull/  | Docker Hub       |
|   build   |------>|  | Images | | Volumes|    | push   | GitHub GHCR      |
|   run     |  API  |  +--------+ +--------+    |------->| AWS ECR          |
|   push    |       |  |Contain.| |containerd|  |        | Private Registry |
+-----------+       |  +--------+ | + runc  |   |        +------------------+
                    |  |Networks|  +--------+   |
                    +---------------------------+
```

### Key Concepts

- **Image** -- Read-only template with app code, runtime, libraries. Built in layers from a Dockerfile.
- **Container** -- Running instance of an image. Writable layer on top. Isolated process with its own filesystem.
- **Registry** -- Repository for storing and distributing images. Like GitHub but for container images.

---

## Writing a Dockerfile

```dockerfile
# Node.js Express API Dockerfile
FROM node:20-alpine

# Create app directory
WORKDIR /app

# Install dependencies first (better caching)
COPY package*.json ./
RUN npm ci --only=production

# Copy application source
COPY src/ ./src/
COPY tsconfig.json ./

# Build TypeScript
RUN npm run build

# Expose the port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://localhost:3000/health || exit 1

# Run as non-root user
USER node

# Start the app
CMD ["node", "dist/server.js"]
```

### Key Instructions

- `FROM` -- base image to build upon
- `WORKDIR` -- set working directory
- `COPY` -- copy files into image
- `RUN` -- execute build commands
- `EXPOSE` -- document the port
- `CMD` -- default run command

### Layer Caching Tips

- Copy `package.json` before source code
- Dependencies cached if lockfile unchanged
- Order instructions from least to most changed
- Each instruction creates a new layer

### .dockerignore

Like .gitignore -- exclude `node_modules`, `.git`, `.env`, test files from the build context.

---

## Multi-Stage Builds

Use multiple `FROM` statements to **separate build-time and runtime dependencies**, dramatically reducing final image size.

```dockerfile
# -- Stage 1: Build --
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
RUN npm prune --production

# -- Stage 2: Production --
FROM node:20-alpine AS production
WORKDIR /app

# Only copy what we need
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

USER node
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

### Single-Stage

~850 MB -- includes TypeScript compiler, dev dependencies, source files, build tools

### Multi-Stage

~150 MB -- only production node_modules and compiled JS. No compiler, no dev deps.

### Common Patterns

- Go: build static binary, copy to `scratch`
- Rust: build binary, copy to `debian-slim`
- React: build with Node, serve with `nginx:alpine`
- Java: build with Maven, run with JRE-only image

---

## Docker Commands Cheat Sheet

| Command | Description | Example |
|---------|-------------|---------|
| `docker build` | Build image from Dockerfile | `docker build -t myapp:1.0 .` |
| `docker run` | Create & start container | `docker run -d -p 3000:3000 myapp:1.0` |
| `docker ps` | List running containers | `docker ps -a` (include stopped) |
| `docker logs` | View container output | `docker logs -f --tail 100 myapp` |
| `docker exec` | Run command in container | `docker exec -it myapp sh` |
| `docker stop` | Gracefully stop container | `docker stop myapp` |
| `docker rm` | Remove stopped container | `docker rm myapp` |
| `docker images` | List local images | `docker images --filter dangling=true` |
| `docker system prune` | Clean up unused resources | `docker system prune -af` |

### Useful Flags for `docker run`

`-d` detached | `-p 8080:3000` port map | `-v ./data:/data` volume | `--name myapp` name | `--rm` auto-remove | `-e KEY=val` env var | `--network mynet` network

---

## Docker Compose

Define and run **multi-container applications** with a single YAML file. One command to start everything.

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/myapp
      - REDIS_URL=redis://cache:6379
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    volumes:
      - ./src:/app/src   # dev hot-reload

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: myapp
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 5s
      timeout: 3s
      retries: 5

  cache:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  pgdata:
```

### Key Commands

- `docker compose up -d` -- start all services
- `docker compose down` -- stop and remove
- `docker compose logs -f app` -- follow logs
- `docker compose build` -- rebuild images
- `docker compose ps` -- service status

### Features

- Service discovery via DNS names
- Health checks & startup ordering
- Named volumes for persistence
- Environment variable files (`.env`)
- Override files for dev/prod configs

### Pro Tip

Use `docker-compose.override.yml` for dev settings (hot-reload, debug ports) that don't go to production.

---

## Container Registries

A **container registry** stores and distributes Docker images. Think of it as npm/PyPI but for container images.

| Registry | Free Tier | Best For |
|----------|-----------|----------|
| **Docker Hub** | 1 private repo, unlimited public | Open source, public images |
| **GitHub GHCR** | 500 MB free storage | GitHub-based workflows |
| **AWS ECR** | 500 MB/month (free tier) | AWS deployments |
| **Google AR** | 500 MB free w/ Cloud Run | GCP deployments |
| **Azure ACR** | Basic tier ~$5/mo | Azure deployments |

### Image Naming Convention

```bash
# Format: registry/namespace/image:tag
docker.io/library/node:20-alpine
ghcr.io/myorg/myapp:v1.2.3
123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
```

### Tag Strategy

- `latest` -- avoid in production (mutable!)
- `v1.2.3` -- semantic version (immutable)
- `sha-a1b2c3d` -- git commit hash
- `main-20260324` -- branch + date

---

## Pushing & Pulling Images

### Manual Workflow

```bash
# 1. Build the image
docker build -t myapp:v1.0.0 .

# 2. Tag for your registry
docker tag myapp:v1.0.0 \
  ghcr.io/myorg/myapp:v1.0.0

# 3. Authenticate
echo $GITHUB_TOKEN | docker login \
  ghcr.io -u USERNAME --password-stdin

# 4. Push to registry
docker push ghcr.io/myorg/myapp:v1.0.0

# 5. Pull on another machine
docker pull ghcr.io/myorg/myapp:v1.0.0
docker run -d -p 3000:3000 \
  ghcr.io/myorg/myapp:v1.0.0
```

### CI/CD Automation (GitHub Actions)

```yaml
# .github/workflows/docker.yml
name: Build & Push
on:
  push:
    tags: ['v*']
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:${{ github.ref_name }}
            ghcr.io/${{ github.repository }}:latest
```

---

## Docker Networking

| Driver | Use Case |
|--------|----------|
| **bridge** | Default. Containers on same host communicate via virtual bridge |
| **host** | Container shares host network stack. No port mapping needed |
| **overlay** | Multi-host networking for Docker Swarm / orchestration |
| **none** | No networking. Complete isolation |
| **macvlan** | Container gets its own MAC address on the physical network |

### DNS Service Discovery

On user-defined networks, containers can reach each other by **service name**. No need for IP addresses.

```bash
# Create a custom network
docker network create mynet

# Run containers on the same network
docker run -d --name api \
  --network mynet myapp:1.0
docker run -d --name db \
  --network mynet postgres:16

# 'api' container can connect to:
#   postgres://db:5432/mydb
# Docker resolves 'db' to the container IP
```

### Network Diagram

```
+--- mynet (bridge) --------------------------+
|                                              |
|  +------+      +------+      +-------+      |
|  | api  |----->|  db  |      | cache |      |
|  | :3000|  -  -| :5432| - - -| :6379 |      |
|  +------+      +------+      +-------+      |
+----------------------------------------------+
```

---

## Volumes & Persistent Data

Containers are **ephemeral** -- data inside is lost when the container is removed. Volumes solve this.

### Named Volumes

- Managed by Docker
- Stored in `/var/lib/docker/volumes/`
- Best for databases & persistent state
- Survive container removal

```bash
docker volume create pgdata
docker run -v pgdata:/var/lib/postgresql/data postgres:16
```

### Bind Mounts

- Map host directory into container
- Full path on host system
- Great for development (hot reload)
- Host file changes visible instantly

```bash
docker run \
  -v $(pwd)/src:/app/src \
  -v $(pwd)/config:/app/config:ro \
  myapp:dev
```

### tmpfs Mounts

- Stored in host memory only
- Never written to disk
- Good for secrets, temp data
- Lost when container stops

```bash
docker run \
  --tmpfs /tmp:rw,size=64m \
  --tmpfs /run/secrets \
  myapp:1.0
```

### Common Mistake

Never store database data inside the container without a volume. A `docker rm` will destroy all your data permanently.

---

## Container Security Best Practices

### Don't Do This

- Run as `root` inside containers
- Use `latest` tag in production
- Bake secrets into images
- Use full OS base images (ubuntu, debian)
- Expose Docker socket to containers
- Skip image scanning
- Run with `--privileged` flag

### Do This Instead

- Use `USER nonroot` in Dockerfile
- Pin image versions with SHA digests
- Pass secrets via env vars or Docker secrets
- Use `alpine` or `distroless` base images
- Scan with Trivy, Snyk, or Grype
- Set `read_only: true` where possible
- Drop all capabilities, add only needed ones

### Secure Dockerfile Example

```dockerfile
# Secure Dockerfile example
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM gcr.io/distroless/nodejs20-debian12
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

# No shell, no package manager, no root
USER nonroot:nonroot
EXPOSE 3000
CMD ["dist/server.js"]
```

### Scanning for Vulnerabilities

```bash
# Scan image for vulnerabilities
trivy image myapp:v1.0.0

# Output:
# Total: 2 (HIGH: 1, CRITICAL: 1)
# +----------+------------+----------+
# | Library  | Vuln ID    | Severity |
# +----------+------------+----------+
# | openssl  | CVE-2024-. | CRITICAL |
# | curl     | CVE-2024-. | HIGH     |
# +----------+------------+----------+
```

---

## Container Orchestration -- Why You Need It

Running one container on one server is simple. Running **dozens across multiple servers** with zero downtime requires orchestration.

### What Orchestrators Handle

- **Scheduling** -- place containers across nodes
- **Scaling** -- add/remove replicas automatically
- **Health checks** -- restart failed containers
- **Rolling updates** -- zero-downtime deployments
- **Service discovery** -- route traffic to healthy instances
- Load balancing, secrets, config management

### Orchestration Platforms

| Platform | Complexity | Best For |
|----------|-----------|----------|
| **Kubernetes** | High | Large-scale, multi-cloud, full control |
| **Docker Swarm** | Low | Small teams, simple orchestration |
| **AWS ECS** | Medium | AWS-native, Fargate serverless |
| **Nomad** | Medium | Multi-workload (containers + VMs + batch) |
| **Cloud Run** | Very Low | Serverless containers, scale to zero |

### Complexity Scale

`1 container` -> `Docker Compose` -> `Swarm / ECS` -> `Kubernetes`

(complexity & scale ->)

---

## Kubernetes Basics

### Cluster Architecture

```
+--- Kubernetes Cluster -------------------------+
|  +--------+     +---------+                    |
|  | Ingress| --> | Service |                    |
|  +--------+     +---------+                    |
|                  / | \                          |
|  +--- Deployment (replicas: 3) -----------+    |
|  | +----------+ +----------+ +----------+ |    |
|  | |   Pod    | |   Pod    | |   Pod    | |    |
|  | | myapp    | | myapp    | | myapp    | |    |
|  | +----------+ +----------+ +----------+ |    |
|  +-----------------------------------------+   |
|                                                 |
|  +-----------+ +--------+ +-----------------+  |
|  | ConfigMap | | Secret | | PersistentVolume|  |
|  +-----------+ +--------+ +-----------------+  |
|                                                 |
|  +--Node 1 (worker)--+ +--Node 2 (worker)--+  |
+--------------------------------------------------+
```

### Key Resources

- **Pod** -- Smallest deployable unit. One or more containers sharing network/storage.
- **Service** -- Stable network endpoint that load-balances across pod replicas.
- **Deployment** -- Manages pod replicas, rolling updates, and rollbacks declaratively.
- **Ingress** -- Routes external HTTP/S traffic to services. Handles TLS termination and path-based routing.

### Common kubectl Commands

```bash
kubectl get pods
kubectl apply -f deployment.yaml
kubectl scale deploy myapp --replicas=5
kubectl rollout status deploy/myapp
kubectl logs -f deploy/myapp
```

---

## Deploying Containers to Cloud

### AWS ECS / Fargate

- ECS = managed container orchestration
- Fargate = serverless (no EC2 to manage)
- Deep AWS integration (ALB, RDS, IAM)
- Task definitions define containers

```bash
# Deploy with AWS CLI
aws ecs update-service \
  --cluster prod \
  --service myapp \
  --force-new-deployment
```

### Google Cloud Run

- Fully managed, scales to zero
- Pay only when handling requests
- Deploy from image or source
- Built-in HTTPS & load balancing

```bash
# Deploy to Cloud Run
gcloud run deploy myapp \
  --image gcr.io/proj/myapp:v1 \
  --region us-central1 \
  --allow-unauthenticated
```

### Azure Container Apps

- Serverless container platform
- Built on Kubernetes (KEDA + Envoy)
- Scale to zero, event-driven
- Dapr integration for microservices

```bash
# Deploy to Azure
az containerapp create \
  --name myapp \
  --resource-group mygroup \
  --image myapp:v1 \
  --target-port 3000 \
  --ingress external
```

### Which to Choose?

**Already on AWS?** Use ECS/Fargate. **Want simplest option?** Google Cloud Run. **Need Kubernetes?** EKS, GKE, or AKS. **Small project?** See the next section for free/cheap options.

---

## Free & Cheap Container Hosting

You don't need a big budget to deploy containers. These platforms offer **generous free tiers** for side projects and startups.

| Platform | Free Tier | Pricing After | Key Features |
|----------|-----------|---------------|--------------|
| **Google Cloud Run** | 2M requests/mo, 360K vCPU-sec | Pay per request + compute | Scale to zero, HTTPS, custom domains |
| **Fly.io** | 3 shared VMs, 160GB bandwidth | ~$2/mo per extra VM | Global edge deploy, built-in Postgres |
| **Railway** | $5 free credit/mo | Usage-based (~$5-10/mo) | Git push deploy, databases included |
| **Render** | Free for static + web services | $7/mo for always-on | Auto-deploy from Git, managed DBs |
| **Coolify** | Self-hosted (free OSS) | VPS cost only (~$5/mo) | Self-hosted PaaS, Heroku alternative |

### Fastest Path to Production

```bash
# Google Cloud Run (from Dockerfile)
gcloud run deploy --source .

# Fly.io
fly launch  # auto-detects Dockerfile
fly deploy

# Railway
railway up  # deploys current directory
```

### Recommendation

For hobby projects: **Cloud Run** (generous free tier, scales to zero). For small production apps: **Fly.io** (predictable pricing, global edge). For teams: **Railway** (great DX, includes databases).

---

## Common Docker Pitfalls & Debugging

### Frequent Mistakes

- **Huge images** -- using `node:20` (1GB) instead of `node:20-alpine` (130MB)
- **No .dockerignore** -- copying `node_modules`, `.git` into the image
- **COPY . .** before `npm install` -- busts layer cache on every code change
- **Using latest tag** -- builds are not reproducible
- **Ignoring signals** -- app doesn't handle SIGTERM, slow shutdowns
- **Logging to files** -- Docker expects stdout/stderr

### Debugging Toolkit

```bash
# Why did my container crash?
docker logs myapp --tail 50

# What's happening inside right now?
docker exec -it myapp sh

# Inspect the full container config
docker inspect myapp

# Check resource usage
docker stats myapp

# See what's eating disk space
docker system df

# Debug a failed build
docker build --progress=plain \
  --no-cache -t myapp .

# Dive into image layers
# (install 'dive' tool)
dive myapp:latest
```

### Container Exits Immediately?

Check: Is `CMD` running a foreground process? Did you use exec form `CMD ["node", "server.js"]` vs shell form `CMD node server.js`? Is the app crashing? Check logs with `docker logs`.

---

## Summary & Further Reading

### Key Takeaways

- Containers package your app with all dependencies
- Docker makes building & running containers easy
- Multi-stage builds keep images small & secure
- Docker Compose manages multi-service apps locally
- Container registries store & distribute images
- Security: non-root, minimal images, scan regularly
- Orchestration (K8s, ECS) handles production scale
- Cloud Run / Fly.io for easy, cheap deployments

### Learning Path

`Dockerfile` -> `Compose` -> `CI/CD` -> `K8s`

### Further Reading

- **Docker Docs** -- docs.docker.com
- **Dockerfile Best Practices** -- docs.docker.com/develop/develop-images/dockerfile_best-practices
- **Kubernetes Docs** -- kubernetes.io/docs
- **"Docker Deep Dive"** by Nigel Poulton
- **"Kubernetes Up & Running"** by Hightower, Burns, Beda

### Hands-On Practice

- Containerise an existing project
- Write a docker-compose.yml with app + DB
- Push to GHCR via GitHub Actions
- Deploy to Cloud Run or Fly.io
- Try `minikube` for local Kubernetes

### Tools Worth Knowing

`dive` (inspect layers) - `hadolint` (lint Dockerfiles) - `trivy` (security scan) - `lazydocker` (TUI) - `ctop` (container top)
