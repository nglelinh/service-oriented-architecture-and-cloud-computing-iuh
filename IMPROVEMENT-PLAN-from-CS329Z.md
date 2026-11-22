# Improvement Plan: SOA & Cloud Computing (IUH) ← CS 329Z pedagogy

**Date:** 2026-09-18  
**Source model:** [Stanford CS 329Z — Engineering AI Agents](https://cs329z.stanford.edu/)  
**Target course:** `service-oriented-architecture-and-cloud-computing-iuh`  
**Live site:** https://nglelinh.github.io/service-oriented-architecture-and-cloud-computing-iuh/

## 1. Executive read of CS 329Z (what to steal)

CS 329Z is **not** a cloud/Hadoop course. It is an **engineering course for agentic / compound AI systems**. The transferable value for IUH is mostly **pedagogy and assessment design**, plus an optional **bridge track** (agents running on cloud).

### 1.1 Core thesis (CS 329Z)
- Shift from monolithic models → **compound AI systems** (LLM + retrievers + tools + optimizers) → **autonomous agents**.
- Students learn to: pick problems, **decompose**, choose components, **curate data**, **build evals**, reason about **tradeoffs**.
- Pattern: **build blocks from scratch → then use frameworks** (so abstraction is earned, not magical).

### 1.2 Curriculum spine (CS 329Z)
| Block | Topics |
|-------|--------|
| Landscape | Pipelines vs compound systems vs agents; when compound wins |
| Builder skills | APIs/SDKs, structured I/O, context engineering, cost/latency |
| Building blocks | RAG from scratch; tool use / MCP; sandboxes & retries |
| Orchestration | DSPy / LangGraph / LlamaIndex; workflows vs agents |
| Patterns | ReAct, plan-and-execute, reflection; memory; multi-agent |
| Optimization | Prompt opt, test-time compute, LoRA (high level) |
| Data | Traces, flywheels, synthetic data, selection/quality |
| Evaluation | 4-tuple (request, environment, stopping criteria, scorer); LLM-as-judge |
| Safety | Prompt injection, sandboxing, HITL, privacy |
| Capstone themes | Coding agents, proactive agents, production/observability |

### 1.3 Assessment design worth copying
| Mechanism | Why it works |
|-----------|--------------|
| **HW1: build harness from scratch** (no frameworks) | Forces understanding of loops, tools, memory |
| **HW2: evaluate a given agent** | Eval is a first-class skill, not an afterthought |
| **HW-based oral quizzes** | Students defend design tradeoffs (anti-copy) |
| **Project milestones** (proposal → midpoint demo → final demo) | Working system early; demos over long essays |
| **Short reports (1 page + appendix)** | Writing supports the system, doesn’t replace it |
| **Paper video + peer reviews** | Research literacy without full paper reviews |
| **AI-tool honor code** | Explicit: AI as collaborator OK; finished solutions not OK |

### 1.4 Three engineering challenges (reuse as course slogans)
1. **Decomposition** — where to draw service / agent boundaries  
2. **Data** — what traces, demos, feedback you need  
3. **Evaluation** — how you know it works under realistic conditions  

---

## 2. Honest snapshot of the IUH course today

### 2.1 What it is
A **Jekyll lecture site** (14 chapters) focused on:
cloud fundamentals → distributed systems → Hadoop/YARN/MapReduce → Spark → NoSQL → Docker/K8s → big data platforms → data prep → cloud providers → security → Terraform.

Useful for SE / data workflows; site is multilingual-ready (EN + partial VI).

### 2.2 Gaps relative to a modern “engineering” course (and vs CS 329Z)
| Gap | Evidence | Impact |
|-----|----------|--------|
| **Sidebar titles ≠ lecture content** | e.g. Ch08 sidebar “Spark MLlib” but posts are Docker/serverless; Ch14 sidebar “Kubernetes” but posts are Terraform | Students navigate wrong; trust drops |
| **No published HW / labs / grading** | Site is content-heavy; little structured practice + assessment | Passive→passive reading |
| **No milestone project** | No proposal / midpoint demo / final system demo | Missing CS 329Z’s strongest learning loop |
| **Thin chapters** | Several chapters = Intro + 1 lecture only | Uneven depth (Spark/Docker thicker; others thin) |
| **Name vs content** | Title stresses **SOA**; body is mostly **cloud + big data** | Positioning unclear for IUH curriculum committees |
| **VI incomplete** | Only Chapter 01 mirrored in `contents/vi/` | Accessibility for IUH students |
| **Little “build from scratch then abstract”** | Lectures explain tools; weak DIY harness path | Matches old survey courses more than CS 329Z |
| **Almost no eval / observability / safety** | Ops appears late (Ch13) without agent/service eval culture | Graduates underprepared for production AI-on-cloud |

### 2.3 What to keep
- Solid cloud fundamentals (NIST, models, datacenters).
- Containers + K8s + Terraform as **deployment spine**.
- Spark / data chapters as optional or compressed **data platform** track.
- Existing Jekyll multilang + GitHub Pages pipeline.

---

## 3. Recommended product vision (choose explicitly)

**Do not replace the IUH course with a clone of CS 329Z.** Prerequisites and audience differ (IUH undergrad SE/IT vs Stanford NLP-prep grads).

### Option A — Pedagogy transplant (recommended default)
Keep **Cloud / SOA / platforms** as the domain. Rebuild **how students learn** using CS 329Z patterns:
- labs from scratch → then managed services / frameworks  
- eval + demo-centric project  
- oral defense of tradeoffs  

### Option B — Dual spine (Cloud + Agentic systems on cloud)
After a compressed cloud core, add a **Module C: Compound AI on cloud** (RAG, tools, agent loop, deploy on K8s/serverless, eval + guardrails). Closest content bridge to CS 329Z without abandoning IUH identity.

### Option C — Full pivot to “Engineering AI Agents” (not recommended short-term)
Would require new prerequisites, accreditation narrative, and near-full rewrite. Only if IUH formally renames/replaces the course.

**Recommendation:** **A now + B as Phase 3.**

---

## 4. Target course architecture (Option A + B)

### Part 0 — Course operating system (2 weeks equivalent of work for staff)
- Fix navigation titles; publish syllabus, schedule, grading, honor code (incl. AI tools).
- Define **three challenges** adapted to cloud/SOA: *decomposition (services), data (pipelines/traces), evaluation (SLOs + tests)*.

### Part 1 — Cloud & service foundations (compressed)
Keep/reshape:
1. Cloud fundamentals + service models (SOA language: contracts, APIs, multi-tenant)  
2. Distributed systems essentials (failure, consistency — thin but sharp)  
3. Containers → K8s → (optional) serverless  
4. Cloud providers + IaC (Terraform) + security basics  

**Compress or make optional electives:** deep Hadoop MapReduce history; large Spark MLlib blocks — keep Spark SQL/streaming only if program still needs big-data hours.

### Part 2 — Engineering practice (CS 329Z-inspired)
- **Lab track:** build a small multi-service app **without** a PaaS magic button, then redeploy with Docker/K8s/Terraform.  
- **Eval track:** define success metrics (latency, cost, correctness tests) before scaling features.  
- **HW oral quiz:** 10 minutes defending design tradeoffs.

### Part 3 — Bridge module (Option B): Agentic systems on cloud
Map CS 329Z → IUH hours (suggested 4–6 lectures + 1 HW):

| CS 329Z idea | IUH framing |
|--------------|-------------|
| Compound AI / agents | Services that call LLMs + tools + datastores |
| RAG from scratch | Retrieval service + embedding store on cloud |
| Tool use / MCP | API tools as SOA boundaries; sandboxing |
| Agent loop | Orchestrator service; retries; idempotency |
| Memory | Object store / DB as externalized memory |
| Evaluation 4-tuple | Request / staging env / stop rules / scorer for agents |
| Safety | Prompt injection ≈ untrusted input across trust boundaries |
| Observability | Tracing, cost caps — same as cloud ops |

Project theme analog to “Making life at Stanford better”:  
**“Making campus / SME ops better with cloud-backed agents”** (IUH / local business).

---

## 5. Concrete deliverables & phased plan

### Phase 0 — Hygiene (1–2 weeks) — **do first**
- [ ] Align every `contents/en/chapterXX/index.html` `title` with actual `_posts/` topics (see AGENTS.md mismatch table).  
- [ ] Publish `/syllabus` page: objectives, schedule, grading, prerequisites, AI honor code.  
- [ ] Add empty stubs: `assignments/`, `project/`, `schedule.md`.  
- [ ] Fix contribution docs still mentioning optimization-course leftovers if any.  
- [ ] Decide Option A vs A+B with department.

**Exit:** Student can trust the sidebar and know how they will be graded.

### Phase 1 — Assessment spine (2–3 weeks design)
- [ ] **HW1 — Cloud service harness (from scratch):** e.g. minimal multi-service TODO/notes API (auth + storage + worker) run locally with Compose; no Kubernetes yet. Written reflection on decomposition.  
- [ ] **HW2 — Evaluate & harden:** given a buggy/deployed stack (or agent), students write tests + SLO checks + one “judge” (script or LLM-as-judge) + error analysis.  
- [ ] **Oral quiz** after each HW (rubric: correctness, tradeoffs, what they’d measure in prod).  
- [ ] **Project milestones:** Week 3 proposal (1p) → midterm prototype demo → final demo day. Theme list published.

**Exit:** Grading weights draft (example): Project 40% · HW 30% · Quizzes 20% · Participation/VI reading 10%.

### Phase 2 — Curriculum reshape (1 term prep)
- [ ] Rewrite Part 1 outline (14 → ~10 core chapters + electives).  
- [ ] Thicken thin chapters OR merge (e.g. MapReduce history → one “batch processing” lecture).  
- [ ] Every required lecture ends with **Learning goals + 1 lab checkpoint**.  
- [ ] Continue Vietnamese for Ch02–Ch05 (priority for IUH delivery).

### Phase 3 — Agent-on-cloud bridge (optional module)
- [ ] 4 lectures: RAG service · tool-calling worker · eval harness · deploy+guardrails on K8s/serverless.  
- [ ] One HW: “internal campus assistant” (echo CS 329Z HW1 spirit) **after** students already know APIs/containers.  
- [ ] Reading pack: Zaharia compound AI blog; Anthropic “Building Effective Agents”; one RAG paper; MCP overview (short).

### Phase 4 — Site & ops polish
- [ ] Schedule table like CS 329Z (week / date / lecture / materials).  
- [ ] Deadlines table with timezone (Asia/Ho_Chi_Minh).  
- [ ] “Lecture materials linked as released” discipline.  
- [ ] Optional: paper-video lite (5–7 min) on a cloud or agent paper for bonus.

---

## 6. Suggested semester sketch (15 weeks, A+B light)

| Week | Focus | CS 329Z echo |
|------|-------|--------------|
| 1 | Cloud + SOA framing; course challenges | Landscape / decomposition |
| 2 | Distributed essentials + APIs as contracts | Builder mindset |
| 3 | Containers; **HW1 out**; **project proposal** | From-scratch build starts |
| 4–5 | Compose → K8s basics | Orchestration |
| 6 | IaC + cloud providers; **HW1 due + oral** | Tradeoff defense |
| 7 | Security & tenancy; midpoint demos | Safety / HITL lite |
| 8 | Data platforms compressed (Spark/NoSQL elective depth) | Data flywheel lite |
| 9 | **Module B:** RAG + retrieval service | RAG lecture |
| 10 | Tools, agent loop, memory on cloud storage | Tool use / memory |
| 11 | **HW2:** eval suite (4-tuple adapted) | Evaluation block |
| 12 | Observability, cost, guardrails | Production / safety |
| 13 | Project polish | — |
| 14–15 | Final demos | Demo day |

---

## 7. Mapping table — CS 329Z mechanism → IUH action

| CS 329Z | IUH action | Priority |
|---------|------------|----------|
| Build from scratch then frameworks | HW1 without K8s/PaaS; later redeploy with K8s | P0 |
| Eval as core skill | HW2 + project must include measurable success | P0 |
| Oral quiz on design | 10-min HW defense | P0 |
| Project demos > long papers | Midpoint + final demo; 1-page reports | P0 |
| 4-tuple eval framework | Adapt to services/agents in Module B | P1 |
| Safety & sandboxing | Trust boundaries, secrets, injection for tools/LLM | P1 |
| Paper video | Optional bonus | P2 |
| Full agent theory (DSPy, multi-agent, proactive) | Only if Option B expanded | P2 |

---

## 8. Risks & constraints
- **Credit hours / CDIO accreditation:** any rename or large Module B needs bộ môn buy-in.  
- **Compute cost:** LLM labs need quota plan (local small models / campus keys / student keys).  
- **Staff load:** oral quizzes and demos are expensive — schedule studio slots.  
- **Student prep:** IUH cohort may lack NLP background — keep Module B applied and scaffolded.

---

## 9. Success metrics (after one offering)
- ≥80% teams show a **working midpoint demo**.  
- HW oral rubrics show students can name **2 real tradeoffs** (cost vs latency, consistency vs availability, etc.).  
- Sidebar/content mismatch = **0**.  
- Student survey: “I can deploy and evaluate a multi-component system” ≥ 4/5.

---

## 10. Immediate next steps (this week)
1. Approve vision: **A** or **A+B**.  
2. Patch all chapter `index.html` titles (Phase 0).  
3. Draft syllabus page + grading weights.  
4. Spec HW1 one-pager (repo template + rubric).  
5. Optional: kick Teaching assistant / cloud agent to implement Phase 0 PR on `nglelinh/service-oriented-architecture-and-cloud-computing-iuh`.

---

*Plan authored for Nguyen Le Linh by Learning assistant, based on CS 329Z public syllabus (Autumn 2026–27 materials as published on cs329z.stanford.edu) and local course repo inventory on 2026-09-18.*
