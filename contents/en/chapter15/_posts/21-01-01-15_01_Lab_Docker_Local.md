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
