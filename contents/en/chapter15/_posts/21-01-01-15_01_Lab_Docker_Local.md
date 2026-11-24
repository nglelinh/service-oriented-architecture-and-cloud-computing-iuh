---
layout: post
title: 15-01 Lab A — Local multilayer app with Docker Compose
chapter: '15'
order: 2
owner: Nguyen Le Linh
lang: en
categories:
- chapter15
lesson_type: optional
---

**Format:** checklist lab (ungraded unless your instructor says otherwise).  
**Timebox:** about 2–3 focused hours for a pair.  
**Cloud:** none. If you do not have Docker, stop and install Docker Engine *or* Docker Desktop per your OS; do not substitute a public VM.

This lab is original IUH coursework. It is **not** a Stanford CS 40 assignment.

Before you ever spend a single cent on AWS or GCP, you can run a full multi-tier service-oriented architecture—reverse proxy, backend API, worker, and PostgreSQL database—right on your laptop in 3 seconds with **Docker Compose**. Even better, running locally exposes the exact networking, persistence, and startup-order bugs (*"why did my API crash before Postgres finished booting?"*) that bite engineers in production Kubernetes clusters!

Lab A walks you through building a clean, reproducible local multilayer stack with Docker Compose—internal bridge networks, named volumes, healthchecks, and zero hardcoded secrets.

---

## Background Knowledge Map: Four Docker Compose Mechanics Behind Local Multi-Tier Apps

1. **User-Defined Bridge Networks & Service DNS:** Containers on the same Compose network resolve each other by **service name** (e.g., `postgres://db:5432`), never by brittle `localhost` or hardcoded container IPs!
2. **Named Volumes vs. Ephemeral Container Layers:** Anything written inside a container’s writable layer disappears on `docker compose down`; mounting a **named volume** (`/var/lib/postgresql/data`) preserves database tables across restarts.
3. **Readiness Healthchecks (`depends_on: condition: service_healthy`):** Starting container B after container A *starts* is not enough—Postgres takes 2–5 seconds to accept SQL connections! Healthchecks (`pg_isready`) guarantee B waits until A is actually ready.
4. **Least-Exposure Port Mapping:** Publish only the front-door proxy/API port (`8080:8080`) to your host laptop; keep internal databases unreachable from outside the Compose network.

---

## Purpose

Stand up a **three-process Course Board** on your laptop: a browser-facing edge, an API, and a datastore. You should be able to point at each process and say which Chapter 01-07 tier it cartoons (edge / app / data).

## Outcomes

- A Compose file with **two user-defined networks** (or one frontend + one backend network) so the datastore port is not published to the host LAN.
- A `/healthz` on the API that fails if the datastore is down.
- Logs on stdout that include a `request_id` (a `trace_id` is even better).
- A short note (half a page) with a diagram.

## Suggested app (write your own code)

**IUH Course Board (local):** students can list and create *course help posts* (course code, title, body). No real student information. Synthetic names only (`sv0001`, not a classmate’s MSSV).

Minimum API:

| Method | Path | Behavior |
| --- | --- | --- |
| GET | `/healthz` | 200 if DB ping works; 503 otherwise |
| GET | `/api/posts` | JSON list |
| POST | `/api/posts` | JSON create; reject empty title |

The edge can be Caddy, nginx, or a tiny static UI served by the API. Pick one. Pin image tags (not `latest`).

## Checklist

### 0. Hygiene

- [ ] Pair names and date at the top of `NOTES.md`.
- [ ] Repo has a `.gitignore` that excludes `.env`, `*.pem`, and `__pycache__`.
- [ ] You can run `docker version` and `docker compose version`.

### 1. Networks, not just ports

- [ ] `docker-compose.yml` defines something like `edge` and `persist` networks.
- [ ] Datastore service: **no** `ports: ["5432:5432"]` (or Redis `6379`) on `0.0.0.0`. Only the API joins the persist network.
- [ ] Edge publishes `127.0.0.1:8080:80` (loopback bind) rather than a bare `8080:80` if your Compose version allows it.

### 2. Health and failure

- [ ] Compose `healthcheck` on the datastore *and* the API.
- [ ] `docker compose ps` shows healthy before you call the API.
- [ ] You **stop the datastore**, confirm `/healthz` is 503, and save the API log lines. Then start it again.

### 3. Observability (minimum)

- [ ] API logs one JSON line per request: `method`, `path`, `status`, `duration_ms`, `request_id`.
- [ ] Optional: print the same `request_id` in the datastore driver logs or as a comment in SQL.
- [ ] You did **not** log request bodies that might contain personal data.

### 4. Elasticity cartoon

- [ ] `docker compose up --scale api=2` works *or* you document why your edge cannot load-balance two APIs yet.
- [ ] If scale-out fails (sticky in-memory state), write that as a finding—not as a silent rewrite to a single process.

### 5. Done when

- [ ] A classmate’s browser on **your** machine hits `http://127.0.0.1:8080` and can create a post.
- [ ] `NOTES.md` has a diagram (Mermaid or a photo of a whiteboard) with three tiers and the two networks.
- [ ] `docker compose down -v` is documented. You know it deletes the volume.

## Stretch (still local)

- Terminate TLS on the edge with `mkcert` for `courseboard.localhost`.
- Add a `Makefile` with `make up`, `make test`, `make down`.
- Run the Chapter 13-04 smoke test against `/healthz`.

## What not to do

- Do not open the datastore to the campus Wi-Fi.
- Do not push images that contain `.env`.
- Do not follow a random blog’s `network_mode: host` as a shortcut.

## Instructor note

Evidence is `NOTES.md` + Compose file + the 503 screenshot/log. No rubric is copied from another university; grade locally if you use this as a marked lab.

---

### Worked Concrete Example: Debugging the Classic `localhost:5432` Connection Refused Bug

Suppose your `api` container tries to connect to your `db` container inside `docker-compose.yml`:

| `DATABASE_URL` in `api` Container | What `api` Actually Connects To | Result |
| :--- | :--- | :--- |
| `postgres://user:pw@localhost:5432/app` | `api`’s **own** loopback interface (no Postgres running there!) | **`ECONNREFUSED` crash!** |
| `postgres://user:pw@172.18.0.2:5432/app` | Hardcoded container IP (changes on next restart!) | Fragile — breaks randomly |
| `postgres://user:pw@db:5432/app` | Docker’s internal DNS resolving service `db` | **Works reliably every time!** |

### Deep-Dive Explanation: Four Habits for Clean Local Docker Labs (Read Slowly)

1. **Always check `docker compose ps` and `docker compose logs -f api` first:** 90% of startup failures show up clearly in the first 20 lines of container logs.
2. **Use `.env` (gitignored!) + `.env.example` (committed):** Teammates copy `.env.example` to `.env` locally so no passwords ever enter Git history.
3. **Know `down` vs. `down -v`:** `docker compose down` keeps your database volume intact; `docker compose down -v` wipes volumes clean for a fresh migration test.
4. **Use multi-stage Dockerfiles:** Build your app in a full SDK image and copy *only* the compiled binary/bundle into a tiny runtime image (`alpine` / `distroless`), cutting image size from 900 MB to ~30 MB!

**Quick check.** Inside a Docker Compose stack with services `web` and `postgres`, why does connecting from `web` to `localhost:5432` fail even if you mapped `ports: ["5432:5432"]` on `postgres`?

---

## Video tham khảo

- **xem trước:** [What is Docker Compose? \| Docker Concepts](https://www.youtube.com/watch?v=f3xb7kt_dH4) — Docker, English. Watch first: Docker’s definition of Compose, the tool this lab uses for a local multilayer app.
- [Docker Compose in 12 Minutes](https://www.youtube.com/watch?v=Qw9zlE3t8Ko) — Jake Wright, English. A Compose file with more than one service, close to the edge, app, and data roles in the lab.
- [What Is Docker - and why it’s EVERYWHERE](https://www.youtube.com/watch?v=W5fK8BmgBow) — Techquickie, English. What a container is, if the Compose file is the first Docker artifact in the lab.
