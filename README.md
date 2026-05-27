# Deepanshu Maheshwari's GitHub Profile

**Role:** Senior Software Engineer (SDE III) at Walmart Global Tech building end-to-end distributed systems for Walmart Canada's SEO platform

**What I build:** Spark/Scala batch pipelines on GCP Dataproc, GraphQL serving layers with DGS BatchLoader, Kafka-based event-driven microservices, multi-tenant Cassandra schemas, and source-agnostic ETL with Strategy-pattern abstraction.

**Key Accomplishments:**
- Owns 4 production systems serving **1M+ daily SEO users** — keyword datalake, topic-page automation, GenAI alt-text platform, ML-driven interlinking
- Architected 7-step Spark/Scala **Keyword Datalake** on GCP Dataproc with 50+ executors — 500K weekly keywords → 35K SEO topic pages
- Built **Spring Boot 3.2 + Kafka (TAAS, mTLS) async translation service** — **300× throughput improvement** over per-record baseline; translated 1.28M items en-CA → fr-CA
- Authored **GraphQL DGS BatchLoader pattern** coalescing N per-topic Cassandra reads into one batched read per request; p99 latency under SLO at 2× peak QPS
- Built **200M+ record Cassandra publisher** for ML-generated interlinks across 5 page types and 3 schemas
- Designed **multi-tenant composite partition keys** serving any-tenant/any-locale slice without secondary indexes
- Maintains **10M+ daily transactions** in production
- Recipient of **Q2 2025 Bravo Award** for automation innovation

**Technical Expertise:**
- **Languages:** Java 17, Scala 2.12, Python, SQL, Bash
- **Backend & Distributed Systems:** Spring Boot 3.2, Apache Kafka (TAAS, mTLS), Apache Spark, GraphQL (DGS DataFetcher / BatchLoader), REST APIs, Microservices, Event-Driven Architecture, Producer-Consumer Patterns
- **Data Systems:** Apache Cassandra (composite-key schema design, LCS compaction), Google BigQuery (cross-project DML), Hive, PostgreSQL, Azure Cosmos DB
- **Cloud & Infrastructure:** GCP (Dataproc workflow templates, GCS, BigQuery, Secret Manager), Azure (DF Cassandra), Docker, Kubernetes/WCNP, CI/CD (Looper, Concord, Jenkins, GitHub Actions)
- **AI/ML:** LLM productionization (Claude, GPT-4o, Gemini, Walmart-internal), BERT, Stanford CoreNLP, OpenNLP, LangChain; LLM cost modeling at 1M+ scale

**Notable Strengths:**
End-to-end ownership across batch pipelines, event-driven microservices, GraphQL serving layers, and multi-tenant data stores. Deep with distributed systems patterns: composite partition keys, BatchLoader coalescing, Strategy-pattern source abstraction, cross-project BigQuery DML workarounds, and Kafka producer-consumer integration. Production code at Walmart Global Tech remains in private repositories — see [PORTFOLIO_README.md](./PORTFOLIO_README.md) for sanitized, generic implementations of the patterns I build.

**Competitive Programming:** 500+ LeetCode problems solved, focusing on dynamic programming and graph algorithms

**Contact:** LinkedIn, email, and technical blog (coming soon) available through profile links
