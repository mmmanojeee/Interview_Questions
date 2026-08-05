# Terraform on Azure — Advanced Interview Questions

A curated set of advanced Terraform + Azure interview questions, organized by domain. Each question includes a **Focus** note pointing to the key concepts, tools, or Azure services an answer should cover.

---

## Table of Contents

1. [Domain 1: Azure State Architecture, Blob Storage & Lock Mechanics](#domain-1-azure-state-architecture-blob-storage--lock-mechanics)
2. [Domain 2: Advanced HCL, Dynamic Logic & Azure Providers](#domain-2-advanced-hcl-dynamic-logic--azure-providers)
3. [Domain 3: Module Architecture & Azure Governance](#domain-3-module-architecture--azure-governance)
4. [Domain 4: Security, Identity & Secrets in Azure](#domain-4-security-identity--secrets-in-azure)
5. [Domain 5: Enterprise CI/CD & Pipeline Governance](#domain-5-enterprise-cicd--pipeline-governance)
6. [Domain 6: Advanced Scenario & Troubleshooting Questions (Azure Focus)](#domain-6-advanced-scenario--troubleshooting-questions-azure-focus)

---

## Domain 1: Azure State Architecture, Blob Storage & Lock Mechanics

### 1.1 How does Terraform manage state locking natively using Azure Storage Accounts (Blob Storage) without relying on external databases?
**Focus:** Blob Leases (azurerm backend native lease management), storage account RBAC, Lease ID behavior, and handling broken/stuck leases (`az storage blob lease break`) during pipeline failures.

### 1.2 Scenario: An engineer manually modified an Azure Network Security Group (NSG) rule via the Azure Portal. During `terraform plan`, Terraform attempts to revert it, but security requires keeping the change. How do you reconcile this without destroying or recreating the NSG?
**Focus:** `terraform refresh`, state target updates, `import`, and HCL refactoring using Azure native properties.

### 1.3 How do you protect Terraform state files from corruption, accidental deletion, or secret leakage in Azure Blob Storage?
**Focus:** Blob Versioning, Soft Delete, Private Endpoints, SSE with Customer-Managed Keys (KMS/Key Vault), Azure AD RBAC, and Microsoft Defender for Storage.

### 1.4 Scenario: You have a monolithic state file containing 2,000 Azure resources (AKS, VNets, Azure SQL, App Services) taking 25+ minutes per `terraform plan`. How do you refactor this state without incurring application downtime?
**Focus:** Splitting state using `terraform state mv` vs `moved {}` blocks, module decoupling, Azure Key Vault parameters for cross-state references, and subscription boundaries.

### 1.5 Compare `terraform state rm`, `terraform state mv`, and `moved {}` blocks in the context of Azure resource refactoring.
**Focus:** Structural changes (e.g., moving an `azurerm_subnet` into a module) without triggering resource recreation or ARM API deletion calls.

---

## Domain 2: Advanced HCL, Dynamic Logic & Azure Providers

### 2.1 How do you handle multi-subscription or multi-tenant deployments using the `azurerm` provider?
**Focus:** Provider aliases (`alias = "hub"`), passing explicit providers to sub-modules, Service Principal cross-tenant authentication, and subscription ID targeting.

### 2.2 Scenario: Write/describe an HCL snippet using dynamic blocks and `for_each` to generate nested Azure Network Security Group rules dynamically based on a map variable.
**Focus:** Dynamic `security_rule` blocks within `azurerm_network_security_group`, handling null attributes, and maintainability.

### 2.3 Explain the execution timing and difference between `precondition` and `postcondition` blocks in an Azure deployment.
**Focus:** Guardrails (e.g., verifying an Azure Resource Group exists before provisioning, or confirming an Azure SQL Server public access setting is Disabled post-provisioning).

### 2.4 How do `count` and `for_each` differ when provisioning multiple Azure Virtual Machines or Subnets?
**Focus:** Array indices vs map key references, avoiding mass VM recreation when inserting a new subnet or VM into a list.

### 2.5 What are `check` blocks in Terraform 1.5+, and how would you use them for continuous health checks on Azure resources?
**Focus:** Continuous validation of Azure endpoints, non-blocking plans, and integration with Azure Monitor/HCP Terraform.

---

## Domain 3: Module Architecture & Azure Governance

### 3.1 How do you design a re-usable corporate Azure module registry across 50+ enterprise engineering teams?
**Focus:** Azure-aligned naming conventions, semantic versioning, private module registry (Azure DevOps Artifacts / Terraform Cloud), input validation rules, and mandatory tag inheritance.

### 3.2 How do you enforce organizational Azure Policies (Azure Governance) alongside Terraform deployments?
**Focus:** Managing `azurerm_policy_assignment` via Terraform vs handling conflict when Azure Policy mutates resources post-apply (e.g., auto-injecting tags or log analytics agents).

### 3.3 Scenario: You update a shared Azure Core Networking module from v1.0.0 to v2.0.0 introducing breaking variable changes. How do you roll this out across 100 microservices without breaking pipelines?
**Focus:** Version locking (`~> 1.0`), deprecation notices, automated PR strategies (Dependabot/Renovate), and blue-green refactoring.

### 3.4 What is the `terraform_remote_state` data source antipattern in Azure environments, and what are the modern alternatives?
**Focus:** Tight state coupling risks; alternatives using Azure Key Vault, App Configuration, or native `azurerm` data sources.

---

## Domain 4: Security, Identity & Secrets in Azure

### 4.1 How do you implement passwordless, keyless authentication for Terraform CI/CD pipelines executing against Azure?
**Focus:** Workload Identity Federation (OpenID Connect / OIDC) with Azure DevOps or GitHub Actions, eliminating long-lived Service Principal client secrets.

### 4.2 How do you protect sensitive data (e.g., Azure SQL admin passwords or Connection Strings) from leaking into plain-text `.tfstate` files stored in Azure Blob Storage?
**Focus:** `sensitive = true` limitations, Azure Key Vault integration, short-lived credentials, and storage account access controls.

### 4.3 Scenario: A developer accidentally committed an Azure Service Principal secret and a state file containing database connection strings to a public Git repo. What is your emergency response workflow?
**Focus:** Secret revocation, Azure AD app registration credential rotation, state file scrubbing, Key Vault secret rotation, and git history cleanup.

### 4.4 Compare Azure Native Policy enforcement with Policy-as-Code tools like Checkov, Trivy, and OPA/Sentinel during CI/CD.
**Focus:** Shift-left static scanning vs runtime Azure ARM/Bicep/Policy enforcement.

---

## Domain 5: Enterprise CI/CD & Pipeline Governance

### 5.1 Explain the architecture of a production-grade Azure DevOps (or GitHub Actions) pipeline for Terraform managing Dev, Staging, and Prod Azure subscriptions.
**Focus:** Pull request plan output, automated validation (`terraform fmt`, `tflint`), Azure Blob backend separation per environment, and manual approval gates.

### 5.2 Scenario: Two concurrent Azure DevOps pipeline runs trigger `terraform apply` on the same Azure resource group state. How does Azure Storage handle this, and how do you recover if a lease gets stuck?
**Focus:** Azure Blob lease mechanics, lease breaking via CLI/PowerShell, and pipeline queue controls.

### 5.3 How do you execute speculative plans safely on PRs targeting Azure without exposing production credentials or state?
**Focus:** Azure Reader-role Service Principals for PR pipelines, ephemeral testing resource groups, and secret isolation.

---

## Domain 6: Advanced Scenario & Troubleshooting Questions (Azure Focus)

### 6.1 Scenario: `terraform apply` fails halfway through an Azure Landing Zone deployment (e.g., VNet created, Virtual Network Gateway timed out after 45 minutes). How do you recover cleanly?
**Focus:** Analyzing partial state, handling long-running Azure resource timeouts, `terraform target`, and re-running pipelines idempotently.

### 6.2 Scenario: An Azure Key Vault managed by Terraform has purge protection enabled. Terraform attempts to destroy and recreate it due to a name change, but fails because the Vault name is reserved in soft-deleted state. How do you resolve this?
**Focus:** Azure Key Vault soft-delete lifecycle, `moved {}` blocks, or manually recovering/purging soft-deleted vaults via Azure CLI.

### 6.3 Scenario: You need to migrate 150 manually created Azure VMs, VNets, and Storage Accounts into Terraform management. What is your automated migration strategy?
**Focus:** `terraform import`, declarative `import {}` blocks (Terraform 1.5+), and tools like AZTFExport (Azure Terrafy).

### 6.4 Scenario: Your company requires a multi-region deployment across Azure East US and West Europe with Traffic Manager or Azure Front Door. How do you structure your `azurerm` providers and modules?
**Focus:** Provider aliases (`azurerm.eastus`, `azurerm.westeurope`), cross-region state outputs, and global resource orchestration.

---

## How to Use This Guide

- **Interviewers:** Pick 1–2 questions per domain based on the seniority level being assessed (Domains 1–2 for mid-level, Domains 3–6 for senior/lead roles).
- **Candidates:** Use the **Focus** notes as a checklist — a strong answer should touch on most of the listed concepts, not just define the primary term.
- **Scenario questions** are best answered with a structured approach: *diagnose → immediate mitigation → long-term fix → prevention*.