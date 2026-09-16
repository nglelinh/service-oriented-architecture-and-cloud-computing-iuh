---
layout: post
title: 13-02 Supply-Chain Security, SLSA, and Production Observability
chapter: '13'
order: 3
owner: Nguyen Le Linh
lang: en
categories:
- chapter13
lesson_type: optional
---

The required lesson covers IAM, encryption, blue/green/canary, CI/CD, and monitoring. This optional note adds the 2022–2026 **software supply chain** layer (SLSA, SBOM, Sigstore) and treats **OpenTelemetry** as the production observability contract—not a vendor agent bolted on after go-live.

## Learning objectives

- Distinguish SBOM (“what is inside”) from SLSA provenance (“how it was built”).
- Place canary + signed artifacts on the same pipeline.
- Connect traces to deployment versions so a bad canary is obvious.

## 1. Why deploy security grew a fourth pipeline stage

SolarWinds (2020) and later ecosystem incidents (including the 2024 xz-adjacent scare) shifted trust from “the binary is from CI” to **attestations**. OpenSSF **SLSA v1.0** (first stable spec, October 2023) defines a Build track (L1–L3): provenance, isolated builds, and hardened platforms. GitHub artifact attestations (GA 2024) and npm provenance (from 2023) made this visible to students on public packages.

**SBOM** (SPDX or CycloneDX) answers inventory/CVE questions. CISA and NIST SSDF (SP 800-218), driven by EO 14028, pushed SBOMs and secure-development attestations for government software. The EU **Cyber Resilience Act** keeps the same pressure on commercial software into the late 2020s.

```text
source (reviewed) → isolated CI → signed provenance + SBOM → verify on deploy → canary
```

```yaml
# Conceptual deploy gate (not a specific vendor)
steps:
  - verify_cosign: image: ghcr.io/iuh/checkout@sha256:...
  - require_slsa: level: 2
  - require_sbom: format: cyclonedx
  - canary:
      percent: 5
      abort_if: "otel.error_rate > 2x baseline"
```

Blue/green from the required notes is necessary but not sufficient if the *green* artifact was built on a laptop.

## 2. Observability is part of the deploy

OpenTelemetry graduated in the CNCF on **11 May 2026**. The application pattern:

1. Every service emits traces/metrics with `service.version` = Git SHA or image digest.
2. Canary analysis compares *that* version’s error rate and latency to the previous digest.
3. Logs stay linked via `trace_id` (W3C Trace Context).

This is the required “monitoring and logging” outcome with a portable SDK.

```python
from opentelemetry import trace

tracer = trace.get_tracer("deploy.canary")

def handle(req):
    with tracer.start_as_current_span("checkout") as span:
        span.set_attribute("service.version", "sha-8f3c")
        return route(req)
```

## 3. Zero trust, still in one paragraph

Identity (workload identity / SPIFFE via a mesh, Chapter 09), least-privilege IAM (required), and **short-lived OIDC** to clouds (no long-lived access keys in CI) are the 2024 default. Supply-chain controls stop a *signed malicious* deploy; zero trust stops a *stolen key* from looking like a normal service.

<div class="content-box insight-box">
<p><strong>Shared responsibility extended.</strong> The cloud encrypts the disk. You still own the lockfile, the builder, and the attestation verifier in the cluster.</p>
</div>

## Challenges

- SLSA L3 isolation is hard on self-hosted runners.
- SBOM noise (thousands of CVEs) without VEX / reachability.
- Sampling traces so canary analysis still sees rare errors.

## Exercises

1. For a student project image, list what would appear in a CycloneDX SBOM vs. a SLSA provenance statement.
2. Design a canary abort rule that uses OTel metrics, not only HTTP 500s (include latency).
3. Explain why storing `AWS_SECRET_ACCESS_KEY` in GitHub Actions variables fails the 2024 CI identity story.

## References

1. SLSA v1.0 specification: [slsa.dev/spec/v1.0](https://slsa.dev/spec/v1.0/) and [What’s new](https://slsa.dev/spec/v1.0/whats-new).
2. NIST SP 800-218, *Secure Software Development Framework (SSDF)*.
3. CISA SBOM resources; U.S. EO 14028 (context for federal attestations).
4. CNCF OpenTelemetry graduation: [cncf.io/projects/opentelemetry](https://www.cncf.io/projects/opentelemetry/).
5. Sigstore / GitHub artifact attestations documentation (2024 GA).
