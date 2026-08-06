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

---

Here is a **seventh set** of brand-new, unique Terraform interview questions and answers tailored specifically for Microsoft Azure. None of these repeat topics covered previously.
### Basic Level Questions
#### 1. What is the difference between terraform apply and terraform apply -auto-approve? When should -auto-approve be used?
 * **Answer:**
   * terraform apply: Generates an execution plan, displays all proposed infrastructure additions, modifications, or deletions, and prompts the user for an explicit interactive confirmation (yes) before applying changes to Azure.
   * terraform apply -auto-approve: Bypasses the interactive prompt and immediately applies the changes.
   * **Use Case:** -auto-approve is designed for automated non-interactive CI/CD pipelines (e.g., Azure DevOps or GitHub Actions) where an explicit plan approval gate has already taken place in a prior review stage.
#### 2. What is the difference between a Provider and a Module in Terraform, and how do you declare provider requirements inside a child module?
 * **Answer:**
   * **Provider:** A plugin (e.g., azurerm or azuread) that translates HCL code into API calls to manage cloud resources.
   * **Module:** A collection of .tf files in a directory that groups related Azure resources together for reusability.
   * **Child Module Provider Declaration:** Child modules should declare required providers and minimum versions inside a required_providers block, but **should not** configure provider instantiation parameters (like authentication credentials or features {} blocks). Provider configurations belong exclusively in the root module.
#### 3. How does location inheritance work between an Azure Resource Group and the resources inside it?
 * **Answer:**
   * Defining location on an azurerm_resource_group specifies where the resource group’s **metadata** is stored.
   * Resources within that resource group do **not** automatically inherit its region. Each child resource (e.g., azurerm_virtual_network) explicitly sets its own location parameter.
   * **Best Practice:** Pass the resource group's location property directly to dependent child resources (location = azurerm_resource_group.rg.location) to ensure regional alignment and avoid unexpected cross-region latency or bandwidth costs.
### Intermediate Level Questions
#### 4. How do you implement global resource tagging consistently across Azure resources using default_tags?
 * **Answer:**
   * Instead of manually specifying tag maps on every individual resource block, you can configure default_tags at the azurerm provider level.
   * **Implementation:**
     ```hcl
     provider "azurerm" {
       features {}
       default_tags {
         tags = {
           Environment = var.environment
           ManagedBy   = "Terraform"
           CostCenter  = "IT-Ops"
         }
       }
     }
     
     ```
   * **Behavior:** Terraform automatically merges default tags into every taggable Azure resource created under that provider instance. Resource-level tags take precedence if key conflicts occur.
#### 5. How do you handle Azure Kubernetes Service (AKS) node pool upgrades and dynamic expansion using Terraform?
 * **Answer:**
   * **Dynamic Node Pools:** Use for_each on azurerm_kubernetes_cluster_node_pool to provision additional system or user node pools dynamically based on workload configuration maps.
   * **Safe Cluster Upgrades:**
     * Explicitly set orchestrator_version and use lifecycle { ignore_changes = [ default_node_pool[0].node_count ] } if Azure autoscaling (cluster-autoscaler) is enabled, preventing Terraform from constantly resetting node counts.
     * Define upgrade_settings inside node pool definitions to configure max surge parameters (e.g., max_surge = "33%"), ensuring zero-downtime rolling upgrades when bumping Kubernetes versions.
#### 6. How do you configure Customer-Managed Keys (CMK) for Azure Storage Account or Key Vault encryption in Terraform?
 * **Answer:**
   * Encrypting data at rest using CMK requires chaining dependencies across Key Vault, Managed Identity, and Storage components:
     1. Create a User-Assigned Managed Identity (azurerm_user_assigned_identity).
     2. Grant the Identity Key Vault access permissions (azurerm_key_vault_access_policy or Azure RBAC role).
     3. Generate a Key Vault Key (azurerm_key_vault_key).
     4. Link the key and identity to the storage account via azurerm_storage_account_customer_managed_key:
       ```hcl
       resource "azurerm_storage_account_customer_managed_key" "cmk" {
         storage_account_id = azurerm_storage_account.st.id
         key_vault_id       = azurerm_key_vault.kv.id
         key_name           = azurerm_key_vault_key.key.name
         user_assigned_identity_id = azurerm_user_assigned_identity.identity.id
       }
       
       ```
### Advanced Level Questions
#### 7. How do you resolve stuck backend state locks (terraform force-unlock) when using Azure Blob Storage?
 * **Answer:**
   * **Cause:** If a CI/CD job crashes or network connectivity breaks during terraform apply, the lease on the Azure Blob holding the terraform.tfstate file may remain locked.
   * **Resolution Steps:**
     1. Retrieve the **Lock ID** displayed in the CLI error message.
     2. Ensure no active pipeline or engineer is running an apply operation.
     3. Execute:
       ```bash
       terraform force-unlock <LOCK-ID>
       
       ```
     4. **Alternative (Azure Portal / CLI):** If force-unlock fails, break the lease directly on the target .tfstate blob within the Azure Storage Account via Azure Portal or az storage blob lease break.
#### 8. How do you combine azurerm, kubernetes, and helm providers in a single Terraform project to deploy infra and apps to AKS?
 * **Answer:**
   * **Architecture:** Use azurerm to build the AKS cluster, and pass cluster credentials dynamically into the kubernetes and helm providers using provider configuration blocks.
   * **Example Configuration:**
     ```hcl
     provider "azurerm" {
       features {}
     }
     
     resource "azurerm_kubernetes_cluster" "aks" {
       name                = "aks-prod"
       location            = azurerm_resource_group.rg.location
       resource_group_name = azurerm_resource_group.rg.name
       dns_prefix          = "aksprod"
       # ...
     }
     
     provider "kubernetes" {
       host                   = azurerm_kubernetes_cluster.aks.kube_config.0.host
       client_certificate     = base64decode(azurerm_kubernetes_cluster.aks.kube_config.0.client_certificate)
       client_key             = base64decode(azurerm_kubernetes_cluster.aks.kube_config.0.client_key)
       cluster_ca_certificate = base64decode(azurerm_kubernetes_cluster.aks.kube_config.0.cluster_ca_certificate)
     }
     
     provider "helm" {
       kubernetes {
         host                   = azurerm_kubernetes_cluster.aks.kube_config.0.host
         client_certificate     = base64decode(azurerm_kubernetes_cluster.aks.kube_config.0.client_certificate)
         client_key             = base64decode(azurerm_kubernetes_cluster.aks.kube_config.0.client_key)
         cluster_ca_certificate = base64decode(azurerm_kubernetes_cluster.aks.kube_config.0.cluster_ca_certificate)
       }
     }
     
     ```
   * **Enterprise Best Practice:** For production setups, separate the cluster provisioning (infrastructure) code from the application deployment (Helm/Kubernetes) code into distinct state files to avoid authentication lookup issues on initial cluster creation.
#### 9. How do you automate Azure Private Endpoint DNS Zone Links and Private Endpoint DNS record registration in Terraform?
 * **Answer:**
   * When attaching a Private Endpoint to a PaaS service (e.g., Azure SQL or Key Vault), private IP resolution requires an Azure Private DNS Zone.
   * **Steps:**
     1. Create the Private DNS Zone (e.g., privatelink.vaultcore.azure.net) using azurerm_private_dns_zone.
     2. Link the Private DNS Zone to the local Virtual Network using azurerm_private_dns_zone_virtual_network_link.
     3. Define an azurerm_private_endpoint and configure its private_dns_zone_group block:
       ```hcl
       private_dns_zone_group {
         name                 = "kv-dns-zone-group"
         private_dns_zone_ids = [azurerm_private_dns_zone.kv_dns.id]
       }
       
       ```
     4. Azure automatically manages the creation and lifecycle of the target A record inside the Private DNS Zone matching the allocated private IP.

---

Here is an **eighth set** of unique, real-world Terraform interview questions and answers specifically tailored for Microsoft Azure. To help you prepare thoroughly, each technical concept includes concrete **HCL code examples** and architectural scenarios.
## Basic Level Questions
### 1. What is the difference between explicit outputs and the data source in Terraform?
 * **Answer:** * **Output Values (output):** Expose resource attributes from a Terraform state file so they can be consumed by CLI tools, higher-level root modules, or external automation steps.
   * **Data Sources (data):** Query the live Azure Resource Manager (ARM) API to fetch properties of resources that already exist in Azure—whether created manually, via Azure Portal, or through a completely different codebase.
 * **Example:** Fetching an existing Virtual Network's ID using a data block versus exposing a newly created subnet's ID via an output block:
```hcl
# Data Source: Reads existing Azure VNet
data "azurerm_virtual_network" "existing_vnet" {
  name                = "vnet-core-prod"
  resource_group_name = "rg-networking-prod"
}

# Output: Exposes the ID of a newly created subnet
output "app_subnet_id" {
  value       = azurerm_subnet.app.id
  description = "The ID of the newly created App Subnet."
}

```
### 2. How do you construct conditional resource creation using the ternary operator in HCL?
 * **Answer:** You can combine the ternary operator (condition ? true_value : false_value) with the count meta-argument. If the boolean condition evaluates to true, count is set to 1 (creating the resource); otherwise, count is set to 0 (skipping creation).
 * **Example:** Conditionally creating an Azure Public IP based on an input variable:
```hcl
variable "enable_public_ip" {
  type    = bool
  default = false
}

resource "azurerm_public_ip" "pip" {
  count               = var.enable_public_ip ? 1 : 0
  name                = "pip-app-prod"
  resource_group_name = "rg-app"
  location            = "East US"
  allocation_method   = "Static"
  sku                 = "Standard"
}

```
## Intermediate Level Questions
### 3. How do you implement Zero-Downtime deployment for an Azure Virtual Machine or Scale Set using lifecycle rules?
 * **Answer:** By default, if an attribute change forces a resource recreation in Azure (such as changing an OS image SKU or network configuration), Terraform destroys the old resource *before* creating the replacement. Applying create_before_destroy = true forces Terraform to provision the new Azure resource first, verify its creation, and then destroy the old one.
 * **Example:** Protecting an Azure Network Interface from downtime during updates:
```hcl
resource "azurerm_network_interface" "nic" {
  name                = "nic-web-prod"
  location            = "East US"
  resource_group_name = "rg-web"

  ip_configuration {
    name                          = "internal"
    subnet_id                     = "/subscriptions/.../subnets/snet-web"
    private_ip_address_allocation = "Dynamic"
  }

  lifecycle {
    create_before_destroy = true
  }
}

```
### 4. How do you assign an Azure User-Assigned Managed Identity to an App Service and grant it Access Key permissions on Key Vault?
 * **Answer:** You must link three components in sequence:
   1. Define the User-Assigned Identity (azurerm_user_assigned_identity).
   2. Assign the identity to the Azure App Service via the identity configuration block.
   3. Create an Azure RBAC Role Assignment (azurerm_role_assignment) granting the identity "Key Vault Secrets User" rights.
 * **Example:**
```hcl
# 1. Create User-Assigned Identity
resource "azurerm_user_assigned_identity" "app_id" {
  name                = "id-webapp-prod"
  location            = "East US"
  resource_group_name = "rg-app"
}

# 2. Attach Identity to App Service
resource "azurerm_linux_web_app" "app" {
  name                = "app-orderservice-prod"
  location            = "East US"
  resource_group_name = "rg-app"
  service_plan_id     = "/subscriptions/.../appServicePlans/plan-prod"

  identity {
    type         = "UserAssigned"
    identity_ids = [azurerm_user_assigned_identity.app_id.id]
  }

  site_config {}
}

# 3. Assign Key Vault Secrets User Role
resource "azurerm_role_assignment" "kv_read" {
  scope                = "/subscriptions/.../resourceGroups/rg-sec/providers/Microsoft.KeyVault/vaults/kv-prod"
  role_definition_name = "Key Vault Secrets User"
  principal_id         = azurerm_user_assigned_identity.app_id.principal_id
}

```
### 5. How do you configure an Azure Storage Account with Network Rules to restrict access strictly to specific Subnets?
 * **Answer:** You use the network_rules sub-block inside azurerm_storage_account. You set default_action = "Deny" and supply authorized subnet IDs via virtual_network_subnet_ids. The target subnets must have the Microsoft.Storage Service Endpoint or Private Endpoint enabled.
 * **Example:**
```hcl
resource "azurerm_storage_account" "secure_st" {
  name                     = "stsecureappprod"
  resource_group_name      = "rg-data"
  location                 = "East US"
  account_tier             = "Standard"
  account_replication_type = "LRS"

  network_rules {
    default_action             = "Deny"
    bypass                     = ["AzureServices"]
    virtual_network_subnet_ids = ["/subscriptions/.../subnets/snet-app"]
    ip_rules                   = ["203.0.113.10"] # Trusted Office IP
  }
}

```
## Advanced Level Questions
### 6. How do you safely move an existing Azure resource into a Child Module using the moved block without destroying it?
 * **Answer:** Prior to Terraform 1.1, moving a resource into a module required manually modifying state with terraform state mv. With moved blocks, you declare the structural refactoring directly in HCL code. When terraform plan runs, it detects the address shift and updates the state file non-destructively.
 * **Example:** Moving a Storage Account into a reusable module:
```hcl
# Before refactoring: azurerm_storage_account.app_storage
# After refactoring: module.storage.azurerm_storage_account.this

moved {
  from = azurerm_storage_account.app_storage
  to   = module.storage.azurerm_storage_account.this
}

```
### 7. How do you construct complex nested inputs using for expressions to dynamically map Azure Subnets and Route Tables?
 * **Answer:** In enterprise hub-and-spoke models, subnets are often defined as a map of objects. You can iterate over this map using for_each and extract calculated properties dynamically.
 * **Example:** Constructing multiple Azure Subnets dynamically from a complex local variable map:
```hcl
locals {
  subnets = {
    "snet-web" = { cidr = "10.0.1.0/24", delegate = false }
    "snet-app" = { cidr = "10.0.2.0/24", delegate = false }
    "snet-aci" = { cidr = "10.0.3.0/24", delegate = true  }
  }
}

resource "azurerm_subnet" "subnets" {
  for_each             = locals.subnets
  name                 = each.key
  resource_group_name  = "rg-networking"
  virtual_network_name = "vnet-prod"
  address_prefixes     = [each.value.cidr]

  dynamic "delegation" {
    for_each = each.value.delegate ? [1] : []
    content {
      name = "aci-delegation"
      service_delegation {
        name    = "Microsoft.ContainerInstance/containerGroups"
        actions = ["Microsoft.Network/virtualNetworks/subnets/action"]
      }
    }
  }
}

```
### 8. How do you prevent sensitive outputs from leaking into Terraform state files and CI/CD logs when generating Azure VM admin passwords?
 * **Answer:** 1. Generate the random password using the random_password provider resource.
   2. Mark input variables or output definitions containing secrets as sensitive = true.
   3. Pass the generated password directly to the Azure VM resource or store it in Azure Key Vault without echoing it to console outputs.
 * **Example:**
```hcl
# Generate secure password
resource "random_password" "vm_password" {
  length           = 16
  special          = true
  override_special = "!#$%&*()-_=+[]{}<>:?"
}

# Store directly in Azure Key Vault
resource "azurerm_key_vault_secret" "vm_secret" {
  name         = "vm-admin-password"
  value        = random_password.vm_password.result
  key_vault_id = "/subscriptions/.../vaults/kv-prod"
}

# Expose output while hiding value from CLI logs
output "kv_secret_uri" {
  value       = azurerm_key_vault_secret.vm_secret.versionless_id
  sensitive   = true # Suppresses output in terminal logs
}

```
