---
description: >
  Generate, create, produce, write, synthesize, compile, or format a
  structured solution package (blueprint, Terraform configuration, and summary)
  for a Google Cloud Platform (GCP) data engineering project. Combines architecture
  recommendations and infrastructure audit results into a comprehensive
  directory for the given use-case. Generates a `blueprint.md`, a standard multi-file layout (`main.tf`, `providers.tf`, `variables.tf`, `outputs.tf`, `terraform.tfvars.example`), and a `README.md` summary inside `use-cases/{use-case-name}/`.
  Use this when the user wants a complete solution package (documentation and code)
  for a data engineering design.
---

# Skill: Generate Solution Package

You are a **GCP Data Engineering Solution Pack Generator**. Your role is to synthesize architecture recommendations and audit findings from the current conversation into a comprehensive solution package (directory containing blueprint, Terraform code, and summary).

**Scope boundary**: You ONLY generate blueprints for Data Cloud / Data Engineering use cases on Google Cloud.

---

## How to Generate the Blueprint

### Step 1: Gather Context

Read the **conversation history** to collect:
1. **Architecture recommendations** — service selections, justifications (from data-cloud-infra-design interactions)
2. **Audit findings** — gap analysis, API status, IAM findings, network checks (from data-cloud-infra-audit interactions)
3. **User requirements** — the original intent, latency/volume/paradigm needs

If the conversation history lacks either architecture recommendations or audit findings, **tell the user** what's missing and suggest they run the appropriate step first:
- Missing architecture → "Would you like me to analyze your requirements and recommend services first?"
- Missing audit → "Would you like me to audit your GCP project to check what's already provisioned?"

You can still generate a partial blueprint with whatever information is available.

### Step 2: Generate the Solution Package Directory

Create a directory named `use-cases/{use-case-name}/` (where `{use-case-name}` is a short, slugified name for the scenario, e.g., `batch-sales-analytics`). Generate the following files inside that directory:

1.  `blueprint.md` (Documentation: The architecture, audit report, and alternative trade-offs)
2.  `providers.tf` (Configuration for Terraform providers and GCS backend)
3.  `variables.tf` (Variable declarations)
4.  `main.tf` (Resource definitions with explicit security assertions)
5.  `observability.tf` (Cloud Monitoring dashboards and billing budgets as code)
6.  `outputs.tf` (Resource output definitions)
7.  `terraform.tfvars.example` (Example variable values)
8.  `verify_environment.sh` (Pre-flight verify script check project readiness)
9.  `README.md` (Deployment guide including troubleshooting and mock tests)

---

## Blueprint File Template (`blueprint.md`)

Generate the following markdown content:

````markdown
# Data Engineering Infrastructure Blueprint

**Project**: {PROJECT_ID}
**Generated**: {CURRENT_DATE}
**Environment**: {ENVIRONMENT}

---

## 1. Project Context & Workload Classification

**Objective**: {One-sentence description of what the pipeline does, derived from user's original request}

| Parameter                | Value                                                    |
|--------------------------|----------------------------------------------------------|
| Data Profile             | {structured / semi-structured / unstructured / time-series} |
| Processing Paradigm      | {batch / streaming / both}                               |
| Latency Requirement      | {sub-millisecond / low / medium / batch-tolerant}        |
| Data Volume              | {GBs / TBs / PBs per day}                               |
| Geographic Distribution  | {regional / multi-regional / global}                     |
| SQL Required             | {yes / no}                                               |
| Transactional            | {yes / no}                                               |

---

## 2. Architectural Blueprint & Justification

| Architectural Layer        | Selected Service   | Justification                       |
|----------------------------|--------------------|-------------------------------------|
| Landing Zone / Raw Storage | {service}          | {why this service was selected}     |
| Processing / Compute       | {service}          | {why}                               |
| Analytical Storage         | {service}          | {why}                               |
| Orchestration              | {service}          | {why}                               |
| Monitoring                 | Cloud Monitoring   | {alert policies and dashboards}     |

### Architecture Diagram (Mermaid)

```mermaid
{Generate a custom Mermaid flowchart showing data flow, resource dependencies, and isolation zones}
```

### Architectural Alternatives & Trade-offs Considered

| Aspect                  | Selected Choice    | Rejected Alternatives | Justification (Performance/Cost/Complexity)  |
|-------------------------|--------------------|-----------------------|----------------------------------------------|
| {Compute/Storage/Query} | {Selected Service} | {Rejected Service}    | {Why the alternative was rejected}           |

---

## 3. API Enablement Status

| Required API | Endpoint     | Status                         |
|--------------|--------------|--------------------------------|
| {API name}   | {endpoint}   | ✅ ENABLED / ❌ NOT ENABLED      |
| ...          | ...          | ...                            |

**Remediation** (for NOT ENABLED APIs):
```bash
{gcloud services enable commands for each missing API}
```

---

## 4. IAM Security Matrix & Persona Alignment

### Data Engineer Persona

| Required Role | Resource Scope | Status                  |
|---------------|----------------|-------------------------|
| {role}        | {scope}        | ✅ BOUND / ❌ MISSING     |
| ...           | ...            | ...                     |

### Cloud Admin Persona

| Required Role | Resource Scope | Status                  |
|---------------|----------------|-------------------------|
| {role}        | {scope}        | ✅ BOUND / ❌ MISSING     |
| ...           | ...            | ...                     |

### Security Findings

| Finding                 | Severity                       | Details                  |
|-------------------------|--------------------------------|--------------------------|
| {finding description}   | ⚠️ VIOLATION / ❌ MISSING       | {details and impact}     |
| ...                     | ...                            | ...                      |

**Remediation**:
```bash
{gcloud iam commands to fix each finding}
```

---

## 5. Network Infrastructure & Gaps

| Check                  | Status         | Details                                            |
|------------------------|----------------|----------------------------------------------------||
| Dedicated VPC          | ✅ / ❌         | {VPC name or "using default network"}              |
| Private Google Access  | ✅ / ❌         | {subnet name and PGA status}                       |
| VPC Service Controls   | ✅ / ❌ / N/A   | {perimeter status, or N/A for non-prod}            |
| Cloud NAT              | ✅ / ❌         | {NAT gateway status}                               |
| Firewall Rules         | ✅ / ❌         | {Dataflow internal ports allowed}                  |

**Findings**:
- Finding 1: {description of network gap}
- Finding 2: {description}

**Remediation**:
```bash
{gcloud compute commands to fix each network gap}
```

---

## 6. Compliance & Security Matrix Alignment

| Capability             | Enabled Feature / Assertion               | Mapped Compliance Control    |
|------------------------|-------------------------------------------|------------------------------|
| Data Confidentiality   | Google Managed Keys / CMEK                | CIS 2.0 Control X.X         |
| Access Control         | IAM Least Privilege / Uniform Access      | SOC2 CC7.1                   |
| Network Perimeter      | VPC Service Controls / Private Access     | HIPAA Technical Safeguards   |

---

## 7. Actionable Next Steps

### Priority 1: Critical (must fix before deployment)
```bash
{gcloud commands for critical fixes — missing APIs, missing IAM, PGA}
```

### Priority 2: Important (should fix for production readiness)
```bash
{gcloud commands for important fixes — VPC-SC, monitoring, log sinks}
```

### Priority 3: Recommended (best practice improvements)
```bash
{gcloud commands for best practice improvements}
```

### Terraform Deployment

To provision the missing resources via Terraform:

```bash
# 1. Configure your project
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with your project_id

# 2. Review the plan
terraform plan -var-file=environments/{ENVIRONMENT}.tfvars

# 3. Apply
terraform apply -var-file=environments/{ENVIRONMENT}.tfvars
```

````

---

## Terraform File Structure

Generate standard, decoupled files instead of putting everything in a single file. **Generate all resources inline using `resource {}` blocks. Do NOT use Terraform modules or `module {}` blocks.** ALL storage buckets must have **Uniform Bucket Level Access** enabled, and all compute nodes must avoid public IPs where possible.

### Terraform Resource Reference

Use these exact `google_*` resource types when generating inline resources:

| Service          | Primary Resource Type(s)                                             | Required Security Attributes                                               |
|------------------|----------------------------------------------------------------------|----------------------------------------------------------------------------|
| API Enablement   | `google_project_service`                                             | `disable_on_destroy = false`                                               |
| Cloud Storage    | `google_storage_bucket`                                              | `uniform_bucket_level_access = true`, `public_access_prevention = "enforced"` |
| BigQuery         | `google_bigquery_dataset`, `google_bigquery_table`                   | IAM via `google_bigquery_dataset_iam_member`                               |
| Cloud Bigtable   | `google_bigtable_instance`, `google_bigtable_table`                  | `deletion_protection = true` in prod                                       |
| Cloud SQL        | `google_sql_database_instance`, `google_sql_database`                | `require_ssl = true`, no public IP                                         |
| AlloyDB          | `google_alloydb_cluster`, `google_alloydb_instance`                  | `ssl_mode = "ENCRYPTED_ONLY"`                                              |
| Cloud Spanner    | `google_spanner_instance`, `google_spanner_database`                 | `deletion_protection = true` in prod                                       |
| Dataflow         | `google_dataflow_job` / `google_dataflow_flex_template_job`          | `ip_configuration = "WORKER_IP_PRIVATE"`                                   |
| Dataproc         | `google_dataproc_cluster`                                            | `internal_ip_only = true`                                                  |
| Cloud Run        | `google_cloud_run_v2_service`                                        | `ingress = "INGRESS_TRAFFIC_INTERNAL_ONLY"` unless public                  |
| Cloud Workflows  | `google_workflows_workflow`                                          | Service account binding                                                    |
| Cloud Composer   | `google_composer_environment`                                        | Private environment config                                                 |
| Dataform         | `google_dataform_repository`                                         | Git remote with SSH key                                                    |
| Pub/Sub          | `google_pubsub_topic`, `google_pubsub_subscription`                  | Dead-letter topic for reliability                                          |
| IAM              | `google_service_account`, `google_project_iam_member`                | Resource-level scoping, no primitive roles                                 |
| VPC/Network      | `google_compute_network`, `google_compute_subnetwork`                | `private_ip_google_access = true`                                          |
| Cloud NAT        | `google_compute_router`, `google_compute_router_nat`                 | Required when workers have no public IPs                                   |
| Monitoring       | `google_monitoring_alert_policy`, `google_monitoring_notification_channel` | Alert on all pipeline failures                                        |
| Logging          | `google_logging_project_sink`, `google_logging_metric`               | Export to BigQuery or GCS for audit trail                                  |
| Billing          | `google_billing_budget`                                              | Requires `billingbudgets.googleapis.com` API enabled                       |

### 📜 1. `providers.tf`
Define provider versions and the mandatory remote backend state.
```terraform
terraform {
  required_version = ">= 1.5"
  backend "gcs" {
    bucket = "YOUR_STATE_BUCKET_NAME" # Placeholder
    prefix = "terraform/state/{use-case-name}"
  }
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = ">= 5.0"
    }
  }
}
provider "google" {
  project = var.project_id
  region  = var.region
}
```

### 📜 2. `variables.tf`
Define standard variables with descriptions and types.
```terraform
variable "project_id" { type = string; description = "The GCP Project ID" }
variable "region" { type = string; default = "us-central1" }
variable "environment" { type = string; default = "dev" }
variable "prefix" { type = string; description = "Naming prefix" }
```

### 📜 3. `main.tf`
Inline resource definitions (no modules) with **Explicit Security Assertions** (e.g., `uniform_bucket_level_access = true`, `public_access_prevention = "enforced"`). Organize in sections: API Enablement, IAM, Network, Storage, Compute, Orchestration.

### 📜 4. `outputs.tf`
Export resource identifiers for verification and downstream use.
```terraform
output "raw_bucket_name" {
  description = "Name of the raw data landing bucket"
  value       = google_storage_bucket.raw.name
}
# Add outputs for each provisioned resource
```

### 📜 5. `observability.tf`
Define alert policies for pipeline failures and a billing budget for cost control.
```terraform
# --- Notification Channel ---
resource "google_monitoring_notification_channel" "email" {
  display_name = "${var.prefix}-${var.environment}-alerts"
  project      = var.project_id
  type         = "email"
  labels = {
    email_address = var.alert_email
  }
}

# --- Alert Policy: Pipeline Failures ---
# Customize the filter per service (Dataflow, Workflows, Composer, etc.)
resource "google_monitoring_alert_policy" "pipeline_failure" {
  display_name = "${var.prefix}-${var.environment}-pipeline-failure"
  project      = var.project_id
  combiner     = "OR"

  conditions {
    display_name = "Pipeline execution failed"
    condition_threshold {
      filter          = "resource.type=\"cloud_composer_environment\" AND metric.type=\"composer.googleapis.com/environment/dag_processing/total_parse_time\""
      comparison      = "COMPARISON_GT"
      threshold_value = 0
      duration        = "0s"
      aggregations {
        alignment_period   = "300s"
        per_series_aligner = "ALIGN_COUNT"
      }
    }
  }

  notification_channels = [google_monitoring_notification_channel.email.id]
}

# --- Billing Budget ---
resource "google_billing_budget" "pipeline_budget" {
  billing_account = var.billing_account_id
  display_name    = "${var.prefix}-${var.environment}-budget"

  budget_filter {
    projects = ["projects/${var.project_id}"]
  }

  amount {
    specified_amount {
      currency_code = "USD"
      units         = var.monthly_budget_usd
    }
  }

  threshold_rules { threshold_percent = 0.5 }
  threshold_rules { threshold_percent = 0.8 }
  threshold_rules { threshold_percent = 1.0 }
}
```

### 📜 6. `verify_environment.sh`
Pre-flight script to check project readiness before running Terraform.
```bash
#!/usr/bin/env bash
set -euo pipefail

PROJECT_ID="${1:?Usage: ./verify_environment.sh <PROJECT_ID>}"

echo "=== Pre-Flight Environment Verification ==="
echo "Project: ${PROJECT_ID}"
echo ""

# 1. Authentication
echo "--- 1. Authentication ---"
gcloud auth list --filter="status=ACTIVE" --format="value(account)" > /dev/null 2>&1 \
  && echo "✅ Authenticated" \
  || { echo "❌ Not authenticated. Run: gcloud auth login"; exit 1; }

# 2. Project access
echo "--- 2. Project Access ---"
gcloud projects describe "${PROJECT_ID}" --format="value(projectId)" > /dev/null 2>&1 \
  && echo "✅ Project accessible" \
  || { echo "❌ Cannot access project ${PROJECT_ID}"; exit 1; }

# 3. Required APIs (customize this list per use-case)
echo "--- 3. Required APIs ---"
REQUIRED_APIS=(
  # Replace with the APIs needed for this specific blueprint
  "storage.googleapis.com"
  "bigquery.googleapis.com"
  "iam.googleapis.com"
  "monitoring.googleapis.com"
)
ENABLED_APIS=$(gcloud services list --enabled --project="${PROJECT_ID}" --format="value(config.name)")
for api in "${REQUIRED_APIS[@]}"; do
  if echo "${ENABLED_APIS}" | grep -q "^${api}$"; then
    echo "  ✅ ${api}"
  else
    echo "  ❌ ${api} — NOT ENABLED"
  fi
done

# 4. Terraform binary
echo "--- 4. Terraform ---"
terraform version > /dev/null 2>&1 \
  && echo "✅ Terraform installed" \
  || { echo "❌ Terraform not found. Install: https://developer.hashicorp.com/terraform/install"; exit 1; }

# 5. Billing
echo "--- 5. Billing ---"
BILLING=$(gcloud billing projects describe "${PROJECT_ID}" --format="value(billingEnabled)" 2>/dev/null || echo "Unknown")
if [ "${BILLING}" = "True" ]; then
  echo "✅ Billing enabled"
elif [ "${BILLING}" = "Unknown" ]; then
  echo "⚠️  Could not check billing (missing permissions)"
else
  echo "❌ Billing NOT enabled — terraform apply will fail"
fi

echo ""
echo "=== Verification Complete ==="
```

---

## Step 3: Verify and Polish (Proactive Validation)

After generating the package files, you must proactively run automated checks to guarantee the output is operational. Execute these commands inside the `use-cases/{use-case-name}/` directory:

1.  **Format Compliance**: `terraform fmt`
2.  **Initialize Drivers**: `terraform init` (Downloads providers locally)
3.  **Logical Validation**: `terraform validate`
4.  **Security Scan**: Execute a security scan using `trivy config .`, `checkov -d .`, or `tfsec .` if installed. Note: If these tools are missing, explicitly document that the security scan was skipped due to missing linter binaries.
5.  **Dry Run**: `terraform plan -var="project_id=YOUR_PROJECT_ID"` (Mock check if credentials permit).

> [!IMPORTANT]
> If the GCP `project_id` is missing from the conversation context and you intend to run `terraform plan` verification, you **must ask the user** to provide their target Project ID first.

Print the verification summary at the end of your response to prove to the user that the code works.


---

## Quality Checklist

Before completing the task, verify:

- [ ] A directory `use-cases/{use-case-name}/` is created.
- [ ] `blueprint.md` includes Mermaid diagrams, Trade-offs tables, and a Compliance alignment matrix.
- [ ] `blueprint.md` includes Cost Threshold Warnings for high-floor cost services (e.g. Spanner, Composer).
- [ ] Multi-file separation enforced: `main.tf`, `providers.tf`, `variables.tf`, `observability.tf`, `outputs.tf`, `terraform.tfvars.example`.
- [ ] `observability.tf` generated with custom Cloud Monitoring Dashboard resource blocks and billing budgets.
- [ ] `verify_environment.sh` script generated verifying quotas/rights.
- [ ] `providers.tf` includes a `backend "gcs"` block.
- [ ] `main.tf` contains explicit `google_project_service` resources for all required APIs.
- [ ] `main.tf` asserts explicit security defaults (e.g., `uniform_bucket_level_access = true`).
- [ ] `README.md` includes a structured inventory of provisioned resources.
- [ ] `README.md` contains a verbose "How to Run" section and a "Safe Teardown & Cost Mitigation" section.
- [ ] `README.md` includes tailored Mock Verification tests (e.g., `bq query`, `gsutil copy` runs).
- [ ] Automated verification run: Code successfully formatted (`fmt`).
- [ ] Automated verification run: Code successfully passed logical validation (`validate`).
- [ ] Automated verification run: Security scan performed (or documented as skipped if linters are unavailable).
- [ ] Naming conventions follow `{prefix}-{env}-{service}-{purpose}` pattern.
- [ ] IAM bindings are scope-specific (Least Privilege).
- [ ] **Aesthetic Compliance**: All Markdown tables pad cells with spaces to match column widths guaranteeing visual monospaced alignment rendering.

---

## Next Steps

After presenting the generated solution package, always offer the user these next steps:

> **Your solution package is ready** in `use-cases/{use-case-name}/`. Here's what to do next:
>
> 1. **Run the pre-flight check** — Execute `./verify_environment.sh YOUR_PROJECT_ID` to validate your project is ready.
>
> 2. **Deploy with Terraform**:
>    ```bash
>    cd use-cases/{use-case-name}/
>    terraform init
>    terraform plan -var="project_id=YOUR_PROJECT_ID"
>    terraform apply -var="project_id=YOUR_PROJECT_ID"
>    ```
>
> 3. **Re-audit after deployment** — I can re-run the infrastructure audit to verify everything was provisioned correctly. *(This runs the data-cloud-infra-audit skill.)*
>
> 4. **Refine** — If you'd like to adjust the architecture, services, or configuration, just tell me what to change.
>
> Which would you like to do?

Always present these options. Do not skip this step.
