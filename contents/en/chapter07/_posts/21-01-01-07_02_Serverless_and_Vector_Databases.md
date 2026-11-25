---
layout: post
title: 07-02 Serverless, Distributed SQL, and Vector Databases (2022–2026)
chapter: '07'
order: 3
owner: Nguyen Le Linh
lang: en
categories:
- chapter07
lesson_type: optional
---

The required lesson covers CAP, BASE vs ACID, and the four NoSQL families. This optional note adds the stores that showed up beside those families in 2022–2026: **serverless** operational databases, **distributed SQL** that wants both ACID and scale, and **vector indexes** for RAG/AI applications.

## Learning objectives

- Place serverless Aurora/Cosmos/Firestore-style products on the IaaS–PaaS axis.
- Contrast “NoSQL for scale” with distributed SQL (Spanner-class) systems.
- Explain a vector index as a *new access path*, not a replacement for a system of record.

Between 2022 and 2026, two forces reshaped the database landscape so sharply that the classic "SQL vs. NoSQL" dichotomy is no longer enough. First, teams got tired of paying for idle primary databases at 3:00 AM and demanded **Serverless** and **Distributed SQL** engines (Aurora Serverless, CockroachDB, Spanner) that auto-scale cleanly while keeping ACID transactions. Second, generative AI and **RAG (Retrieval-Augmented Generation)** introduced a brand-new query type: *"find the 5 documents whose meaning is closest to this 1,536-dimensional embedding vector!"*—giving rise to **Vector Indexes** (HNSW, IVF).

## Key ideas: Serverless Elasticity, Distributed SQL, and Vector Search

1. **Serverless Databases (Decoupled Storage & Compute):** Separates stateless SQL compute nodes from a shared, auto-expanding distributed log/storage tier—allowing compute to scale smoothly from 0.5 ACUs to 128 ACUs (or scale to zero) in seconds.
2. **Distributed SQL (Raft/Paxos + Multi-Region ACID):** Systems like Spanner and CockroachDB shard relational tables horizontally like NoSQL while running **Raft/Paxos consensus** per shard and synchronized clocks (TrueTime/HLC) to preserve strict serializable SQL transactions!
3. **Vector Databases & Approximate Nearest Neighbors (ANN):** Stores dense semantic embeddings $$v \in \mathbb{R}^d$$ and accelerates similarity search (cosine / dot-product / $$L_2$$) via graph indexes like **HNSW** (*Hierarchical Navigable Small World*), trading tiny recall loss (98–99% recall) for 10,000× faster retrieval!

### Worked Example (Step by Step): Cosine Similarity and Vector Index Memory Math

**Part A — Exact cosine similarity by hand:** Compare a user query embedding $$q = (1, 0)$$ against two unit-normalized document embeddings $$d_1 = (0.8, 0.6)$$ and $$d_2 = (-0.6, 0.8)$$:

$$
\cos(q, d_1) = rac{1(0.8) + 0(0.6)}{1 \cdot 1} = \mathbf{+0.80} 	ext{ (strong match!)}, \qquad \cos(q, d_2) = rac{1(-0.6) + 0(0.8)}{1 \cdot 1} = \mathbf{-0.60} 	ext{ (opposite meaning)}.
$$

**Part B — Why brute-force search breaks at 10 million embeddings:** Suppose a RAG system indexes **$$N = 10{,}000{,}000$$** chunks using **1,536-dimensional** `float32` (4 bytes/float) vectors:
- Raw vector RAM footprint:
  $$
  10^7 	imes 1{,}536 	imes 4	ext{ bytes} pprox 6.144 	imes 10^{10}	ext{ bytes} pprox \mathbf{61.4	ext{ GB RAM}}!
  $$
- **Brute-force scan** computes $$10^7 	imes 1{,}536 pprox 1.54 	imes 10^{10}$$ multiply-adds per query (hundreds of milliseconds on CPU).
- **HNSW graph index** navigates layered skip-list-like neighborhoods in $$O(\log N)$$ hops—visiting only **~200–500 candidate vectors** and answering in **< 5 ms**!

### Explanation: Placing 2022–2026 Databases Inside Your Architecture

1. **Treat a vector index as a secondary access path, not your system of record:** keep authoritative text, permissions, and ACID billing in PostgreSQL (often using `pgvector` for < 10M vectors) or a dedicated store synchronized cleanly with your source of truth.
2. **Watch cold-start and cross-region consensus costs:** serverless databases save huge money on dev/spiky workloads but can incur brief wake-up latency; Distributed SQL gives global ACID at the cost of cross-AZ Raft round trips on every write.
3. **Quantize vectors when RAM dominates cost:** compressing `float32` (4 bytes) to `int8` (1 byte) or binary quantization cuts that 61.4 GB RAM bill by **4× to 32×** with minimal RAG recall loss!
4. **Always pair vector search with metadata filters (Hybrid Search):** combining BM25 keyword search + HNSW vector search + `tenant_id = 42` ACL filtering prevents both hallucinated matches and cross-tenant data leaks.

**Quick check.** If we apply `int8` scalar quantization (1 byte per dimension instead of 4) to our 10 million 1,536-dim vectors, how much RAM does the raw vector table require? $$61.4	ext{ GB} / 4 = \mathbf{15.36	ext{ GB}}$$—easily fitting on a single modest cloud node!

## 1. Serverless is elasticity applied to the database

NIST rapid elasticity used to mean “more VMs.” Serverless databases scale **capacity with connections and storage**, often with a scale-to-zero pause:

- Amazon Aurora Serverless v2 (ACU scaling), Google Cloud Spanner / AlloyDB, Azure Cosmos DB serverless, Fauna, Neon/PlanetScale-style Postgres branches.
- The CAP trade-off does not disappear: you still choose consistency vs. latency when a region dies.

```python
# Application view: pool settings must assume the DB can pause or autoscale
import os
import psycopg

dsn = os.environ["DATABASE_URL"]  # often a pooler (PgBouncer, RDS Proxy)

def fetch_order(order_id: str):
    with psycopg.connect(dsn, connect_timeout=5) as conn:
        with conn.cursor() as cur:
            cur.execute("select status from orders where id = %s", (order_id,))
            return cur.fetchone()
```

Cold starts move from the function (Chapter 08) to the **first query after pause**. Timeouts and pooling are now part of the data-access SOA contract.

## 2. Distributed SQL: ACID without a single primary (when it works)

CockroachDB, Yugabyte, Google Spanner, and similar systems market **serializable** (or strong) SQL over shards. They do not repeal CAP: they use consensus (Paxos/Raft) and clocks (TrueTime or hybrid logical clocks) so the *common* path looks like a single Postgres. The application win is: fewer handmade shards than 2015 MongoDB playbooks.

Use them when you need **multi-region writes** and SQL joins. Do not use them as a cheap cache.

## 3. Vector databases: the AI access path

RAG and embedding search added a fifth “shape” next to KV/document/column/graph: **approximate nearest neighbor (ANN)** over float vectors (`pgvector`, Pinecone, Weaviate, Milvus, MongoDB Atlas Vector Search, OpenSearch k-NN).

```sql
-- PostgreSQL + pgvector (illustrative)
CREATE TABLE docs (
  id uuid PRIMARY KEY,
  body text,
  embedding vector(1536)
);
-- ANN query: not a replacement for WHERE tenant_id = ...
SELECT id, body
FROM docs
WHERE tenant_id = 'iuh'
ORDER BY embedding <-> $1
LIMIT 8;
```

**Application rule:** the vector store is a *retrieval service*. The system of record (orders, grades, payments) stays in an ACID or carefully chosen NoSQL store. Mixing them in one unauthenticated index is how RAG leaks tenants.

<div class="content-box warning-box">
<p><strong>Filter then ANN, or ANN then filter?</strong> Pre-filters (tenant, ACL) must be in the index path or you will retrieve the neighbor and then drop it—or worse, return it. This is authorization, not just recall@k.</p>
</div>

## 4. How this maps to the required taxonomy

| Workload | 2020s default | CAP-ish note |
| --- | --- | --- |
| Session / cart | KV or Redis | AP-leaning cache |
| Product catalog | Document | Flexible schema |
| Ledger / enrollment | Distributed SQL or RDBMS | CP-leaning |
| Semantic search | Vector + metadata filters | Approximate |

## Challenges

- Serverless *min capacity* bills that surprise teams who believed “scale to zero is free.”
- Vector index rebuilds after embedding-model changes (a migration, not a reindex trivia).
- Multi-cloud replication of both SQL and vectors (two consistency stories).

## Video tham khảo

- **xem trước:** [What is a Vector Database? Powering Semantic Search & AI Applications](https://www.youtube.com/watch?v=gl1r1XV0SLw) — IBM Technology, English. Watch first: what a vector database is, the AI access path this optional lesson adds beside the four NoSQL families.
- [What is a serverless database? (in under 3 minutes)](https://www.youtube.com/watch?v=GxjKsKSOWJk) — CockroachDB, English. Serverless databases as capacity that scales with use, the elasticity section of the lesson.
- [What is Serverless?](https://www.youtube.com/watch?v=vxJobGtqKVM) — IBM Technology, English. What serverless means when the database, not only the function, can pause and scale.

## Exercises

1. Pick one campus system (library, LMS, parking). Which family + which 2020s product would you choose, and which CAP pain do you accept?
2. Write a three-step migration plan from “all embeddings in a JSON column” to `pgvector` with tenant filters.
3. Explain why a serverless DB plus a serverless function still needs a connection pooler.

## References

1. Amazon Aurora Serverless v2 user guide (ACU scaling): AWS documentation.
2. `pgvector` project: [github.com/pgvector/pgvector](https://github.com/pgvector/pgvector).
3. Google, *Spanner: Google’s Globally-Distributed Database* (OSDI 2012) — still the design ancestor of distributed SQL products.
4. MongoDB Atlas Vector Search docs; Pinecone / Weaviate product docs (compare ANN + metadata filters).
5. NIST SP 800-145 — elasticity and measured service still apply to *database* capacity, not only VMs.
