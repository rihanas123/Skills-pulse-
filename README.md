# SkillPulse — GitHub Actions & Kubernetes Masterclass

A small, real application with a real CI/CD pipeline. SkillPulse lets you track skills you are learning and the hours you put in. The app is intentionally tiny so the focus stays on the pipeline, deployment, and infrastructure.



## Why DevOps matters

DevOps is the cultural and technical answer to the old “build vs run” split.

- Developers want to ship features.
- Operations want stability.
- DevOps says the same team owns the change all the way to production.
- Automation makes shipping safe, repeatable, and reversible.

When DevOps works:
- Deploys are boring.
- Rollbacks are cheap.
- Feedback is fast.
- Ownership is clear.

That automation is called a pipeline.

---

## Why CI/CD is the heart of DevOps

CI/CD is two ideas wearing one acronym:

- Continuous Integration: every change is built and tested automatically.
- Continuous Delivery / Deployment: every passing change becomes a release candidate.

The core lesson:
- The earlier you catch a bug, the cheaper it is to fix.
- The only way to make that reliable is to automate everything.

---

## Why GitHub Actions

A pipeline needs a runner. GitHub Actions is the lowest-friction option for this repo.

- It lives with the code in `.github/workflows/*.yml`
- It is free for public repos and generous for private repos
- The Marketplace lets you reuse actions instead of writing bash from scratch

The trade-off is GitHub lock-in, which is acceptable for most teams.

---

## What this project demonstrates

A real pipeline, end to end, in a small app.

```
Developer
   └─ git push ─▶ GitHub Repo
                     ├─ CI Workflow ─ build images, tag sha/latest, push
                     └─ CD Workflow ─ deploy via SSH or update kind manifests
```

### CI — `.github/workflows/ci.yml`

Triggered on every push to `main`. It:

- checks out the code
- builds backend and frontend Docker images
- tags each image with `:latest` and `:<sha>`
- pushes both images to Docker Hub

CI produces an artifact: the image. Production runs that same artifact.

### CD — `.github/workflows/cd.yml`

Triggered when CI succeeds. It:

- SSH into an EC2 instance
- clones or updates the repo
- checks for `.env`
- runs `docker compose pull` and `docker compose up -d`
- prunes old Docker images

This is a simple real deploy path and a common first pipeline.

---

## The application

A three-tier app:

- Frontend: HTML/CSS/vanilla JS served by Nginx
- Backend: Go 1.26 + Gin
- Database: MySQL 8.4

Nginx reverse-proxies `/api/` and `/health` to the backend so public traffic is one port.

API surface:

- `GET /api/skills`
- `POST /api/skills`
- `GET /api/skills/:id`
- `DELETE /api/skills/:id`
- `POST /api/skills/:id/log`
- `GET /api/dashboard`
- `GET /health`

---

## Run locally

```bash
cp .env.example .env
docker compose up -d --build
```

Visit `http://localhost`

To tear down:

```bash
docker compose down -v
```

---

## Run on Kubernetes (kind)

Same app, same images, but with real Kubernetes primitives.

Prerequisites:
- Docker Desktop running
- `kind`
- `kubectl`

```bash
make up
```

Open `http://localhost:8888`

To delete:

```bash
make down
```

`make up` runs:
- `docker build` for backend and frontend
- `kind create cluster`
- `kind load docker-image`
- `kubectl apply` for namespace, MySQL, backend, frontend
- rollout status checks

---

## Continuous deployment to kind

GitHub Actions cannot reach your local kind cluster, so the deploy flow is GitOps-style.

1. Push code
2. CI builds and pushes images
3. `cd-k8s.yml` updates `k8s/20-backend.yaml` and `k8s/30-frontend.yaml` with the new SHA tags
4. Commit is pushed to `main`
5. Locally: `git pull && make apply`

This keeps the repo as the source of truth.

---

## Secrets and variables

For GitHub Actions:

- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`
- `EC2_HOST`
- `EC2_USER`
- `EC2_SSH_KEY`

For kind deploy path:
- `DEPLOY_ENABLED = "true"` as a repo variable

---

## Project layout

- `backend/` — Go service
- `frontend/` — static UI + Nginx
- `mysql/init.sql` — schema + seed data
- `docker-compose.yml`
- `.env.example`
- `.github/workflows/ci.yml`
- `.github/workflows/cd.yml`
- `k8s/` — kind manifests and config

---

## Useful commands

- `make status` — pods, services, endpoints
- `make logs` — tail logs
- `make mysql` — open MySQL shell
- `make restart` — rebuild and roll backend/frontend

---

