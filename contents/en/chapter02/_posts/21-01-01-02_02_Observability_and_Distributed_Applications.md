---
layout: post
title: 02-02 Observability and Modern Distributed Applications (2022–2026)
chapter: '02'
order: 3
owner: Nguyen Le Linh
lang: en
categories:
- chapter02
lesson_type: optional
---

The required lesson defines a distributed system by concurrency, no global clock, and independent failure. This optional note shows how production teams *observe* those properties in 2022–2026, and why traces, metrics, and chaos experiments are now part of the application, not an afterthought.

## Learning objectives

- Explain why logs-only monitoring fails once a request fans out across services.
- Describe OpenTelemetry as a vendor-neutral telemetry contract (CNCF graduated, May 2026).
- Connect independent failure to *chaos* and *SLO-based* operations.

## 1. From “is the host up?” to “is the user journey intact?”

A single checkout may touch an API gateway, a cart service, a payments gRPC stub, a cache, and a warehouse topic. Host CPU being green does not mean the *causal path* succeeded. Modern observability treats a request as a **distributed trace**: a tree of spans with a shared `trace_id`.

OpenTelemetry (OTel) started in the CNCF in 2019, reached incubating status in 2021, and **graduated on 11 May 2026**. CNCF describes it as one of the highest-velocity projects in the foundation. Graduation is not hype: it is a signal that the *wire format and SDKs* are stable enough to standardize on, instead of locking every service to one vendor agent.

```python
from opentelemetry import trace

tracer = trace.get_tracer("checkout.service")

def charge(order_id: str, amount_cents: int) -> str:
    with tracer.start_as_current_span("charge") as span:
        span.set_attribute("order.id", order_id)
        span.set_attribute("amount.cents", amount_cents)
        # Independent failure: payment may timeout while cart already committed.
        return _payments_client.capture(order_id, amount_cents)
```

The span does not fix the race. It makes the race **visible** across process boundaries—the practical answer to “there is no global clock.”

## 2. Three signals, one correlation key

| Signal | Answers | Typical backend |
| --- | --- | --- |
| Metrics | Is the system healthy *in aggregate*? | Prometheus / managed metrics |
| Logs | What did *this* instance emit? | Loki, CloudWatch, etc. |
| Traces | Which *path* was slow or broken? | Jaeger, Tempo, vendor APM |

The 2022–2026 consensus is: **instrument once (OTel), export many**. Context propagation (W3C `traceparent`) is the naming/communication layer from the required notes, implemented as HTTP/gRPC headers.

<div class="content-box insight-box">
<p><strong>Naming revisited.</strong> Service discovery gives you an address. A <code>trace_id</code> gives you an identity for a <em>unit of work</em>. Distributed systems need both.</p>
</div>

## 3. Independent failure, practiced on purpose

Netflix’s Chaos Monkey popularized *injecting* failure. The 2020s version is narrower and more scientific: game days against **error-budget** policy (Google SRE). If your SLO is 99.9% successful checkouts, you have a monthly budget of failed requests. Chaos tests whether retries, timeouts, and bulkheads actually consume that budget slowly—or all at once.

A tiny timeout/retry policy (still a distributed-systems design issue):

```python
import time
import random

def call_with_budget(fn, timeout_s=0.2, attempts=3):
    for i in range(attempts):
        start = time.monotonic()
        try:
            return fn()
        except TimeoutError:
            if i == attempts - 1:
                raise
            # jitter avoids synchronized retry storms (no global clock!)
            time.sleep(min(timeout_s, 0.05) * (2 ** i) * random.random())
        if time.monotonic() - start > timeout_s:
            raise TimeoutError("local deadline exceeded")
```

## 4. Applications students will actually meet

- **Service mesh / sidecar or ambient data plane** exporting golden signals (latency, traffic, errors, saturation) without each team rewriting exporters.
- **Event-driven** workers where the “request” is a CloudEvent: you still need a correlation id on the message.
- **Multi-region** active-active: traces become the only honest way to see which region served the user.

CNCF’s 2024 survey (published April 2025) reports **80%** Kubernetes production use. Observability spend and complexity scale with that density; OTel is the industry’s attempt to keep the *contract* stable while backends change.

## Challenges

- High-cardinality labels (`user_id` on every metric) can melt a time-series database.
- Tracing 100% of traffic is expensive; tail-based sampling needs a collector that sees the full trace.
- Chaos without a rollback plan is just an outage.

## Exercises

1. Draw a sequence diagram of a three-service call. Mark where a `traceparent` header must be copied.
2. Explain, in four sentences, why a dashboard of average latency can hide a 1% timeout class.
3. Propose one chaos experiment for “independent failure” of a cache (not the whole VM).

## References

1. CNCF, *OpenTelemetry* project page (graduated 11 May 2026): [cncf.io/projects/opentelemetry](https://www.cncf.io/projects/opentelemetry/).
2. CNCF blog, “OpenTelemetry has graduated… Now what?” (24 July 2026): [cncf.io/blog/2026/07/24/opentelemetry-has-graduated-now-what](https://www.cncf.io/blog/2026/07/24/opentelemetry-has-graduated-now-what/).
3. CNCF, *Cloud Native 2024 Annual Survey*: [cncf.io/reports/cncf-annual-survey-2024](https://www.cncf.io/reports/cncf-annual-survey-2024/).
4. W3C, *Trace Context* (`traceparent`): [w3.org/TR/trace-context](https://www.w3.org/TR/trace-context/).
5. Beyer et al., *Site Reliability Engineering* (Google), SLO/error-budget chapters — still the operational reading paired with these tools.
