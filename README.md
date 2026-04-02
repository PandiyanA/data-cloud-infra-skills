# GCP Data Engineering — AI-Powered Resource Provisioning

> **Version 1.0.0** · Last updated April 2026

An AI-assisted infrastructure provisioning toolkit for Google Cloud data engineering. Instead of manually writing Terraform, you **describe what you need in natural language** and the AI designs, audits, and generates production-ready inline Terraform for you.

---

## Quick Start — Try These Prompts

Copy-paste any of these into your AI assistant to see the skills in action:

### 🏗️ Design a pipeline
```
I need a batch pipeline that ingests CSV files from GCS every hour,
transforms them with Dataflow, and loads them into BigQuery for analytics dashboards.
```

### 🔍 Audit an existing project
```
Audit my GCP project my-project-id for data engineering readiness.
The environment is dev.
```

### 📦 Generate the full solution package
```
Generate the complete solution package with Terraform code and blueprint
for the architecture we just designed.
```

---

## How It Works

This project combines three AI skills with a live GCP connection:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        AI Skills (.gemini/skills/)                      │
│                                                                         │
│   ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐   │
│   │  data-cloud-     │   │  data-cloud-     │   │  data-cloud-     │   │
│   │  infra-design    │──▶│  infra-audit     │──▶│  infra-generate  │   │
│   │                  │   │                  │   │                  │   │
│   │  "I need a batch │   │  Checks APIs,    │   │  Produces a      │   │
│   │   pipeline for   │   │  IAM, network,   │   │  complete        │   │
│   │   sensor data"   │   │  resources in    │   │  solution pack   │   │
│   │                  │   │  your GCP project│   │  with Terraform  │   │
│   └──────────────────┘   └──────────────────┘   └──────────────────┘   │
│            │                       │                       │            │
│            ▼                       ▼                       ▼            │
│   Recommends services     Finds gaps & issues    Generates inline      │
│   with cost warnings      + gcloud remediation   Terraform + docs      │
└─────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        MCP Extension (.gemini/extensions/)              │
│                                                                         │
│   gcloud-mcp  ·  storage-mcp  ·  observability-mcp                     │
│   Live access to your GCP project for auditing                          │
│   Falls back to gcloud CLI if MCP is unavailable                        │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## The 3-Step Workflow

### Step 1: Design — *"What should I build?"*

**Skill**: `data-cloud-infra-design`

Describe your data engineering need in plain English. The AI walks you through a structured discovery flow — asking 3 sequential questions with clickable options — and applies decision matrices to recommend the right GCP services.

```
You:  "I need to ingest IoT sensor data in real-time, detect anomalies,
       and make it available for dashboards."

AI:   Recommends Pub/Sub → Dataflow → Bigtable (hot) + BigQuery (analytics)
      Includes cost warnings for high-floor services
      Specifies IAM roles, naming conventions, labels
```

**What the AI evaluates:**

| Decision Area | How It Decides |
|---------------|----------------|
| Storage       | Data profile × Latency × Volume → GCS, BigQuery, Bigtable, Cloud SQL, AlloyDB, or Spanner |
| Compute       | Processing paradigm → Dataflow (streaming), Dataproc (Spark), or Cloud Run (containers) |
| Orchestration | Complexity → Workflows (simple), Composer (complex DAGs), Dataform (SQL-only), Pub/Sub (async) |
| Cost          | Warns about high-floor services (Spanner ~$657/mo, Composer ~$350/mo, Bigtable ~$467/mo) |

---

### Step 2: Audit — *"What's already there?"*

**Skill**: `data-cloud-infra-audit`

The AI connects to your live GCP project (via MCP or gcloud CLI) and runs 5 systematic checks with environment-aware severity:

| Check                      | What It Verifies                                                       |
|----------------------------|------------------------------------------------------------------------|
| **1. API Enablement**      | Are all required APIs enabled?                                         |
| **2. IAM Security**        | Correct roles per persona? Primitive role violations? Service accounts? |
| **3. Network Security**    | Private Google Access? VPC Service Controls? Dedicated VPC?            |
| **4. Provisioned Resources** | Which resources exist vs. what the architecture needs?               |
| **5. Monitoring**          | Alert policies? Log sinks? Dashboards?                                 |

Each finding is tagged: ✅ `OK` · ❌ `MISSING` · ⚠️ `VIOLATION`

Severity adjusts by environment: dev findings are ℹ️ advisory, prod findings are ❌ enforced.

---

### Step 3: Blueprint — *"Give me the deliverable"*

**Skill**: `data-cloud-infra-generate`

Combines the design + audit into a complete solution package:

```
use-cases/{use-case-name}/
├── blueprint.md              # Architecture doc with Mermaid diagrams
├── providers.tf              # Provider config + GCS remote backend
├── variables.tf              # Variable declarations
├── main.tf                   # Inline resources (no modules)
├── observability.tf          # Alert policies + billing budgets
├── outputs.tf                # Resource identifiers
├── terraform.tfvars.example  # Example variable values
├── verify_environment.sh     # Pre-flight readiness check
└── README.md                 # Deployment guide + troubleshooting
```

All Terraform is generated as **inline `resource {}` blocks** — no module dependencies, no external references. Every resource includes security hardening attributes by default.

---

## Supported GCP Services

The AI can recommend and generate Terraform for any of these services:

### Storage
| Service                | Use Case                              | Terraform Resource              |
|------------------------|---------------------------------------|---------------------------------|
| Cloud Storage (GCS)    | Landing zone, raw data, archives      | `google_storage_bucket`         |
| BigQuery               | Data warehouse, analytics, BI         | `google_bigquery_dataset`       |
| Cloud Bigtable         | Low-latency NoSQL, time-series        | `google_bigtable_instance`      |
| Cloud SQL              | Managed relational (PostgreSQL/MySQL) | `google_sql_database_instance`  |
| AlloyDB                | Enterprise PostgreSQL, HTAP           | `google_alloydb_cluster`        |
| Cloud Spanner          | Global transactional, five-9s         | `google_spanner_instance`       |

### Compute
| Service          | Use Case                            | Terraform Resource              |
|------------------|-------------------------------------|---------------------------------|
| Cloud Dataflow   | Streaming & batch ETL (Apache Beam) | `google_dataflow_job`           |
| Cloud Dataproc   | Spark / Hadoop workloads            | `google_dataproc_cluster`       |
| Cloud Run        | Serverless containers, APIs         | `google_cloud_run_v2_service`   |

### Orchestration
| Service          | Use Case                       | Terraform Resource              |
|------------------|--------------------------------|---------------------------------|
| Cloud Workflows  | HTTP-based service chaining    | `google_workflows_workflow`     |
| Cloud Composer   | Complex Airflow DAGs           | `google_composer_environment`   |
| Dataform         | SQL-centric ELT in BigQuery    | `google_dataform_repository`    |
| Pub/Sub          | Async event-driven messaging   | `google_pubsub_topic`           |

### Infrastructure
| Service          | Use Case                       | Terraform Resource                |
|------------------|--------------------------------|-----------------------------------|
| VPC / Subnets    | Network isolation              | `google_compute_network`          |
| Cloud NAT        | Outbound for private workers   | `google_compute_router_nat`       |
| IAM              | Service accounts, least-privilege | `google_service_account`       |
| Monitoring       | Alerts, dashboards, budgets    | `google_monitoring_alert_policy`  |

---

## Project Structure

```
gcp-resource-provisioning-opus/
│
├── .gemini/
│   ├── skills/
│   │   ├── data-cloud-infra-design/    # Step 1: Design architecture
│   │   │   └── SKILL.md
│   │   ├── data-cloud-infra-audit/     # Step 2: Audit GCP project
│   │   │   └── SKILL.md
│   │   └── data-cloud-infra-generate/  # Step 3: Generate solution package
│   │       └── SKILL.md
│   └── extensions/
│       └── data-provisioner/           # MCP servers for live GCP access
│           └── gemini-extension.json
│
├── use-cases/                          # Generated solution packages (gitignored)
│   └── {use-case-name}/               # Created by the data-cloud-infra-generate skill
│
├── .bin/terraform                      # Local Terraform binary (optional)
├── .gitignore
└── README.md
```

---

## Prerequisites

- A GCP project with billing enabled
- [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) (`gcloud`) — authenticated via `gcloud auth application-default login`
- [Terraform](https://developer.hashicorp.com/terraform/install) >= 1.5 (or use the bundled `.bin/terraform`)
- An AI assistant with Gemini skills support (e.g., Gemini Code Assist, Jules)

---

## Getting Started

### 1. Clone and open the project

```bash
git clone <repo-url>
cd gcp-resource-provisioning-opus
```

Open the project in your AI-enabled IDE.

### 2. Describe what you need

Tell the AI what kind of data pipeline you're building:

> *"I need a batch pipeline that ingests CSV files from GCS, transforms them with Dataflow, and loads them into BigQuery for analytics."*

### 3. Let the AI design it

The `data-cloud-infra-design` skill walks you through 3 questions with clickable options, applies decision matrices, and recommends services with cost awareness.

### 4. Audit your existing project (optional)

If you already have a GCP project, the `data-cloud-infra-audit` skill checks what's already provisioned and what's missing — with environment-aware severity.

### 5. Generate the solution package

The `data-cloud-infra-generate` skill produces a complete directory with:
- Architecture documentation (Mermaid diagrams, trade-off analysis)
- Inline Terraform with security hardening
- Pre-flight verification script
- Deployment guide

### 6. Apply with Terraform

```bash
cd use-cases/{use-case-name}/
./verify_environment.sh YOUR_PROJECT_ID   # Pre-flight check
terraform init
terraform plan -var="project_id=YOUR_PROJECT_ID"
terraform apply -var="project_id=YOUR_PROJECT_ID"
```

---

## Testing Guide

If you're testing these skills, here's what to verify for each:

### `data-cloud-infra-design`

| What to Test                           | Expected Behavior                                                                  |
|----------------------------------------|------------------------------------------------------------------------------------|
| Ask for a batch pipeline               | AI asks 3 sequential questions with clickable options, then recommends services    |
| Ask for a streaming pipeline           | AI triggers Fork C (Streaming Branch) with Bigtable/Dataflow options               |
| Ask for an OLTP database               | AI triggers Fork B (Database Branch) with Cloud SQL/AlloyDB/Spanner options        |
| Ask for SQL-only transforms            | AI recommends Dataform, skips compute                                              |
| Mention a high-cost service            | AI includes a ⚠️ cost warning with monthly estimate                                |
| Ask about web hosting (out of scope)   | AI politely declines and explains it only handles data engineering                 |

### `data-cloud-infra-audit`

| What to Test                          | Expected Behavior                                                     |
|---------------------------------------|-----------------------------------------------------------------------|
| Provide a project ID + environment    | AI runs 5 checks in order using gcloud CLI or MCP                     |
| Set environment to `dev`              | Security findings (VPC-SC, CMEK) reported as ℹ️ advisory              |
| Set environment to `prod`             | All findings enforced at full severity                                |
| Project missing required APIs         | AI lists each missing API with `gcloud services enable` remediation   |
| MCP unavailable                       | AI falls back to individual gcloud commands                           |

### `data-cloud-infra-generate`

| What to Test                            | Expected Behavior                                                                    |
|-----------------------------------------|--------------------------------------------------------------------------------------|
| Run after design + audit                | AI generates 9-file solution package in `use-cases/`                                 |
| Check `main.tf`                         | All resources are inline `resource {}` blocks — no `module {}` blocks                |
| Check security attributes               | GCS buckets have `uniform_bucket_level_access = true`, Dataflow has private IPs      |
| Check `providers.tf`                    | Includes `backend "gcs"` block                                                       |
| Check `observability.tf`                | Includes alert policies and billing budget                                           |
| Run `terraform fmt` on output           | Should pass with no changes                                                          |
| Run `terraform validate` on output      | Should pass                                                                          |

---

## Reporting Issues

When reporting issues, please include:

1. **Which skill** you were testing
2. **The prompt** you used (copy-paste)
3. **What happened** vs. what you expected
4. **Screenshots** of the AI's response (if relevant)
5. **Your environment**: AI assistant name/version, OS, gcloud version

---

## Design Principles

| Principle              | How It's Applied                                                                         |
|------------------------|------------------------------------------------------------------------------------------|
| **Intent-Driven**      | Describe what you need; the AI decides how to build it                                   |
| **Inline Resources**   | All Terraform is generated as standalone `resource {}` blocks — no module dependencies   |
| **Secure by Default**  | No primitive roles, scoped IAM, Private Google Access, uniform bucket access, no public IPs |
| **Cost-Aware**         | Warns about high-floor services before recommending them                                 |
| **Environment-Aware**  | Audit severity adjusts by environment (dev / staging / prod)                             |
| **Observable**         | Every architecture includes monitoring alerts and billing budgets                        |
| **Auditable**          | Live project auditing with MCP or gcloud CLI fallback                                    |

---

## Naming Conventions

All resources follow: `{prefix}-{environment}-{service}-{purpose}`

Examples: `pipeline-dev-gcs-raw-data`, `pipeline-prod-bq-analytics`

---

## License

Internal use. See your organization's policies.
