# 🦉 Production Engineering Portfolio

> Generic, sanitized implementations of distributed systems, data pipelines, and infrastructure automation solutions built at enterprise scale.

[![Java](https://img.shields.io/badge/Java-Expert-ED8B00?style=flat&logo=openjdk)](/)
[![Scala](https://img.shields.io/badge/Scala-Expert-DC322F?style=flat&logo=scala)](/)
[![Spring_Boot](https://img.shields.io/badge/Spring_Boot-Expert-6DB33F?style=flat&logo=springboot)](/)
[![Spark](https://img.shields.io/badge/Spark-Expert-E25A1C?style=flat&logo=apachespark)](/)
[![Kafka](https://img.shields.io/badge/Kafka-Expert-231F20?style=flat&logo=apachekafka)](/)
[![Cassandra](https://img.shields.io/badge/Cassandra-Advanced-1287B1?style=flat&logo=apachecassandra)](/)
[![GraphQL](https://img.shields.io/badge/GraphQL-Advanced-E10098?style=flat&logo=graphql)](/)
[![GCP](https://img.shields.io/badge/GCP-Advanced-4285F4?style=flat&logo=googlecloud)](/)
[![OpenAI](https://img.shields.io/badge/OpenAI-Expert-412991?style=flat&logo=openai)](/)

---

## Impact at a Glance

| Metric | Value |
|--------|-------|
| Daily Transactions | **10M+** |
| Pages Processed Monthly | **2M+** |
| Items Translated (en-CA → fr-CA) | **1.28M+** |
| RELM Records Published to Cassandra | **200M+** |
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

**Problem:** Manual keyword analysis and topic-page creation was unscalable for an enterprise SEO platform. Needed an automated pipeline ingesting 500K+ keywords/week, filtering through quality validators, enriching via search APIs, deduplicating via ML, and publishing to production datastore.

**Solution:**
- **7-step Dataproc workflow**: SourceProcessor → KeywordProcessor → SearchResponseFetcher → KeywordDeDupProcessor → PageDeDupProcessor → MetaDataProcessor → TopicPageProcessor
- **NLP enrichment** (step 1): Stanford CoreNLP 4.4.0 for lemmatization and POS tagging; OpenNLP 1.9.3 for sentence detection; heuristic media/local/info intent classification
- **8 configurable validators** sourced from GCP Secret Manager: existing-assortment check, rank check (≤ 20), keyword intent, search volume threshold (≥ 100 MSV), intent scope, retail intent, offensive keyword filter (GCS-backed blocked list), media type
- **ML-based deduplication** via Walmart ML Platform workflows (replaced BERT with internal model, cutting batch time 6hr → 2.4hr)
- **External API integration**: Preso Search API enrichment via Spark `mapPartitions` batched per-partition (replaced shuffle-heavy join)
- **Cassandra publish** with composite key + LCS compaction
- **50+ executor** auto-scaling Dataproc workflow templates
- **Multi-competitor BrightEdge bucket parsing**: `CompetitorParser` Strategy pattern for 7 retailers (walmart_ca, amazon_ca, costco_ca, homedepot_ca, bestbuy_ca, metro_ca, instacart_ca)

**Technologies:** Scala 2.12, Apache Spark, GCP Dataproc, Stanford CoreNLP 4.4, OpenNLP 1.9, Apache Cassandra, GCP Secret Manager, BigQuery analytics sink

**Impact:** Processes **500K+ keywords weekly** → **35K production-grade SEO topic pages**; ~6-hour end-to-end runtime; powers entire CA topic page production end-to-end; **60% improvement** in batch runtime from BERT migration

---

### 2. 🧬 Source-Agnostic ETL Pipeline Framework
> Modular data pipeline architecture with Strategy-pattern source abstraction and AI integration

**Problem:** Multiple data sources (BigQuery, GCS, Hive, REST APIs) each required a different ingestion code path. New sources meant copy-paste-modify of the entire pipeline. Cross-project BigQuery moves hit DML restrictions.

**Solution:**
- **Strategy-pattern source abstraction**: `DataSourceReader` interface + `DataSourceReaderFactory` with pluggable implementations (`BigQueryReader`, `GcsReader`, future Snowflake/Kafka)
- Source type switches via runtime config (`source.type=bigquery|gcs`); zero code changes to add new sources
- Preserved target Cassandra schema across all input sources — downstream services unaware of source variation
- **Cross-Project BigQuery DML pattern**: staging-table NDJSON load → same-project MERGE workaround to bypass BQ's cross-project DML restriction
- Delta semantics via `has_processed` / `is_translated` / `retry_count` flag pattern for incremental processing
- **AI-powered image alt text generation** — integrated OpenAI API and LangChain for SEO-optimized alt text across 100K+ product images
- Extended downstream service schema to serve structured alt text arrays (asset ID + alt text string) via GraphQL gateway
- Optimized batch processing with checkpoint recovery
- Cost analysis module: GPT-4o at ~$0.0058/request, Vector DB at ~$1,096/month — used to drive build-vs-buy decisions

**Technologies:** Java, Apache Spark, Scala, SQL, BigQuery (cross-project DML), GCS, OpenAI API, LangChain, GraphQL

**Impact:** Processes **100K+ products daily** with 99.9% reliability; shipped AI-generated content to production item pages achieving WCAG accessibility compliance; new sources plug in without ETL changes

---

### 3. 🔄 Kafka-Based Async Translation Pipeline
> Event-driven Spring Boot service replacing synchronous REST dependency with producer-consumer pattern

**Problem:** Existing per-record REST integration with the internal translation platform was 300× too slow at scale, fragile (any downstream blip cascaded back), and locked translation throughput to translation-API latency. Needed to translate 1.28M items en-CA → fr-CA.

**Solution:**
- Spring Boot 3.2 service with 3-stage pipeline: `IngestionService` → `TranslationOrchestrator` (batches 1000 records → Kafka) → `KafkaConsumerService` (`@KafkaListener` upserts fr-CA into BigQuery)
- TAAS Kafka with mTLS/SSL via JKS keystores loaded from GCP Secret Manager
- Idempotent upsert handling via composite tracking flags (`has_processed` / `is_translated` / `retry_count`)
- Cross-project BigQuery DML workaround: staging-table NDJSON load → same-project MERGE pattern to bypass BQ's cross-project DML restriction
- Dead-letter handling with exponential backoff retry caps
- Decoupled scaling: producer and consumer scale independently; translation latency no longer blocks the producer

**Technologies:** Java 17, Spring Boot 3.2, Apache Kafka (TAAS, mTLS), Google BigQuery, GCP Secret Manager, GCP Dataproc

**Impact:** **300× throughput improvement** over per-record baseline; **1.28M items translated** en-CA → fr-CA in ~5–6 hours total; eliminated synchronous downstream coupling — enables future multi-language extensibility (es-MX, etc.) without service changes

---

### 4. 🔗 200M+ Record Interlinking Publisher
> Distributed Cassandra publisher for ML-generated internal-link recommendations at enterprise scale

**Problem:** Millions of web pages with no programmatic interlinking — manual link curation was unscalable. The ML team produces 200M+ recommendation records per refresh that need to land in production Cassandra without OOMs or schema mismatches.

**Solution:**
- **Scale**: Publisher (Scala 2.12, Spark 3.4.3, spark-cassandra-connector 3.4.1) handles 200M+ records per run
- **Schema split**: 3 distinct Cassandra schemas covering 5 page types — `BrowseCpSchema` (browse/category: `{id, banner, pageType, locale}`), `ItemSchema` (product: `{id, locale}`), `TopicBrandSchema` (topic/brand: `id` string)
- **OOM prevention**: Sequential file processing for product pages (the 200M+ tier) instead of full-set in-memory aggregation
- **Batch writes**: 100K-record chunks tuned for Cassandra coordinator pressure
- **Auto-date detection**: Latest GCS date folder picked up automatically — no manual config per run
- **3P brand exclusion filter**: rule-based interlink quality control
- **Polling trigger**: polling-based refresh mechanism deployed first; event-driven Cloud Functions trigger scoped for next iteration
- Cross-service implementation across shared commons and domain-specific services

**Technologies:** Scala 2.12, Apache Spark 3.4.3, Apache Cassandra 4.x, GCS, spark-cassandra-connector 3.4.1

**Impact:** Deployed interlinking across browse, category, product, topic, and brand pages in production; powers production internal-link graph at enterprise scale; sequential processing on product tier eliminates OOM at 200M+ scale

---

### 5. 🚦 GraphQL Serving with DGS BatchLoader
> Coalescing N-to-1 read pattern for federated GraphQL with multi-tenant Cassandra backing

**Problem:** GraphQL endpoints serving interlink recommendations on browse/category/product/topic/brand pages were doing N independent Cassandra reads per request, blowing past p99 latency SLO under peak QPS.

**Solution:**
- Authored a `DataFetcher<CompletionStage<T>>` + `BatchLoader<KeyType, ValueType>` pair using Netflix DGS framework
- Custom key type carrying `requestId`, market-specific params, and `GraphQLRequestMetadata` — enables tracing across the batch
- BatchLoader coalesces all per-record lookups within a single GraphQL request into a single batched Cassandra query
- **JVM heap tuned (4Gi → 8Gi)** to absorb in-memory payloads per request
- Probe timeouts, istio CPU (100m → 70m), and HPA targets retuned to handle new request shape
- Staged CPU rollout (120m → 200m → 1300m → 2300m) on downstream serving tier as traffic ramped
- Federated mGQL gateway schema delta submitted to expose new fields without breaking existing consumers
- Confluence runbook authored covering schema, BatchLoader caching semantics, failure modes, and on-call playbook

**Technologies:** Java 17, Spring Boot, Netflix DGS, GraphQL, Apache Cassandra, mGQL Federation, Istio, Kubernetes

**Impact:** Cut Cassandra read amplification by orders of magnitude per GraphQL request; **p99 latency under SLO at 2× peak QPS** validated via load test; deployed across 5 page types serving 1M+ daily SEO visitors

---

### 6. 💾 Multi-Tenant Cassandra Schema & Bulk Loader
> High-performance write layer with composite-key schema design for multi-tenant data serving

**Problem:** Production Cassandra cluster hit write bottlenecks under peak load of 10M daily transactions causing data loss incidents. Additionally, V1 schema couldn't support per-locale rows across tenants without secondary indexes.

**Solution:**

**Schema Design:**
- Composite partition key `(key, locale, tenant_id)` — all three participate in partition key
- Routes by item but keeps locales co-resident for read efficiency
- Enables any-tenant/any-locale slice without secondary indexes
- LCS compaction tuned for read-heavy workload
- `@Counted`/`@Timed`/`@ExceptionCounted` metrics annotations for observability

**Bulk Loader:**
- Connection pooling with smart lifecycle management
- Prepared statement caching achieving 90% hit rate
- Multiple write strategies: batch inserts (100-record chunks), async writes
- Retry logic with exponential backoff and dead-letter queuing
- Time-series optimization for temporal data patterns
- `ALLOW FILTERING` query anti-pattern detection and resolution
- Backup table auditing and cleanup automation

**Technologies:** Java, Apache Cassandra 4.x, Connection Pooling, LCS Compaction, Dynatrace observability

**Impact:** Improved write performance by **45%**, eliminated data loss incidents entirely; freed **~2TB** of unused storage in production; achieved **<100ms page load times** for 1M+ daily visitors; new tenant onboarding via config change, not schema change

---

### 7. 🔄 Data Validation Framework
> Scalable, configurable data quality checks for high-volume ETL pipelines

**Problem:** Raw data ingestion at 500K+ records/week required multi-stage validation with zero tolerance for bad data reaching production.

**Solution:**
- Parallel validation processing across configurable worker threads
- 15+ validation types: required fields, regex patterns, SQL injection detection, profanity filtering, search volume thresholds, assortment checks
- Pluggable filter architecture — add new validators without touching core pipeline
- Real-time monitoring and error reporting with BigQuery audit logging

**Technologies:** Python, Threading, Data Quality Patterns, BigQuery

**Impact:** Processes 500K+ records weekly narrowing down to ~50K high-value targets with 99% accuracy

---

### 8. ⚡ Spark Optimization Toolkit
> Performance tuning toolkit for large-scale Spark job optimization

**Problem:** Spark jobs processing 500K+ records hit performance bottlenecks — 6-hour batch processing times and resource inefficiency on cloud clusters.

**Solution:**
- Query plan analysis and cost-based optimization
- Dynamic partition tuning for data-aware parallelism
- Data skew handling via salting and adaptive query execution
- Broadcast join optimization for small-table joins
- Cache management strategies with eviction policies
- GCP Dataproc workflow template integration with auto-scaling

**Technologies:** PySpark, Scala, GCP Dataproc, Performance Tuning

**Impact:** Achieved **60% performance improvement** (6 hours → 2.4 hours); auto-scaling saved **$30K/month** in compute costs

---

### 9. 📊 Monitoring & Alerting System
> Real-time observability across 15+ critical production services

**Problem:** Distributed services across multiple environments (US, CA) needed unified monitoring with instant incident response.

**Solution:**
- Real-time metrics collection and aggregation across distributed services
- Intelligent alert deduplication to reduce noise
- Multi-channel notifications (Slack, Email, PagerDuty)
- SLA tracking with uptime reporting
- Anomaly detection for proactive issue identification
- OpenObserve integration for centralized log querying

**Technologies:** Python, Prometheus patterns, Time-series analysis, OpenObserve

**Impact:** Reduced incident response time by **85%** (30 min → 5 min); contributed to maintaining 99.95% uptime across production fleet

---

### 10. 🚀 CI/CD Automation Suite
> Multi-stage deployment pipeline with zero-downtime release strategies

**Problem:** Manual deployments took 4 hours, caused 15% error rate, and couldn't support the velocity of 7+ production services shipping features weekly.

**Solution:**
- Multi-stage pipeline orchestration (build → test → security scan → deploy → validate)
- Blue-green and canary deployment strategies
- Automated testing gates with unit + integration coverage checks
- Security scanning integration (Apiiro, CodeGate)
- Health checks and validation gates at every stage
- Automatic rollback on failure with state preservation
- Slack notifications for pass/fail with deployment metadata
- **Auto-CRQ generation tooling** for Kitt.yaml deployments — eliminated boilerplate from routine production releases

**Technologies:** Java, Bash, Docker, Kubernetes, GitHub Actions, Looper, Concord

**Impact:** **75% reduction** in deployment time (4h → 1h); **100% elimination** of manual deployment errors

**🏆 Recognition:** Q2 2025 Bravo Award Winner

---

### 11. 🌅 Sunset Automation System
> Intelligent page lifecycle management with automated health-check-driven retirement

**Problem:** Thousands of stale, redirected, or zero-traffic pages degraded site quality and consumed crawl budget — manual identification was impossible at scale.

**Solution:**
- Automated URL health check pipeline: **302/404 detection**, redirect chain validation, assortment checks
- Database schema for sunset candidates: URL, date flagged, `isRedirection`, `isLowAssortment`, `isZeroClicks`
- BigQuery output layer for analytics team pickup
- Sitemap auto-removal of bad URLs with audit logging
- Rate-limited sunset execution to avoid downstream service overload
- Sunset lifecycle: flag → validate → business approval (301 redirect) → sitemap removal → database cleanup

**Technologies:** Java, BigQuery, Cassandra, Health Check Patterns

**Impact:** Auto-removes **2,000+ stale URLs monthly**; recovered **15% of lost organic traffic** through duplicate page consolidation (301 redirects for 5,000+ pages)

---

### 12. 🔒 Zero-Downtime JDK Migration Framework
> Systematic approach to migrating distributed Java services across major JDK versions

**Problem:** 7 production services running on JDK 8 with accumulating security vulnerabilities — needed migration to JDK 17 with zero customer impact.

**Solution:**
- Service-by-service migration strategy with dependency graph analysis
- Automated compatibility scanning for deprecated APIs and removed modules
- Security vulnerability remediation (Apiiro-flagged CVEs) as part of migration
- CI/CD pipeline validation at each migration stage (CodeGate integration)
- Staged rollout: non-prod → canary → prod with automated rollback triggers
- Post-migration validation: log monitoring, error rate comparison, performance benchmarking

**Services Migrated:**
1. RELM Data Publisher
2. Topic Page Generation
3. FTP Page Generation
4. SEO Page Performance
5. Metadata Service
6. IPSUM Service
7. Data Ingestion Service

**Technologies:** Java (JDK 8 → 17), Apiiro, CodeGate, CI/CD

**Impact:** **7 services migrated with zero downtime**; all security vulnerabilities remediated; no production incidents during or after migration

---

## Performance Metrics Summary

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Translation Throughput | Per-record REST | Batched Kafka (1000/batch) | **300× improvement** |
| Deployment Time | 4 hours | 1 hour | **75% reduction** |
| Incident Response | 30 minutes | 5 minutes | **85% reduction** |
| Batch Processing | 6 hours | 2.4 hours | **60% reduction** |
| Write Performance | Baseline | Optimized | **45% improvement** |
| GraphQL Read Amplification | N reads per request | 1 batched read | **N-to-1 coalescing** |
| System Uptime | 99% | 99.95% | **10× fewer incidents** |
| Manual Errors | 15% | 0% | **100% elimination** |
| Page Load Time | Variable | <100ms | **Consistent sub-100ms** |
| Storage Waste | ~2TB unused | 0 | **2TB freed** |
| Monthly Compute Cost | Baseline | Optimized | **$30K/month saved** |
| Annual Infra Cost | Baseline | Optimized | **$550K+ saved** |

---

## Architecture Patterns

### Scalability
- Horizontal scaling with distributed processing (50+ Spark executors)
- Producer-consumer Kafka pipelines decoupling write throughput from downstream latency
- BatchLoader-coalesced reads on GraphQL serving layer (N-to-1 per request)
- Sequential file processing for 200M+ record Cassandra writes (OOM avoidance)
- Connection pooling and prepared statement caching for write paths
- Strategy-pattern source abstraction enabling new data sources without code change
- Auto-scaling cloud clusters based on workload

### Reliability
- Circuit breakers for failure isolation across services
- Retry mechanisms with exponential backoff
- Dead-letter queue patterns for unrecoverable failures
- Health checks and validation gates at every deployment stage
- Checkpoint management for pipeline recovery
- Idempotent upsert semantics with composite tracking flags (`has_processed`/`is_translated`/`retry_count`)
- Zero-downtime migration strategies (canary, blue-green, rollback)

### Performance
- Prepared statement caching (90% hit rate)
- Query plan analysis and cost-based optimization
- Multi-tenant composite partition keys avoiding secondary indexes
- LCS compaction tuned for read-heavy Cassandra workloads
- Cross-project BigQuery DML via staging-table NDJSON + same-project MERGE
- JVM heap tuning under in-memory payload pressure
- Rate limiting to protect downstream services
- Proactive storage auditing and cleanup

---

## Key Learnings

> "Building systems that handle millions of transactions requires thinking differently about every line of code. Connection pooling alone reduced overhead by 80%."

> "Every manual step is a potential failure point. CI/CD automation eliminated manual errors entirely while reducing deployment time by 75%."

> "You can't improve what you don't measure. Comprehensive monitoring reduced incident response time from 30 minutes to 5 minutes."

> "Sometimes a 10-line configuration change can improve performance by 60%. Understanding your tools deeply is crucial."

> "Zero-downtime migrations aren't about being careful — they're about having automated rollback, staged rollouts, and knowing your dependency graph cold."

> "Replacing a synchronous REST call with a Kafka producer-consumer pair is one of the highest-leverage architectural changes in a backend system. We saw 300× throughput because the producer no longer blocked on downstream latency."

> "Designing the right composite partition key is worth more than any amount of read optimization. We picked `(key, locale, tenant_id)` because it lets a single table serve any tenant or any locale slice without secondary indexes."

---

## Technology Stack

| Category | Technologies | Proficiency |
|----------|-------------|-------------|
| **Languages** | Java 17, Scala 2.12, Python, SQL, Bash | Expert |
| **Backend Frameworks** | Spring Boot 3.2, Netflix DGS (GraphQL), Kafka client libraries | Expert |
| **AI/ML** | OpenAI API, LangChain, BERT, Hugging Face, Stanford CoreNLP, OpenNLP | Expert |
| **Big Data** | Apache Spark, Hive, Dataproc, Airflow | Expert |
| **Streaming** | Apache Kafka (TAAS, mTLS, SSL/JKS), Producer-Consumer Patterns | Expert |
| **Databases** | Apache Cassandra (composite-key schema, LCS compaction), BigQuery (cross-project DML), PostgreSQL, Azure Cosmos DB | Advanced |
| **Cloud** | GCP (Dataproc, BigQuery, GCS, Secret Manager), Azure (Cosmos DB, Blob, DF Cassandra) | Advanced |
| **Infrastructure** | Docker, Kubernetes/WCNP, Istio, CI/CD, GitHub Actions, Looper, Concord | Expert |
| **APIs** | REST, GraphQL (DGS DataFetcher / BatchLoader / Federated mGQL), Service Integration | Expert |
| **Monitoring** | Prometheus, Grafana, OpenObserve, Dynatrace, Apiiro | Advanced |

---

## How to Use This Portfolio

Each project directory includes:
1. **Complete source code** with inline documentation
2. **Example usage** and runnable demonstrations
3. **Performance metrics** and optimization notes
4. **Production considerations** and best practices

```bash
git clone https://github.com/NightCodeOwl/NightCodeOwl.git
cd <project-directory>
# Follow the README in each project folder
```

---

## What This Demonstrates

✅ **Production-Grade Engineering** — Built with reliability, error handling, and monitoring from day one

✅ **Scale** — Designed to handle 200M+ records, 10M+ daily transactions, and 1M+ daily users

✅ **Performance** — Every system optimized for speed and resource efficiency (300× throughput on Kafka pipeline, N-to-1 read coalescing on GraphQL, 45% write throughput on Cassandra)

✅ **AI Integration** — LLM pipelines (Claude, GPT-4o, Gemini, BERT, LangChain) shipping to production at enterprise scale

✅ **Distributed Systems Depth** — Composite-key schema design, BatchLoader coalescing, Strategy-pattern source abstraction, cross-project BigQuery DML, mTLS Kafka producer-consumer

✅ **Security** — Vulnerability remediation, zero-downtime migrations, and compliance automation

✅ **Business Impact** — $550K+ in annual savings, $2M+ in business value generated

---

## Contact

- 💼 [LinkedIn](https://www.linkedin.com/in/deepanshu-maheshwari-748823172/)
- 📧 [deepanshumaheshwarri99@gmail.com](mailto:deepanshumaheshwarri99@gmail.com)
- 🐙 [GitHub](https://github.com/NightCodeOwl)

---

> *All code in this portfolio is original work created for demonstration purposes. It represents simplified, generic versions of solutions implemented in production environments. No proprietary code, data, or business logic from any employer is included.*

*"The best code is not just functional — it's scalable, maintainable, and delivers real business value."*
