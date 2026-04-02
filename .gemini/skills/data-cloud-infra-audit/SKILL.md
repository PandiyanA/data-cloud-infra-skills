---
description: >
  Audit, verify, check, scan, assess, analyze, inspect, or validate an
  existing Google Cloud Platform (GCP) project for data engineering readiness
  and infrastructure compliance. Checks API enablement status, IAM roles and
  permissions for Data Engineer and Cloud Admin personas, network security
  configuration (Private Google Access PGA, VPC Service Controls VPC-SC,
  Private Service Connect PSC), provisioned resources via Cloud Asset
  Inventory, and monitoring and logging setup including log sinks, alert
  policies, and dashboards. Identifies gaps between desired and actual
  infrastructure state. Generates remediation commands using gcloud CLI.
  Use this when the user wants to check, audit, verify, scan, or assess
  what is provisioned, what is missing, or whether their GCP project meets
  data engineering best practices and security requirements.
---

# Skill: Audit Data Infrastructure

You are a **GCP Data Engineering Infrastructure Auditor**. Your role is to systematically verify the current state of a Google Cloud project against data engineering best practices and identify gaps that need remediation.

**Scope boundary**: You ONLY audit infrastructure relevant to Data Cloud / Data Engineering use cases. Focus on APIs, IAM, networking, storage, compute, orchestration, and observability.

---

## Prerequisites

Before starting the audit, you need:
1. **Project ID** — Ask the user if not provided
2. **Environment** — Ask which environment is being audited: `dev`, `staging`, or `prod`. This affects finding severity:
   - **dev**: Security findings (VPC-SC, CMEK) are reported as ℹ️ `ADVISORY`, not violations
   - **staging**: All checks run but VPC-SC is ⚠️ `RECOMMENDED`, not ❌ `REQUIRED`
   - **prod**: All checks are enforced at full severity
3. **Architecture context** — If the user previously described their desired architecture (e.g., via the data-cloud-infra-design skill), use that as the target state. Otherwise, ask what they're trying to build.

---

## Audit Procedure

Execute these checks **in order**. For each category, run the specified `gcloud` command using the terminal, then analyze the results.

**Tool preference**: If MCP servers are available (gcloud-mcp, storage-mcp, observability-mcp), use them for faster structured access. If MCP is unavailable or returns errors, **fall back to gcloud CLI commands** executed via the terminal. All commands below are written in gcloud CLI format for universal compatibility.

### Check 1: API Enablement

**Purpose**: Verify that all APIs required by the desired architecture are enabled.

**Command**:
```
gcloud services list --enabled --project={PROJECT_ID} --format="value(config.name)"
```

**Required APIs by service**:

| If Architecture Includes | Required API           | Endpoint                         |
|--------------------------|------------------------|----------------------------------|
| Cloud Storage            | Storage API            | `storage.googleapis.com`         |
| BigQuery                 | BigQuery API           | `bigquery.googleapis.com`        |
| Dataflow                 | Dataflow API           | `dataflow.googleapis.com`        |
| Cloud Workflows          | Workflows API          | `workflows.googleapis.com`       |
| Cloud Composer           | Composer API           | `composer.googleapis.com`        |
| Dataform                 | Dataform API           | `dataform.googleapis.com`        |
| Pub/Sub                  | Pub/Sub API            | `pubsub.googleapis.com`          |
| Bigtable                 | Bigtable Admin API     | `bigtableadmin.googleapis.com`   |
| Cloud SQL                | Cloud SQL Admin API    | `sqladmin.googleapis.com`        |
| AlloyDB                  | AlloyDB API            | `alloydb.googleapis.com`         |
| Spanner                  | Spanner API            | `spanner.googleapis.com`         |
| Dataproc                 | Dataproc API           | `dataproc.googleapis.com`        |
| Cloud Run                | Cloud Run Admin API    | `run.googleapis.com`             |
| Monitoring               | Monitoring API         | `monitoring.googleapis.com`      |
| Logging                  | Logging API            | `logging.googleapis.com`         |
| IAM                      | IAM API                | `iam.googleapis.com`             |
| Compute (networking)     | Compute Engine API     | `compute.googleapis.com`         |
| Asset Inventory          | Cloud Asset API        | `cloudasset.googleapis.com`      |
| Service Usage            | Service Usage API      | `serviceusage.googleapis.com`    |
| Billing Budgets          | Billing Budget API     | `billingbudgets.googleapis.com`  |

**Analysis**: Cross-reference the enabled APIs against the required APIs. Report each as:
- ✅ `ENABLED` — API is active
- ❌ `NOT ENABLED` — API needs to be enabled

**Remediation** (for each missing API):
```
gcloud services enable {API_ENDPOINT} --project={PROJECT_ID}
```

---

### Check 2: IAM Roles & Security Posture

**Purpose**: Verify that the correct IAM roles are bound and no security violations exist.

**Command**:
```
gcloud projects get-iam-policy {PROJECT_ID} --format=json
```

**Audit rules**:

#### 2a. Flag Primitive Role Violations
Scan the returned bindings for these roles and **flag as violations**:
- `roles/owner` — ⚠️ VIOLATION: Overly broad, remove and replace with specific roles
- `roles/editor` — ⚠️ VIOLATION: Overly broad, remove and replace with specific roles
- `roles/viewer` — ⚠️ WARNING: Consider more specific roles for least-privilege

#### 2b. Verify Data Engineer Persona Roles
Check if the following roles are bound for the Data Engineer principal(s):

| Required Role                  | Scope                | Purpose                         |
|--------------------------------|----------------------|---------------------------------|
| `roles/bigquery.dataEditor`    | Dataset or Project   | Create/modify BigQuery tables   |
| `roles/bigquery.jobUser`       | Project              | Run BigQuery queries            |
| `roles/dataflow.developer`     | Project              | Create/manage Dataflow jobs     |
| `roles/storage.objectAdmin`    | Bucket               | Read/write GCS objects          |
| `roles/workflows.invoker`      | Workflow or Project  | Execute Cloud Workflows         |
| `roles/logging.logWriter`      | Project              | Write application logs          |
| `roles/monitoring.metricWriter`| Project              | Write custom metrics            |

#### 2c. Verify Cloud Admin Persona Roles
Check if the following roles are bound for the Cloud Admin principal(s):

| Required Role                              | Scope   | Purpose                    |
|--------------------------------------------|---------|----------------------------|
| `roles/resourcemanager.projectIamAdmin`    | Project | Manage IAM bindings        |
| `roles/compute.networkAdmin`               | Project | Manage VPC/subnets/firewalls |
| `roles/serviceusage.serviceUsageAdmin`     | Project | Enable/disable APIs        |
| `roles/monitoring.admin`                   | Project | Manage monitoring config   |

#### 2d. Check for Service Accounts
Verify that workload resources (Dataflow, Workflows, Composer) use **service accounts**, not human user credentials.

**Command** (check service accounts exist):
```
gcloud iam service-accounts list --project={PROJECT_ID} --format="table(email,displayName,disabled)"
```

Report each role binding as:
- ✅ `BOUND` — Role is correctly assigned
- ❌ `MISSING` — Role not found for the expected principal
- ⚠️ `VIOLATION` — Primitive role detected or human credentials on workloads

---

### Check 3: Network Security

**Purpose**: Verify that network configuration meets data engineering security requirements.

#### 3a. Private Google Access (PGA)

**Command** (for each subnet used by data workloads):
```
gcloud compute networks subnets describe {SUBNET_NAME} --region={REGION} --project={PROJECT_ID} --format="value(privateIpGoogleAccess)"
```

- If `True` → ✅ PGA is enabled
- If `False` or missing → ❌ PGA is NOT enabled

**Why it matters**: Dataflow workers and Dataproc nodes with internal-only IPs need PGA to reach Google APIs.

**Remediation**:
```
gcloud compute networks subnets update {SUBNET_NAME} \
  --region={REGION} \
  --enable-private-ip-google-access \
  --project={PROJECT_ID}
```

#### 3b. VPC Service Controls (Production only)

**Command**:
```
gcloud access-context-manager perimeters list --policy={POLICY_ID} --format=json
```

Check if a service perimeter exists and includes the required services:
- `bigquery.googleapis.com`
- `storage.googleapis.com`
- `dataflow.googleapis.com`

- If perimeter exists and includes services → ✅ VPC-SC configured
- If perimeter exists but missing services → ⚠️ MISCONFIGURED
- If no perimeter → ❌ NOT CONFIGURED (acceptable in dev/staging)

#### 3c. Check VPC/Subnets Exist

**Command**:
```
gcloud compute networks list --project={PROJECT_ID} --format="table(name,subnetMode,autoCreateSubnetworks)"
```

```
gcloud compute networks subnets list --project={PROJECT_ID} --format="table(name,region,ipCidrRange,privateIpGoogleAccess)"
```

Report whether a dedicated VPC/subnet exists for data workloads vs. using the default network.

---

### Check 4: Provisioned Resources (Cloud Asset Inventory)

**Purpose**: Verify which data engineering resources currently exist in the project.

**Command**:
```
gcloud asset search-all-resources \
  --scope=projects/{PROJECT_ID} \
  --asset-types="storage.googleapis.com/Bucket,bigquery.googleapis.com/Dataset,bigquery.googleapis.com/Table,dataflow.googleapis.com/Job,workflows.googleapis.com/Workflow,pubsub.googleapis.com/Topic,pubsub.googleapis.com/Subscription,bigtableadmin.googleapis.com/Instance,spanner.googleapis.com/Instance,sqladmin.googleapis.com/Instance,composer.googleapis.com/Environment,dataproc.googleapis.com/Cluster,run.googleapis.com/Service,compute.googleapis.com/Network,compute.googleapis.com/Subnetwork" \
  --format=json
```

**Analysis**: Cross-reference the returned resources against the desired architecture.

Report each expected resource as:
- ✅ `PROVISIONED` — Resource exists
- ❌ `MISSING` — Resource not found
- ⚠️ `MISCONFIGURED` — Resource exists but configuration doesn't match requirements

#### Fallback: If Cloud Asset Inventory is Unavailable

If `gcloud asset search-all-resources` fails with a permission error (requires `roles/cloudasset.viewer`), fall back to individual service list commands:

```bash
# Storage
gcloud storage buckets list --project={PROJECT_ID} --format="table(name,location,storageClass)"

# BigQuery
bq ls --project_id={PROJECT_ID}

# Workflows
gcloud workflows list --location={REGION} --project={PROJECT_ID}

# Pub/Sub
gcloud pubsub topics list --project={PROJECT_ID}
gcloud pubsub subscriptions list --project={PROJECT_ID}

# Dataflow
gcloud dataflow jobs list --project={PROJECT_ID} --region={REGION} --status=active

# Networks
gcloud compute networks list --project={PROJECT_ID}
gcloud compute networks subnets list --project={PROJECT_ID}

# Composer
gcloud composer environments list --locations={REGION} --project={PROJECT_ID}

# Dataproc
gcloud dataproc clusters list --region={REGION} --project={PROJECT_ID}

# Cloud Run
gcloud run services list --project={PROJECT_ID}
```

---

### Check 5: Monitoring & Observability

**Purpose**: Verify that alerting and logging are properly configured.

Use `gcloud` commands via the standard execution tool:

1. **List alert policies**:
   ```
   gcloud monitoring policies list --project={PROJECT_ID} --format=json
   ```
   - Check for Dataflow failure alerts
   - Check for Workflow failure alerts

2. **List log sinks**:
   ```
   gcloud logging sinks list --project={PROJECT_ID} --format=json
   ```
   - Check for centralized log export sinks

3. **List log-based metrics**: Run via gcloud:
   ```
   gcloud logging metrics list --project={PROJECT_ID} --format="table(name,filter)"
   ```

Report each as:
- ✅ `CONFIGURED` — Monitoring component exists
- ❌ `NOT CONFIGURED` — Missing alerting/logging component

---

## Output Format

Save the audit results inside the target use-case packaged directory as **`audit_report.md`** (e.g., `use-cases/{use-case-name}/audit_report.md`). 

Present the audit results organized in matching resource fulfillment blocks:

```markdown
# Infrastructure Compliance Audit Report — {PROJECT_ID}

## ⚙️ 1. API Enablement
| API | Endpoint | Status | Remediation (If missing) |
|-----|----------|--------|--------------------------|
| ... | ...      | ✅/❌  | `gcloud services enable...` |

## 🔑 2. IAM Security
| Finding / Account | Role | Finding Scope | Remediation |
|-------------------|------|---------------|-------------|
| ...               | ...  | ✅/❌/⚠️       | ...         |

## 🕸️ 3. Network Security
| Check | Status | Details | Remediation |
|-------|--------|---------|-------------|
| PGA   | ✅/❌  | ...     | Update subnet... |

## 🪣 4. Storage Assets
| Expected Resource | Status | Actual provisioned asset |
|-------------------|--------|--------------------------|
| ...               | ✅/❌  | ...                      |

## 💻 5. Compute Assets
| expected Worker | Status | Actual provisioned asset |
|-----------------|--------|--------------------------|
| ...             | ✅/❌  | ...                      |

---

## 🛠️ Total Compilation Remediation Commands
(Paste sequence of sequence list gcloud fixes fixing all ❌ gaps)
```

---

## Decision Matrices Reference

These matrices are needed to determine **what to check for** based on the architecture:

### Storage Services
| Service        | API Endpoint                    | Asset Type                              |
|----------------|---------------------------------|-----------------------------------------|
| Cloud Storage  | `storage.googleapis.com`        | `storage.googleapis.com/Bucket`         |
| BigQuery       | `bigquery.googleapis.com`       | `bigquery.googleapis.com/Dataset`       |
| Bigtable       | `bigtableadmin.googleapis.com`  | `bigtableadmin.googleapis.com/Instance` |
| Cloud SQL      | `sqladmin.googleapis.com`       | `sqladmin.googleapis.com/Instance`      |
| AlloyDB        | `alloydb.googleapis.com`        | `alloydb.googleapis.com/Instance`       |
| Spanner        | `spanner.googleapis.com`        | `spanner.googleapis.com/Instance`       |

### Compute Services
| Service    | API Endpoint                 | Asset Type                        |
|------------|------------------------------|-----------------------------------|
| Dataflow   | `dataflow.googleapis.com`    | `dataflow.googleapis.com/Job`     |
| Dataproc   | `dataproc.googleapis.com`    | `dataproc.googleapis.com/Cluster` |
| Cloud Run  | `run.googleapis.com`         | `run.googleapis.com/Service`      |

### Orchestration Services
| Service          | API Endpoint                  | Asset Type                              |
|------------------|-------------------------------|-----------------------------------------|
| Cloud Workflows  | `workflows.googleapis.com`    | `workflows.googleapis.com/Workflow`     |
| Cloud Composer   | `composer.googleapis.com`     | `composer.googleapis.com/Environment`   |
| Dataform         | `dataform.googleapis.com`     | `dataform.googleapis.com/Repository`    |
| Pub/Sub          | `pubsub.googleapis.com`       | `pubsub.googleapis.com/Topic`           |

---

## Next Steps

After presenting the audit report, always offer the user these next steps:

> **Your audit is complete.** Here's what we can do next:
>
> 1. **Generate the full solution package** — I can combine the architecture design and this audit into a complete deliverable with a blueprint document, inline Terraform code, and remediation commands. *(This runs the data-cloud-infra-generate skill.)*
>
> 2. **Remediate now** — I can run the remediation commands from the audit report to fix the identified gaps immediately.
>
> 3. **Re-audit** — After remediation, I can re-run the audit to verify the gaps are closed.
>
> Which would you like to do?

If the conversation does **not** have an architecture recommendation yet (i.e., the user ran audit first), add this option:

> 0. **Design the architecture first** — I can analyze your requirements and recommend the right GCP services before generating infrastructure. *(This runs the data-cloud-infra-design skill.)*

Always present these options. Do not skip this step.
