---
layout: post
title: 13-04 CI/CD for Cloud Applications
chapter: '13'
order: 5
owner: Nguyen Le Linh
lang: en
categories:
- chapter13
lesson_type: optional
---

The required Chapter 13 lesson introduces automated pipelines and names GitHub Actions. This optional lesson is the **IUH working model**: what a two-person team actually puts in the pipeline, how a build artifact reaches a *named environment*, and how you avoid storing cloud passwords in the repo. It is not a copy of any external assignment. The running example is still the **IUH Course Board**.

## Learning objectives

- Draw a pipeline as **source → verify → package → deploy → observe**, and say which steps a student laptop must not skip.
- Explain **build once, promote many** (the same image digest in `staging` and `prod`).
- Use **OIDC-style short-lived credentials** as the default story for “CI talks to a cloud,” and treat long-lived access keys as a lab smell.
- Write a rollback rule that names *who* presses it and *what* signal triggers it.

## 1. CI and CD are different jobs

**Continuous Integration (CI)** answers: “Does this change *build* and *behave* on a clean machine?”  
**Continuous Delivery / Deployment (CD)** answers: “Can we *release* that verified artifact to an environment—manually approved or automatically?”

Students often glue both into one workflow file and then cannot tell why “the pipeline is red.” Split the questions even if you keep one YAML file.

```text
push / pull request
        │
        ▼
   [CI] lint + unit tests + build image
        │
        ▼
   artifact (image digest, tarball, static site)
        │
        ▼
   [CD] deploy to staging → smoke → (approval) → prod
        │
        ▼
   observe (health + RED + a trace)
```

**Delivery** means prod is one approval away. **Deployment** means prod updates itself after green checks. For IUH coursework, **delivery** is the safer default: a human in the pair approves prod.

## 2. Build once, promote the digest

A common failure: CI builds image tag `latest` on the main branch, staging runs `latest` from Monday, prod rebuilds `latest` on Friday. Those are **different bits**.

The IUH rule:

1. CI produces an immutable artifact: `ghcr.io/iuh-courseboard/api@sha256:…` (or a tagged SemVer *plus* the digest in the report).
2. Staging and prod pull **that digest**.
3. Rollback means pointing the environment at the **previous digest**, not “rebuild an older commit and hope.”

{% raw %}
```yaml
# Conceptual GitHub Actions job — IUH Course Board
# (illustrative; pin action versions in a real repo)
jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Unit tests
        run: python -m pytest
      - name: Build image
        run: docker build -t courseboard-api:${{ github.sha }} ./api
      - name: Record digest
        run: docker inspect --format='{{.Id}}' courseboard-api:${{ github.sha }}
```
{% endraw %}

You do not need a container registry to *learn* the idea: save the image ID in `artifacts/build.json` and treat it as the digest in your report.

## 3. Environments are policy, not folder names

`dev`, `staging`, and `prod` are **promotions**, not copies of `docker-compose.yml` with different jokes in the welcome string.

| Environment | Data | Who can deploy | Networking |
| --- | --- | --- | --- |
| Local / compose | Synthetic | Anyone in the pair | Loopback + user-defined bridge |
| Staging | Synthetic or masked | Pipeline after CI green | Same *shape* as prod, cheaper size |
| Prod | Real or instructor-approved demo | Approval + short-lived creds | Public edge + private data (Chapter 01-07) |

Configuration belongs in **environment variables or a secret store**, not in rebuilt images. The image is the *code*. The environment is the *bindings*.

<div class="content-box warning-box">
<p><strong>Secrets in Git are a failed lab.</strong> If a <code>.env</code> with a cloud key appears in a commit, rotate the key, purge it from history if the instructor asks, and switch to repository secrets or OIDC. Screenshots of keys also count.</p>
</div>

## 4. How CI authenticates to a cloud (the 2024 default)

Long-lived `AWS_ACCESS_KEY_ID` in GitHub Secrets still works and still leaks. The pattern you should be able to *explain* is:

1. The workflow requests an **OIDC token** from GitHub.
2. The cloud **identity provider** trusts that token for a *specific* repository, branch, and role.
3. The role lasts minutes and can only deploy the staging account.

Azure **federated credentials** and GCP **Workload Identity Federation** are the same story. Chapter 13-02 already called this “short-lived OIDC.” This lesson adds the pipeline shape.

If your IUH lab never leaves the laptop, still write the paragraph: *what you would have used instead of a 90-day access key*.

## 5. Deploy steps that match Chapter 09 and 13

CD should reuse the strategies you already have names for:

- **Rolling** — default for Kubernetes Deployments; pipeline waits on `kubectl rollout status`.
- **Canary** — pipeline shifts 5–10% of traffic, reads an SLI, then continues or aborts (Chapter 09 enrichment + 13-02).
- **Blue/green** — pipeline flips a Service or DNS name after smoke tests.

A smoke test is not `curl` until you see HTML. A smoke test is:

1. Hit `/healthz` and a write/read path on Course Board.
2. Assert HTTP 200 and a JSON shape.
3. Prefer a response that includes `service.version` so you know **this** digest is live.

```python
import json
import urllib.request

def smoke(base: str, expected_sha: str) -> None:
    with urllib.request.urlopen(f"{base}/healthz", timeout=5) as res:
        body = json.loads(res.read().decode())
    if body.get("service.version") != expected_sha:
        raise SystemExit(f"wrong digest: {body}")
```

If the smoke test cannot see a version, your observability lesson is unfinished.

## 6. A pipeline a pair can finish in a week

Minimum IUH-complete workflow (local or GitHub-hosted):

1. **On pull request:** lint + unit tests. No deploy.
2. **On main:** tests + image build + (optional) push to a registry the team owns.
3. **Manual `workflow_dispatch` or environment approval:** deploy staging.
4. **Smoke + 5 minutes of RED** (even a script that samples `/healthz`).
5. **Prod** only with a written rollback: previous digest, who runs it, 15-minute timebox.

GitHub Actions is the default because this site already deploys with it. GitLab CI and Azure Pipelines are acceptable if the pair can explain the same five stages.

## 7. What “green” is allowed to mean

A green pipeline that never ran on a clean runner is local theater. Use the hosted runner *or* document a `act`-style local run and its limits.

Also fail CI when:

- The image builds as `root` and the Dockerfile has no `USER` (Chapter 08).
- Unit tests were skipped with an empty `|| true`.
- The compose file publishes `5432` to `0.0.0.0` (Chapter 01-07).

Policy as code in Chapter 14-02 is the grown-up form of these gates.

## Challenges

- Self-hosted runners that are unpatched become a supply-chain hole (Chapter 13-02).
- Matrix builds that take the free GitHub minutes to zero—keep the matrix small.
- Deploying from `latest` on two environments.
- Approvals that are a second GitHub account in the same pair… who never looks at the diff.

## Exercises

1. Split a one-job “build and SSH and docker compose up” script into CI vs CD. What artifact crosses the boundary?
2. Write a four-line rollback runbook for Course Board prod, including the signal that triggers it (use Chapter 13-03 vocabulary).
3. List three claims an OIDC role should *deny* (for example, `iam:*` or delete the billing account).
4. Your smoke test only hits `/`. Why is that not enough for a multilayer app?

## Where this goes next

- [14-01 Terraform]({{ site.baseurl }}{% multilang_post_url contents/chapter14/21-01-01-14_01_Infrastructure_as_Code %}) — the pipeline’s “provision” cousin.
- [12-03 FinOps]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %}) — a forgotten runner or NAT that CI never destroyed.
- [Capstone brief]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %})

## References

1. GitHub Actions documentation: workflows, environments, OIDC (`id-token: write`).
2. SLSA / artifact attestations — see [13-02]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_02_Supply_Chain_Security_and_Observability %}) for the signing layer.
3. [infracourse.cloud](https://infracourse.cloud/) (CS 40) includes CI/CD as a deploy-track topic. IUH pipelines and Course Board examples are original.
