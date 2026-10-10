# 🦉 Production Engineering Portfolio

> Backend services, distributed data pipelines and platform migrations, described at the design level. Production code stays in private employer repositories.

**Deepanshu Maheshwari** · Software Engineer III @ Walmart Global Tech · Bengaluru
SDE I & II (Jan 2022 – Jul 2024) → SDE III (Aug 2024 – Present) · International SEO Platform
🌐 [https://nightcodeowl.github.io/NightCodeOwl/](https://nightcodeowl.github.io/NightCodeOwl/)

[![Java](https://img.shields.io/badge/Java_17-ED8B00?style=flat&logo=openjdk)](/)
[![Scala](https://img.shields.io/badge/Scala-DC322F?style=flat&logo=scala)](/)
[![Spring_Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat&logo=springboot)](/)
[![Kafka](https://img.shields.io/badge/Kafka-231F20?style=flat&logo=apachekafka)](/)
[![Spark](https://img.shields.io/badge/Spark-E25A1C?style=flat&logo=apachespark)](/)
[![Cassandra](https://img.shields.io/badge/Cassandra-1287B1?style=flat&logo=apachecassandra)](/)
[![GraphQL](https://img.shields.io/badge/GraphQL-E10098?style=flat&logo=graphql)](/)
[![GCP](https://img.shields.io/badge/GCP-4285F4?style=flat&logo=googlecloud)](/)

---

## At a Glance

| Metric | Value |
|--------|-------|
| Catalog items translated over async Kafka | **1.28M** |
| Records per Cassandra publish run | **200M+** |
| AI-generated alt-text records live for Canada | **3.2M** |
| Publishable pages restored after a pipeline collapse | **199,917** (from 4.1M input records) |
| Eligible topic-page keywords after eligibility analysis | **221,618 → 435,846** |
| Market pipelines unified into one config-driven job | **3 → 1** (CA / MX / US) |
| Services migrated JDK 8 → 17 with zero downtime | **7** (8 merged PRs) |
| Production Cassandra storage reclaimed | **~2TB** |
| URLs validated pre-publish per sitemap run | **111K+** |

---

## Featured Systems

### 1. 🔐 Zero-Downtime Kafka Identity Migration
> Moving a production Kafka service from certificate-based mTLS to SPIFFE Workload Identity

**Approach:**
- Three reversible phases: **prep → enablement → cleanup**, each independently rollback-safe
- Coordinated Kafka ACLs and permissions with the platform team
- Tuned Istio mTLS and per-environment egress rules

**Technologies:** Kafka, SPIFFE / Workload Identity, Istio, Kubernetes

**Outcome:** Cut over with **zero downtime**.

---

### 2. 🌎 One Pipeline for Three Markets
> Replacing per-market alt-text code paths with a single config-driven Spark job

**Approach:**
- Sole owner of the epic: authored the design, objective and success criteria
- Replaced three per-market code paths (Canada, Mexico, US) with one **configuration-driven Spark job**
- Phased rollout: **dual-write → shadow validation → cutover → legacy decommission**

**Technologies:** Java, Apache Spark, BigQuery, Cassandra

**Outcome:** Adding a new market is a **configuration change only**.

---

### 3. 🔄 Async Translation Service
> Replacing a rate-limited synchronous REST dependency with Kafka messaging

**Problem:** A per-record REST integration with the internal translation platform was rate-limited and tied throughput to API latency.

**Approach:**
- Java 17 / Spring Boot service built as **three independently runnable phases**: BigQuery ingest → async Kafka produce → consume and `MERGE` back to BigQuery
- Bulk record locking and batched attribute reads
- Cross-project BigQuery staging, then a same-project `MERGE` to work around cross-project DML limits
- mTLS keystores loaded from GCP Secret Manager
- Completion tracking and graceful shutdown so any phase can be re-run safely
- Deployed as a Dataproc workflow

**Technologies:** Java 17, Spring Boot, Apache Kafka (mTLS), BigQuery, GCP Secret Manager, Dataproc

**Outcome:** **1.28M catalog items** sent for English → French localization.

---

### 4. 🚦 GraphQL Content Serving with DGS BatchLoader
> Delivering AI-generated image alt text to item pages through federated GraphQL

**Approach:**
- Spark ingestion from BigQuery and Hive into Cassandra
- Netflix DGS `BatchLoader` data providers replace one Cassandra read per record with **concurrent, partition-local fan-out**
- Federated gateway schema exposes the new fields without breaking existing consumers
- **Production OOMKills:** heap-dump and GC analysis showed genuine payload growth, not a leak. Fixed by retuning JVM heap, container limits, probes and autoscaling, with CPU ramped **120m → 2300m** as traffic grew

**Technologies:** Java 17, Spring Boot, Netflix DGS, GraphQL Federation, Cassandra, Kubernetes, Istio

**Outcome:** **N+1 Cassandra reads eliminated**; **3.2M records** live for Canada; restart loops and throttling ended.

---

### 5. 💾 Multi-Tenant Cassandra Schema
> One table serving every tenant and locale

**Approach:**
- Composite partition key `(key, locale, tenant_id)`: every tenant/locale read is a **single-partition lookup**, with no secondary indexes or scatter-gather
- LCS compaction tuned for a read-heavy workload
- Repository-layer metrics surfaced in Dynatrace
- Audited and dropped unused production backup tables

**Technologies:** Java, Apache Cassandra, LCS compaction, Dynatrace

**Outcome:** **~2TB** of production storage reclaimed.

---

### 6. 🔗 200M+ Record Cassandra Publisher
> Landing ML-generated internal-link recommendations in production Cassandra

**Approach:**
- Scala / Spark 3.4 publisher on GCP Dataproc, scaling to **50 executors**
- Traced a write bottleneck to **client-side overhead, not cluster capacity**
- Fixed with prepared-statement reuse, batch tuning and **bounded in-flight writes** that protect read latency
- Sequential file processing so the job never holds the full set in memory
- Polling-based refresh trigger and third-party-brand exclusion

**Technologies:** Scala, Apache Spark 3.4, Cassandra, GCS, Dataproc

**Outcome:** **200M+ records** per run; interlinking live on Canada topic and brand pages.

---

### 7. 🔍 Finding the Missing Topic Pages
> Production diagnosis in a 4.1M-record Spark pipeline

**Approach:**
- Instrumented a **seven-stage validation funnel** with per-stage retention logging
- Traced the volume collapse to over-strict assortment and page-rank filters
- Ran a follow-up eligibility analysis and configuration change

**Technologies:** Java, Apache Spark, data validation, observability

**Outcome:** Output restored to **199,917 publishable pages**; eligible keywords grew from **221,618 to 435,846**.

---

### 8. 🔒 Zero-Downtime JDK 17 Migration & Security Hardening
> Seven production services off JDK 8, driven by security findings

**Approach:**
- Sequenced by **blast radius**, with rolling deploys
- Closed command-injection, SQL-injection, log-injection and unsafe-deserialization paths found by SAST scanning

**Technologies:** Java 17, Maven, SAST tooling

**Outcome:** **7 services** migrated in **8 merged PRs** with **zero downtime**.

---

### 9. 🚀 CI/CD and Pre-Publish Validation
> Release and data-quality tooling for the platform

- Owned CI/CD for a **10-module** Java/Scala monorepo: per-module Looper and Concord pipelines, Artifactory publishing, automated production deploys and Slack pass/fail gating
- Pre-publish sitemap URL validation checking **111K+ URLs per run**, with rate-limited, batched probes that flag 302/404 pages, remove them automatically and log to BigQuery

**Technologies:** Looper, Concord, Maven, Artifactory, Java, BigQuery

**🏆 Recognition:** Q2 2025 Bravo Award for automation innovation

---

### 10. 📢 Advertising Data Platform (SDE I & II)
> Airflow and Spark pipelines for the DSP and EA advertising products

- Built and productionized the DSP and EA advertising **Airflow** pipelines across **62 PRs**: dynamic DAG generation, dynamic Dataproc cluster sizing, execution timeouts and change-managed releases
- Refactored the **Spark** audience-segmentation service across **32 PRs**: retries, SFTP delivery, metadata integration, job metrics and Prometheus alerts to Slack
- Integrated CCM2 and AppVault across ASDA item, order and marketing **Scala/Spark** aggregations; set up GCP infrastructure as code (IAM, service accounts, VPC Service Controls)

**Technologies:** Airflow, Python, Apache Spark, Scala, Dataproc, Prometheus, GCP IAM

---

## Key Learnings

> "Not every OOMKill is a leak. Heap dumps showed memory growing with payload size. The fix was capacity and limits, not a code hunt."

> "Most 'database is slow' problems I've seen were client-side: statement preparation, batch sizing and in-flight limits. Measure before you scale the cluster."

> "Make every phase of a migration reversible on its own. Then zero downtime stops being luck."

> "The right composite partition key is worth more than any read optimization. `(key, locale, tenant_id)` let one table serve any tenant or locale without secondary indexes."

> "When output drops, instrument the funnel before touching the filters. Per-stage counts show you exactly where the data went."

---

## Technology Stack

| Category | Technologies |
|----------|-------------|
| **Languages** | Java 17, Scala, Python, SQL |
| **Backend** | Spring Boot, Netflix DGS (GraphQL), REST, Kafka clients |
| **Streaming & Batch** | Apache Kafka (mTLS), Apache Spark, Apache Airflow, GCP Dataproc |
| **Databases** | Apache Cassandra (composite-key design, LCS), BigQuery, Hive, PostgreSQL, Azure Cosmos DB |
| **Cloud & Platform** | GCP (Dataproc, BigQuery, GCS, IAM, VPC Service Controls, Secret Manager), Kubernetes, Istio, SPIFFE Workload Identity, Terraform |
| **CI/CD** | Looper, Concord, Maven, Artifactory |
| **Observability** | Prometheus, Dynatrace |

---

## Contact

- 🌐 [Portfolio website](https://nightcodeowl.github.io/NightCodeOwl/)
- 💼 [LinkedIn](https://www.linkedin.com/in/deepanshuu-maheshwari/)
- 📧 [deepanshuumaheshwari@gmail.com](mailto:deepanshuumaheshwari@gmail.com)
- 🐙 [GitHub](https://github.com/NightCodeOwl)

---

> *This portfolio describes systems at the architecture and design level. No proprietary code, data or confidential business logic from any employer is included.*
