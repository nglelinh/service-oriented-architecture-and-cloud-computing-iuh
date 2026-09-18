---
layout: post
title: 13-03 Observability Depth — Logs, Metrics, Traces, OpenTelemetry
chapter: '13'
order: 4
owner: Nguyen Le Linh
lang: en
categories:
- chapter13
lesson_type: optional
---

Chapter 13’s required notes mention monitoring and logging. The Chapter 02 optional note explains why traces exist in a system with no global clock. The Chapter 13-02 optional note ties OpenTelemetry to canaries and signed artifacts. This lesson is the **depth pass**: what each signal is *for*, how OpenTelemetry actually moves data, and how an IUH team avoids a dashboard that cannot answer “why is checkout slow?”

## Learning objectives

- Separate **logs, metrics, and traces** by the question each answers, then correlate them with one context (`trace_id` / `service.version`).
- Describe an OpenTelemetry path: SDK → (optional Collector) → backend, including the W3C `traceparent` header.
- Choose a small **SLI/SLO** set and say what you would *not* alert on.
- Name cardinality and sampling as the two ways observability bills and lies.

## 1. Three signals, three jobs

| Signal | Question | Failure mode if you only have this |
| --- | --- | --- |
| **Metrics** | Is the *service* healthy in aggregate this minute? | You see “p95 is 800 ms” and cannot name the slow dependency. |
| **Logs** | What did *this process* emit about *this event*? | You drown in lines and cannot follow one user across four containers. |
| **Traces** | Which *causal path* did this request take? | You have a pretty graph and no capacity signal (CPU, queue depth). |

Production teams keep all three. The 2020s contract is: **instrument once**, export many. OpenTelemetry (OTel) is that contract. It entered the CNCF in 2019, incubated in 2021, and **graduated on 11 May 2026**. Graduation means the data model and SDKs are stable enough to teach as infrastructure, not as a vendor agent you rip out next year.

```text
request ──► span "edge"
               ├── span "api.http"
               │      ├── span "db.query"
               │      └── span "cache.get"
               └── span "worker.publish"
```

A **span** is a timed operation. A **trace** is a tree of spans that share a `trace_id`. Logs should carry that `trace_id`. Metrics should carry `service.name` and `service.version` so a canary is visible as a *new time series*, not a mysterious bump on the old one.

## 2. RED, USE, and the metrics you actually need

Two teaching mnemonics are enough for this course:

- **RED** (request-scoped services): **R**ate, **E**rrors, **D**uration.
- **USE** (resources): **U**tilization, **S**aturation, **E**rrors (disk full, dropped packets).

An IUH Course Board API needs RED on the HTTP edge and USE on the database container (CPU, connections in use, disk). You do not need 400 custom metrics on day one.

```python
import time
from collections import Counter

_requests = Counter()
_errors = Counter()
_latency_ms = []

def observe(route: str, status: int, started: float) -> None:
    _requests[route] += 1
    if status >= 500:
        _errors[route] += 1
    _latency_ms.append((route, (time.monotonic() - started) * 1000))

def error_rate(route: str) -> float:
    n = _requests[route]
    return 0.0 if n == 0 else _errors[route] / n
```

The snippet is a cartoon of a **counter** and a **histogram-like list**. Real systems use a metrics library and an exporter. The idea is the same: *aggregates first*, traces when an aggregate looks wrong.

<div class="content-box insight-box">
<p><strong>SLI vs SLO vs SLA.</strong> An SLI is a measurement (99th percentile latency, success ratio). An SLO is a target you chose (99.5% of Course Board GETs under 300 ms in a week). An SLA is a <em>contract</em> with consequences. Students should write SLOs. They should not invent SLAs for a lab.</p>
</div>

## 3. Logs are cheap to write and expensive to use

Good lab logs are **structured** (JSON) and **event-shaped** (`msg`, `request_id`, `trace_id`, `user_id` hashed, `duration_ms`). Bad lab logs are `print("here")` in three languages.

Rules that survive contact with a 2-person IUH project:

1. Log **start and end** of an external call, not every line of business logic.
2. Never log passwords, session cookies, raw student IDs, or full card-like numbers. Hash or drop.
3. Send container logs to **stdout/stderr** (Chapter 08). Let the platform ship them. Do not write a custom FTP-to-disk logger.
4. A log without a `trace_id` is a postcard with no address.

## 4. OpenTelemetry as a pipeline, not a product

OTel’s moving parts:

1. **API** — the calls your code uses (`start_as_current_span`).
2. **SDK** — sampling, processors, resource attributes (`service.name=courseboard-api`).
3. **Instrumentation** — libraries that wrap FastAPI, JDBC, HTTPX, gRPC so you do not span every function by hand.
4. **Collector** (optional but common) — a sidecar or gateway that receives OTLP and exports to Jaeger, Prometheus, Grafana Tempo, or a vendor.
5. **Context propagation** — W3C **Trace Context** (`traceparent`) on HTTP/gRPC so the next process continues the same tree.

```python
from opentelemetry import trace
from opentelemetry.propagate import inject

tracer = trace.get_tracer("courseboard.api")

def call_worker(http_post, url: str, payload: dict) -> None:
    with tracer.start_as_current_span("worker.publish") as span:
        span.set_attribute("messaging.destination", "board.events")
        headers = {}
        inject(headers)  # writes traceparent
        http_post(url, json=payload, headers=headers)
```

You can complete Deploy Track labs **without** a Collector: print `trace_id` on each request, or run a single Jaeger all-in-one container. The learning outcome is the *path*, not a particular SaaS.

Chapter 13-02 already showed `service.version` on a canary. Keep that habit: the same attribute belongs on metrics and on deploy annotations.

## 5. Sampling, cardinality, and how observability lies

Two ways a student project “has observability” and still cannot debug:

**Cardinality explosion.** A metric label `user_id` or `student_email` turns one time series into fifty thousand. Backends slow down or bills jump. Labels should be *low-cardinality*: `route`, `code`, `region`, `version`. Put unique IDs on *spans and logs*.

**Sampling.** Head sampling (“keep 5% of traces”) is cheap and misses the one failed payment. Tail sampling (“keep errors and slow traces”) needs a Collector that can look at the finished tree. For IUH labs, **keep 100%** until you have more than a toy load. Then say which policy you chose.

```text
good labels:  http.route=/api/posts  http.status=500  service.version=sha-8f3c
bad labels:   user=sv123456  email=a@iuh.edu.vn  query="select ..."
```

## 6. Alerts that a two-person team can wake up for

Alert on **symptoms** that match an SLO, not on every CPU blip:

| Wake someone | Leave for the weekly review |
| --- | --- |
| Error ratio $$2\times$$ baseline for 10 minutes | A single 5xx |
| SLO burn that will exhaust the week’s error budget in hours | p99 slowly worse on a unused admin route |
| Disk > 90% on the database volume | A debug log line you do not like |

Tie alerts to **runbooks** of five lines: what to look at, how to roll back (Chapter 09 / 13-04), who owns DNS if the cert expired (Chapter 01-07).

## 7. Minimal Course Board observability

Enough for the capstone, not a platform career:

1. RED metrics on the API (Prometheus format *or* even a `/metrics` text file you scrape).
2. JSON logs with `trace_id` and `service.version`.
3. One trace per HTTP request, children for DB and outbound HTTP.
4. A health endpoint (`/healthz`) that checks the database, used by Compose/Kubernetes—not as a substitute for RED.
5. A screenshot or exported panel in the report that shows a **forced failure** (stop the DB, watch the error rate and the broken span).

<div class="content-box exercise-box">
<p><strong>Lab honesty.</strong> A green dashboard with no injected failure does not prove you can observe. Break something on purpose, then write what the three signals showed.</p>
</div>

## Challenges

- Auto-instrumentation that spans noise (every `list.append`) and hides the one slow query.
- Clocks: span duration is local. Compare traces, not wall clocks across laptops (Chapter 02).
- Privacy: traces can carry query strings with student names. Drop or hash attributes in the SDK.
- Vendor lock-in: if you only use a vendor agent API, the Chapter 12 multi-cloud story repeats in telemetry.

## Exercises

1. For Course Board `POST /api/posts`, write one RED SLI, one USE SLI for Postgres, and an SLO for a one-week lab demo.
2. A trace shows the API span at 1.2 s and the DB span at 20 ms. Where do you look next, and which signal confirms it?
3. Mark each proposed metric label as safe or unsafe: `http.status`, `tenant_id` (3 tenants), `session_id`, `sql_text`.
4. Explain in four sentences what the Collector does that the SDK alone does not.

## Where this goes next

- [13-02 Supply-chain and canary OTel]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_02_Supply_Chain_Security_and_Observability %}) — version attributes on deploys.
- [13-04 CI/CD]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %}) — fail a pipeline when a smoke test has no `trace_id`.
- [Deploy Track Lab A]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_01_Lab_Docker_Local %}) — stdout logs and a healthcheck.

## References

1. OpenTelemetry documentation and CNCF graduation announcement (11 May 2026).
2. W3C Trace Context (`traceparent`) — the propagation header this lesson assumes.
3. Google SRE workbook chapters on SLIs/SLOs (public); USE method (Brendan Gregg); RED method (as commonly taught for request services).
4. [infracourse.cloud](https://infracourse.cloud/) — CS 40 treats observability as part of deploy-at-scale. This IUH lesson is original courseware.
