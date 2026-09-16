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
