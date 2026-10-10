# Deepanshu Maheshwari's GitHub Profile

🌐 **Portfolio website:** [https://nightcodeowl.github.io/NightCodeOwl/](https://nightcodeowl.github.io/NightCodeOwl/)

**Role:** Software Engineer III (SDE III) at Walmart Global Tech, building backend and data systems for Walmart's international SEO platform

**What I build:** Event-driven Spring Boot + Kafka services, GraphQL serving layers with DGS BatchLoader, multi-tenant Cassandra schemas, and Spark/Scala batch pipelines on GCP Dataproc.

**Key Accomplishments:**
- Replaced a rate-limited synchronous REST integration with an **async Kafka translation service** (Java 17 / Spring Boot): 1.28M catalog items sent for English → French localization
- Migrated a production Kafka service from certificate-based mTLS to **SPIFFE Workload Identity with zero downtime**, in reversible phases
- Sole owner of the epic **unifying the CA/MX/US alt-text pipelines** into one config-driven Spark job, rolled out via dual-write, shadow validation and cutover
- Built **GraphQL DGS BatchLoader** serving over Cassandra that eliminates N+1 reads; 3.2M AI-generated alt-text records live for Canada
- Built a **200M+ record Cassandra publisher** on Dataproc (50 executors); traced its write bottleneck to client-side overhead, not cluster capacity
- Diagnosed a topic-page collapse in a 4.1M-record pipeline with a 7-stage validation funnel, **restoring 199,917 publishable pages**
- Led the **JDK 8 → 17 migration of 7 services with zero downtime** (8 merged PRs), closing injection and unsafe-deserialization paths
- Earlier: built the DSP/EA advertising **Airflow** platform (62 PRs) and the Spark audience-segmentation service (32 PRs)
- Recipient of the **Q2 2025 Bravo Award** for automation innovation

**Technical Expertise:**
- **Languages:** Java 17, Scala, Python, SQL
- **Backend & Distributed Systems:** Spring Boot, Apache Kafka, GraphQL (DGS DataFetcher / BatchLoader / Federation), REST, Microservices, Event-Driven Architecture
- **Data Systems:** Apache Cassandra (composite-key schema design, LCS compaction), BigQuery, Hive, PostgreSQL, Azure Cosmos DB, Apache Spark, Apache Airflow
- **Cloud & Platform:** GCP (Dataproc, GCS, BigQuery, IAM, VPC Service Controls, Secret Manager), Kubernetes, Istio, SPIFFE Workload Identity, Terraform, Looper, Concord
- **Observability:** Prometheus, Dynatrace

Production code at Walmart Global Tech stays in private repositories. See [PORTFOLIO_README.md](./PORTFOLIO_README.md) for design-level write-ups of these systems.

**🌐 Portfolio:** [https://nightcodeowl.github.io/NightCodeOwl/](https://nightcodeowl.github.io/NightCodeOwl/)

**Contact:** [LinkedIn](https://www.linkedin.com/in/deepanshuu-maheshwari/) · [deepanshuumaheshwari@gmail.com](mailto:deepanshuumaheshwari@gmail.com)
