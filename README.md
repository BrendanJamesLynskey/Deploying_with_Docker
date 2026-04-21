# ⚓ Deploying with Docker

An interactive Reveal.js presentation on Docker and containers for deployment — Dockerfiles, Compose, registries, Kubernetes basics, and cloud container hosting.

## ▶ [Open the Presentation](https://brendanjameslynskey.github.io/Deploying_with_Docker/)

## 📄 [Markdown Version](presentation.md)

---

## Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | Topics | Fundamentals, building, registries, orchestration |
| 02 | The Problem Containers Solve | Dependency hell and environment inconsistency |
| 03 | What Is a Container? | Isolated process with namespaces and cgroups |
| 04 | Docker Architecture | CLI, daemon, images, containers, registries |
| 05 | Writing a Dockerfile | Instructions, layer caching, .dockerignore |
| 06 | Multi-Stage Builds | Separate build and runtime stages for smaller images |
| 07 | Docker Commands Cheat Sheet | build, run, ps, logs, exec, prune |
| 08 | Docker Compose | Multi-container apps via single YAML file |
| 09 | Container Registries | Docker Hub, GHCR, ECR, GAR, ACR |
| 10 | Pushing & Pulling Images | Manual workflow and CI/CD automation |
| 11 | Docker Networking | Bridge, host, overlay, macvlan drivers |
| 12 | Volumes & Persistent Data | Named volumes, bind mounts, tmpfs |
| 13 | Container Security Best Practices | Non-root, minimal base, scanning |
| 14 | Container Orchestration — Why | Scheduling, scaling, rolling updates |
| 15 | Kubernetes Basics | Pods, Services, Deployments, Ingress |
| 16 | Deploying Containers to Cloud | ECS/Fargate, Cloud Run, Container Apps |
| 17 | Free & Cheap Container Hosting | Cloud Run, Fly.io, Railway, Render |
| 18 | Common Pitfalls & Debugging | Huge images, signals, cache busting |
| 19 | Summary & Further Reading | Key takeaways, learning path, tools |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | Append `?print-pdf` to URL, then print |

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm, no dependencies to install.

## References

[Docker Documentation](https://docs.docker.com) · [Kubernetes Documentation](https://kubernetes.io/docs/) · [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) · *Docker Deep Dive* by Nigel Poulton

## License

Educational use. Code examples provided as-is.
