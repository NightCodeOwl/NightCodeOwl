# 🦉 Production Engineering Portfolio

> Distributed systems, data pipelines, and infrastructure automation built at enterprise scale — described at the design-pattern level. Production code remains in private employer repositories.

**Deepanshu Maheshwari** · Software Engineer III @ Walmart Global Tech · Bengaluru
SDE I & II (Jan 2022 – Jul 2024) → SDE III (Aug 2024 – Present) · International SEO Platform

[![Java](https://img.shields.io/badge/Java-Expert-ED8B00?style=flat&logo=openjdk)](/)
[![Scala](https://img.shields.io/badge/Scala-Expert-DC322F?style=flat&logo=scala)](/)
[![Spring_Boot](https://img.shields.io/badge/Spring_Boot-Expert-6DB33F?style=flat&logo=springboot)](/)
[![Spark](https://img.shields.io/badge/Spark-Expert-E25A1C?style=flat&logo=apachespark)](/)
[![Kafka](https://img.shields.io/badge/Kafka-Expert-231F20?style=flat&logo=apachekafka)](/)
[![Cassandra](https://img.shields.io/badge/Cassandra-Advanced-1287B1?style=flat&logo=apachecassandra)](/)
[![GraphQL](https://img.shields.io/badge/GraphQL-Advanced-E10098?style=flat&logo=graphql)](/)
[![GCP](https://img.shields.io/badge/GCP-Advanced-4285F4?style=flat&logo=googlecloud)](/)
[![LLMs](https://img.shields.io/badge/LLM_Integration-Advanced-412991?style=flat&logo=openai)](/)

---

## Impact at a Glance

| Metric | Value |
|--------|-------|
| Daily Users Served (AI-generated content) | **1M+** |
| Daily Transactions | **10M+** |
| URLs Validated Pre-Publish | **2M+** |
| Items Translated (en-CA → fr-CA) | **1.28M+** |
| Interlink Records Published to Cassandra | **200M+** |
| Weekly Keywords Processed | **500K+** |
| Producer-Consumer Throughput Gain | **300×** |
| System Uptime | **99.95%** |
| Business Value Generated | **$2M+** in organic traffic |
| Annual Cost Savings | **$550K+** |
| Production Storage Freed | **~2TB** |

---

## Featured Projects

### 1. 🌐 Keyword Datalake & Topic Page Generation Pipeline
> End-to-end 7-step Spark/Scala pipeline transforming search keyword signals into production SEO pages

**Problem:** Manual keyword analysis and topic-page creation was unscalable for an enterprise SEO platform. Needed an automated pipeline ingesting 500K+ keywords/week, filtering through quality validators, enriching via search APIs, deduplicating via ML, and publishing to the production datastore.

**Solution:**
- **7-step Dataproc workflow**: source ingestion → keyword processing → search-response enrichment → keyword dedup → page dedup → metadata → topic-page generation
- **NLP enrichment**: Stanford CoreNLP for lemmatization and POS tagging; OpenNLP for sentence detection; heuristic media/local/info intent classification
- **8 configurable validators** sourced from GCP Secret Manager: existing-assortment check, rank threshold, keyword intent, minimum search volume, intent scope, retail intent, offensive-keyword filter (GCS-backed blocked list), media type
- **ML-based deduplication** via an internal ML platform (replaced BERT with an internal model, cutting batch time 6hr → 2.4hr)
- **Search API enrichment** via Spark `mapPartitions` batched per partition (replaced a shuffle-heavy join)
- **Cassandra publish** with composite key + LCS compaction
- **50+ executor** auto-scaling Dataproc workflow templates
- **Multi-competitor data parsing**: Strategy-pattern parser handling feeds for 7 competitor retailers

**Technologies:** Scala 2.12, Apache Spark, GCP Dataproc, Stanford CoreNLP, OpenNLP, Apache Cassandra, GCP Secret Manager, BigQuery

**Impact:** Processes **500K+ keywords weekly** → **35K production-grade SEO topic pages**; ~6-hour end-to-end runtime; **60% improvement** in batch runtime from the BERT migration

---

### 2. 🧬 Source-Agnostic ETL Pipeline Framework
> Modular data pipeline architecture with Strategy-pattern source abstraction and LLM integration

**Problem:** Multiple data sources (BigQuery, GCS, Hive, REST APIs) each required a different ingestion code path. New sources meant copy-paste-modify of the entire pipeline. Cross-project BigQuery moves hit DML restrictions.

**Solution:**
- **Strategy-pattern source abstraction**: `DataSourceReader` interface + factory with pluggable implementations (BigQuery, GCS, future Snowflake/Kafka)
- Source type switches via runtime config; zero code changes to add new sources
- Preserved the target Cassandra schema across all input sources — downstream services unaware of source variation
- **Cross-project BigQuery DML pattern**: staging-table NDJSON load → same-project MERGE to bypass BigQuery's cross-project DML restriction
- Delta semantics via processed / translated / retry-count flags for incremental processing
- **LLM-generated image alt text** — productionized LLM outputs (OpenAI API, LangChain) as SEO-optimized alt text across 100K+ product images
- Extended the downstream schema to serve structured alt-text arrays (asset ID + text) through a federated GraphQL gateway
- Checkpoint recovery for batch processing
- LLM cost modeling (per-request inference vs. vector DB) to drive build-vs-buy decisions

**Technologies:** Java, Apache Spark, Scala, SQL, BigQuery, GCS, OpenAI API, LangChain, GraphQL

**Impact:** Processes **100K+ products daily** with 99.9% reliability; shipped AI-generated content to production item pages supporting WCAG accessibility compliance; new sources plug in without ETL changes

---

### 3. 🔄 Kafka-Based Async Translation Pipeline
> Event-driven Spring Boot service replacing a rate-limited synchronous REST dependency with a producer-consumer pattern

**Problem:** The per-record REST integration with the internal translation platform was rate-limited, fragile (any downstream blip cascaded back), and locked throughput to translation-API latency. Needed to translate 1.28M items en-CA → fr-CA.

**Solution:**
- Spring Boot 3.2 service with a 3-stage pipeline: ingestion → orchestrator (batches 1,000 records → Kafka) → `@KafkaListener` consumer upserting fr-CA output into BigQuery
- Managed Kafka with mTLS, keystores loaded from GCP Secret Manager
- Idempotent upserts via composite tracking flags, so retries and replays are safe
- Cross-project BigQuery DML workaround: staging-table NDJSON load → same-project MERGE
- Dead-letter handling with capped exponential-backoff retries
- Decoupled scaling: producer and consumer scale independently; translation latency no longer blocks the producer

**Technologies:** Java 17, Spring Boot 3.2, Apache Kafka (mTLS), Google BigQuery, GCP Secret Manager, GCP Dataproc

**Impact:** **300× throughput improvement** over the per-record baseline; **1.28M items translated** en-CA → fr-CA in ~5–6 hours of processing; eliminated synchronous downstream coupling and enabled future languages without service changes

---

### 4. 🔗 200M+ Record Interlinking Publisher
> Distributed Cassandra publisher for ML-generated internal-link recommendations

**Problem:** Millions of web pages had no programmatic interlinking. The ML team produces 200M+ recommendation records per refresh that must land in production Cassandra without OOMs or schema mismatches.

**Solution:**
- **Scale**: Scala/Spark 3.4 publisher with spark-cassandra-connector handling 200M+ records per run
- **Schema split**: 3 Cassandra schemas covering 5 page types (browse/category, product, topic/brand)
- **OOM prevention**: sequential file processing for the 200M+ product tier instead of full in-memory aggregation, with bounded in-flight writes
- **Batch writes**: 100K-record chunks tuned for coordinator pressure
- **Auto-date detection**: latest GCS date folder picked up automatically
- **Third-party brand exclusion filter** for interlink quality
- **Refresh trigger**: polling-based trigger shipped first; event-driven trigger scoped as the next iteration

**Technologies:** Scala 2.12, Apache Spark 3.4, Apache Cassandra 4.x, GCS, spark-cassandra-connector

**Impact:** Interlinking live across browse, category, product, topic and brand pages; sequential processing eliminated OOMs at 200M+ scale

---

### 5. 🚦 GraphQL Serving with DGS BatchLoader
> Coalescing N-to-1 read pattern for federated GraphQL over multi-tenant Cassandra

**Problem:** GraphQL endpoints serving interlink recommendations made N independent Cassandra reads per request, breaching the p99 latency SLO at peak QPS.

**Solution:**
- `DataFetcher<CompletionStage<T>>` + `BatchLoader` pair on Netflix DGS, with concurrent partition-local fan-out
- Custom key type carrying request ID, market parameters and request metadata for tracing across the batch
- All per-record lookups in a request coalesced into batched Cassandra reads
- **Production OOMKill root cause**: heap-dump and GC analysis showed payload growth, not a memory leak; fixed by tuning JVM heap (4Gi → 8Gi), container limits, probes and autoscaling
- Staged CPU rollout on the serving tier as traffic ramped
- Federated gateway schema change exposed new fields without breaking existing consumers
- Runbook covering schema, BatchLoader caching semantics, failure modes and on-call playbook

**Technologies:** Java 17, Spring Boot, Netflix DGS, GraphQL Federation, Apache Cassandra, Istio, Kubernetes

**Impact:** Cut Cassandra read amplification per request by orders of magnitude; **p99 within SLO at 2× peak QPS** validated by load test; serving 5 page types to 1M+ daily users

---

### 6. 💾 Multi-Tenant Cassandra Schema & Bulk Loader
> High-performance write layer with composite-key schema design for multi-tenant serving

**Problem:** The production Cassandra write path hit bottlenecks at a peak of 10M daily transactions, causing data-loss incidents. The V1 schema couldn't support per-locale rows across tenants without secondary indexes.

**Solution:**

**Schema Design:**
- Composite partition key `(key, locale, tenant_id)` for single-partition lookups — no secondary indexes or scatter-gather
- Any-tenant / any-locale slices served from one table
- LCS compaction tuned for a read-heavy workload
- Metrics annotations (`@Counted`, `@Timed`, `@ExceptionCounted`) with Dynatrace dashboards

**Bulk Loader:**
- Diagnosed the write bottleneck as **client-side overhead**, not cluster capacity
- Prepared-statement reuse (~90% cache hit rate), batch-size tuning and connection-pool reconfiguration
- Batch inserts and async writes; retries with exponential backoff and dead-letter queuing
- `ALLOW FILTERING` anti-pattern detection and removal
- Backup-table auditing and cleanup automation

**Technologies:** Java, Apache Cassandra 4.x, Connection Pooling, LCS Compaction, Dynatrace

**Impact:** **45% write throughput improvement**; eliminated data-loss incidents; freed **~2TB** of unused production storage; **<100ms page loads** for 1M+ daily visitors; new tenants onboard via config, not schema change

---

### 7. ✅ Data Validation & Pre-Publish URL Validation
> Configurable data-quality checks and fault-tolerant URL validation for high-volume pipelines

**Problem:** Ingestion at 500K+ records/week needed multi-stage validation with zero tolerance for bad data reaching production, and 2M+ URLs had to be verified before publishing.

**Solution:**
- Parallel validation across configurable worker threads
- 15+ validation types: required fields, regex, SQL-injection detection, profanity filtering (ML-based), search-volume thresholds, assortment checks
- Pluggable validator architecture — new checks without touching the core pipeline
- **Pre-publish URL validation for 2M+ URLs**: probes behind a **circuit breaker** to protect downstream services
- BigQuery audit logging, automated remediation and Slack on-call alerting

**Technologies:** Python, Java, Threading, Circuit Breaker, BigQuery, Slack alerting

**Impact:** 500K+ records weekly narrowed to ~50K high-value targets at 99% accuracy; 2M+ URLs validated before publish

---

### 8. ⚡ Spark Optimization Toolkit
> Performance tuning for large-scale Spark jobs

**Problem:** Spark jobs over 500K+ records hit 6-hour batch times and inefficient cluster usage.

**Solution:**
- Query-plan analysis and cost-based optimization
- Dynamic partition tuning for data-aware parallelism
- Skew handling via salting and adaptive query execution
- Broadcast joins for small-table joins
- Cache management strategies
- Dataproc workflow templates with autoscaling

**Technologies:** PySpark, Scala, GCP Dataproc

**Impact:** **60% faster** (6h → 2.4h); autoscaling saved **$30K/month** in compute

---

### 9. 📊 Monitoring & Alerting System
> Real-time observability across 15+ production services

**Problem:** Services across multiple markets (US, CA) needed unified monitoring with fast incident response.

**Solution:**
- Metrics collection and aggregation across distributed services
- Alert deduplication to reduce noise
- Multi-channel notifications (Slack, email, PagerDuty)
- SLA tracking with uptime reporting and anomaly detection
- Centralized log querying with OpenObserve

**Technologies:** Python, Prometheus, Grafana, Dynatrace, OpenObserve

**Impact:** Incident response cut **85%** (30 min → 5 min); contributed to **99.95% uptime**

---

### 10. 🚀 CI/CD Automation Suite
> Multi-stage deployment pipelines with zero-downtime releases

**Problem:** Manual deployments took 4 hours with a 15% error rate, too slow for 7+ services shipping weekly.

**Solution:**
- Pipeline orchestration: build → test → security scan → deploy → validate
- Blue-green and canary deployments with automatic rollback
- Automated test gates with coverage checks
- Integrated SAST / security scanning
- **Expand-and-contract schema changes** for rollback-safe releases
- Slack pass/fail notifications with deployment metadata
- Auto-generated change-request tooling for routine production releases

**Technologies:** Java, Bash, Docker, Kubernetes, GitHub Actions, enterprise CI/CD platforms

**Impact:** **75% faster** deployments (4h → 1h); **manual deployment errors eliminated**

**🏆 Recognition:** Q2 2025 Bravo Award

---

### 11. 🌅 Sunset Automation System
> Health-check-driven page lifecycle management

**Problem:** Thousands of stale, redirected or zero-traffic pages degraded site quality and wasted crawl budget.

**Solution:**
- URL health-check pipeline: 302/404 detection, redirect-chain validation, assortment checks
- Sunset-candidate tracking: date flagged, redirection, low assortment, zero clicks
- BigQuery output for analytics pickup
- Automatic sitemap removal with audit logging
- Rate-limited execution to protect downstream services
- Lifecycle: flag → validate → business approval (301) → sitemap removal → cleanup

**Technologies:** Java, BigQuery, Cassandra

**Impact:** **2,000+ stale URLs** removed monthly; recovered **15% of lost organic traffic** by consolidating 5,000+ duplicate pages via 301s

---

### 12. 🔒 Zero-Downtime JDK Migration & Security Hardening
> Migrating distributed Java services across major JDK versions and clearing security findings

**Problem:** 7 production services ran on JDK 8 with accumulating vulnerabilities; the platform repo carried open SAST findings.

**Solution:**
- Service-by-service migration **sequenced by blast radius** using dependency-graph analysis
- Automated scanning for deprecated APIs and removed modules
- Staged rollout: non-prod → **canary bake** → rolling prod deploys with automated rollback triggers
- Post-migration validation: logs, error-rate comparison, performance benchmarks
- **Cleared SAST findings across the platform repo**: command injection, SQL injection, log sanitization, untrusted deserialization

**Services migrated:** 7 services spanning data publishing, page generation, page-performance analytics, metadata and data ingestion

**Technologies:** Java (JDK 8 → 17), SAST tooling, CI/CD

**Impact:** **7 services migrated with zero downtime** and no production incidents; security findings remediated

---

## Performance Metrics Summary

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Translation Throughput | Per-record REST | Batched Kafka (1,000/batch) | **300×** |
| GraphQL Read Amplification | N reads per request | Batched reads | **N-to-1 coalescing** |
| p99 Latency at 2× Peak | Breaching SLO | Within SLO | **SLO held** |
| Deployment Time | 4 hours | 1 hour | **75% reduction** |
| Incident Response | 30 minutes | 5 minutes | **85% reduction** |
| Batch Processing | 6 hours | 2.4 hours | **60% reduction** |
| Write Throughput | Baseline | Optimized | **45% improvement** |
| System Uptime | 99% | 99.95% | **10× fewer incidents** |
| Manual Deploy Errors | 15% | 0% | **Eliminated** |
| Page Load Time | Variable | <100ms | **Consistent sub-100ms** |
| Storage Waste | ~2TB unused | 0 | **2TB freed** |
| Monthly Compute Cost | Baseline | Optimized | **$30K/month saved** |
| Annual Infra Cost | Baseline | Optimized | **$550K+ saved** |

---

## Architecture Patterns

### Scalability
- Horizontal scaling with distributed processing (50+ Spark executors)
- Kafka producer-consumer pipelines decoupling throughput from downstream latency
- BatchLoader-coalesced reads with partition-local fan-out on the GraphQL layer
- Sequential processing with bounded in-flight writes for 200M+ record Cassandra loads
- Connection pooling and prepared-statement reuse on write paths
- Strategy-pattern source abstraction for new data sources without code changes
- Autoscaling clusters and pods based on workload

### Reliability
- Circuit breakers for failure isolation
- Retries with exponential backoff and dead-letter queues
- Idempotent upserts with composite tracking flags
- Adaptive throttling to protect downstream services
- Checkpoint-based pipeline recovery
- Health checks and validation gates at every deployment stage
- Zero-downtime strategies: canary, blue-green, expand-and-contract, automated rollback

### Performance
- Multi-tenant composite partition keys avoiding secondary indexes
- LCS compaction tuned for read-heavy Cassandra workloads
- Prepared-statement caching (~90% hit rate)
- Query-plan analysis and cost-based optimization in Spark
- Cross-project BigQuery DML via staging-table NDJSON + same-project MERGE
- JVM heap and container tuning driven by heap-dump/GC analysis
- Proactive storage auditing and cleanup

---

## Key Learnings

> "Not every OOMKill is a leak. Heap dumps showed our memory was growing with payload size — the fix was capacity and limits, not a code hunt."

> "Most 'database is slow' problems I've seen were client-side: statement preparation, batch sizing and pool configuration. Measure before you scale the cluster."

> "Replacing a synchronous REST call with a Kafka producer-consumer pair is one of the highest-leverage changes in a backend system. The producer stopped blocking on downstream latency, and throughput went up 300×."

> "Idempotency is what makes event-driven systems safe to operate. If every message can be replayed without side effects, retries stop being scary."

> "The right composite partition key is worth more than any read optimization. `(key, locale, tenant_id)` let one table serve any tenant or locale slice without secondary indexes."

> "Zero-downtime migrations aren't about being careful — they're about automated rollback, staged rollouts, and knowing your dependency graph cold."

> "Every manual step is a failure point. Automating deployments cut release time by 75% and removed manual errors entirely."

---

## Technology Stack

| Category | Technologies | Proficiency |
|----------|-------------|-------------|
| **Languages** | Java 17, Scala 2.12, Python, SQL, Bash | Expert |
| **Backend Frameworks** | Spring Boot 3.2, Netflix DGS (GraphQL), Kafka clients | Expert |
| **Streaming** | Apache Kafka (mTLS), producer-consumer patterns | Expert |
| **Big Data** | Apache Spark, GCP Dataproc, Hive, Trino, Airflow | Expert |
| **Databases** | Apache Cassandra (composite-key design, LCS), BigQuery, PostgreSQL, Azure Cosmos DB | Advanced |
| **APIs** | REST, GraphQL (DataFetcher / BatchLoader / Federation) | Expert |
| **Cloud** | GCP (Dataproc, BigQuery, GCS, Secret Manager), Azure (Cosmos DB, Blob) | Advanced |
| **Infrastructure** | Docker, Kubernetes, Istio, CI/CD, GitHub Actions | Advanced |
| **AI/ML** | LLM productionization (Claude, GPT-4o, Gemini), OpenAI API, LangChain, BERT, Hugging Face, Stanford CoreNLP, OpenNLP | Advanced |
| **Monitoring** | Prometheus, Grafana, Dynatrace, OpenObserve | Advanced |

---

## What This Demonstrates

✅ **Production-grade engineering** — reliability, error handling and observability from day one

✅ **Scale** — 200M+ record loads, 10M+ daily transactions, 1M+ daily users

✅ **Performance** — 300× throughput on Kafka, N-to-1 read coalescing on GraphQL, 45% write throughput on Cassandra

✅ **Production debugging** — root-causing OOMKills and write bottlenecks from heap dumps, GC logs and client metrics

✅ **AI integration** — LLM outputs (Claude, GPT-4o, Gemini) shipped to production at enterprise scale

✅ **Security** — SAST remediation and zero-downtime migrations

✅ **Business impact** — $550K+ annual savings, $2M+ in organic traffic value

---

## Contact

- 💼 [LinkedIn](https://www.linkedin.com/in/deepanshuu-maheshwari/)
- 📧 [deepanshuumaheshwari@gmail.com](mailto:deepanshuumaheshwari@gmail.com)
- 🐙 [GitHub](https://github.com/NightCodeOwl)

---

> *This portfolio describes systems at the architecture and design-pattern level. No proprietary code, data, internal system names or confidential business logic from any employer is included.*
