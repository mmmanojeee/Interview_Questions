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

---

Here is a **ninth set** of interview questions and practical HCL code examples focusing on advanced, scenario-based Azure Terraform configurations.
## Basic Level Questions
### 1. How do you override default variable values using a custom .tfvars file during terraform plan or apply?
 * **Answer:** You can explicitly pass a custom variable definitions file at execution time using the -var-file command-line flag.
 * **Scenario:** Applying a specific staging configuration file (staging.tfvars) instead of defaults.
```hcl
# execution command:
# terraform plan -var-file="environments/staging.tfvars"

```
```hcl
# environments/staging.tfvars
environment_name = "staging"
vm_count         = 2
sku_tier         = "Standard_B2s"

```
### 2. How do you append custom tags to default tags declared in the Azure provider?
 * **Answer:** Individual resource blocks automatically inherit provider-level default_tags. When you define specific tags inside a resource block, Terraform merges them, letting the resource-level tags override matching keys or add unique ones.
 * **Example:**
```hcl
provider "azurerm" {
  features {}
  default_tags {
    tags = {
      Owner     = "DevOps-Team"
      ManagedBy = "Terraform"
    }
  }
}

resource "azurerm_resource_group" "rg" {
  name     = "rg-app-prod"
  location = "East US"

  # Adds or overrides tags for this specific resource
  tags = {
    CostCenter = "CC-1092"
    Owner      = "App-Team" # Overrides provider default tag
  }
}

```
## Intermediate Level Questions
### 3. How do you implement Azure App Service Deployment Slots using Terraform for zero-downtime application releases?
 * **Answer:** Define the primary App Service and its staging slot using azurerm_linux_web_app_slot. Configure settings so that environment variables and connection strings swap seamlessly between deployment slots.
 * **Example:**
```hcl
resource "azurerm_service_plan" "plan" {
  name                = "asp-web-prod"
  resource_group_name = "rg-web"
  location            = "East US"
  os_type             = "Linux"
  sku_name            = "P1v2"
}

# Production Primary App Slot
resource "azurerm_linux_web_app" "app" {
  name                = "app-web-prod-001"
  resource_group_name = "rg-web"
  location            = "East US"
  service_plan_id     = azurerm_service_plan.plan.id

  site_config {}
}

# Staging Slot for Blue/Green Releases
resource "azurerm_linux_web_app_slot" "staging" {
  name           = "staging"
  app_service_id = azurerm_linux_web_app.app.id

  site_config {}
}

```
### 4. How do you configure dynamic subnet delegation for Azure PaaS services (e.g., Azure Databricks or App Service VNet Integration)?
 * **Answer:** Inside azurerm_subnet, define a nested delegation block targeting the actions required by the specific Azure PaaS service.
 * **Example:** Delegating a subnet to Azure Web Apps for regional VNet Integration:
```hcl
resource "azurerm_subnet" "app_service_subnet" {
  name                 = "snet-app-integration"
  resource_group_name  = "rg-networking"
  virtual_network_name = "vnet-hub"
  address_prefixes     = ["10.0.10.0/24"]

  delegation {
    name = "app-service-delegation"

    service_delegation {
      name    = "Microsoft.Web/serverFarms"
      actions = ["Microsoft.Network/virtualNetworks/subnets/action"]
    }
  }
}

```
### 5. How do you securely handle sensitive outputs using Key Vault References without outputting secrets in CLI summaries?
 * **Answer:** Read raw credentials using data sources or dynamic generators, store them in Key Vault, and set sensitive = true on output blocks.
 * **Example:**
```hcl
resource "azurerm_key_vault_secret" "db_password" {
  name         = "db-admin-pass"
  value        = "SuperSecretPassword123!"
  key_vault_id = "/subscriptions/.../vaults/kv-prod"
}

output "db_password_secret_id" {
  value       = azurerm_key_vault_secret.db_password.id
  sensitive   = true # Prevents secret rendering during terraform apply
  description = "The secret URI to pass to key vault integration modules."
}

```
## Advanced Level Questions
### 6. How do you manage Azure Policy Assignments with compliance parameters using Terraform?
 * **Answer:** Use azurerm_subscription_policy_assignment or azurerm_resource_group_policy_assignment to target governance rules at defined scopes. Parameters are passed using valid JSON syntax.
 * **Example:** Enforcing explicit allowed locations across a subscription:
```hcl
resource "azurerm_subscription_policy_assignment" "audit_locations" {
  name                 = "enforce-allowed-locations"
  subscription_id      = "/subscriptions/00000000-0000-0000-0000-000000000000"
  policy_definition_id = "/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf7b01975c4c" # Built-in Allowed Locations Policy

  parameters = jsonencode({
    "listOfAllowedLocations" = {
      "value" = ["eastus", "eastus2"]
    }
  })
}

```
### 7. How do you map dynamic roles across multiple Azure Resource Groups using nested loops and flatten expressions?
 * **Answer:** When assigning multiple Azure RBAC roles across multiple target resource groups, flat list structures are needed for for_each. Use a nested for expression wrapped inside flatten().
 * **Example:**
```hcl
locals {
  user_roles = {
    "user-group-alpha" = {
      rgs   = ["rg-dev-app", "rg-dev-data"]
      roles = ["Contributor", "Log Analytics Contributor"]
    }
  }

  # Flatten nested maps into a list of objects
  assignments = flatten([
    for group_key, group_val in locals.user_roles : [
      for rg in group_val.rgs : [
        for role in group_val.roles : {
          key   = "${group_key}-${rg}-${role}"
          rg    = rg
          role  = role
        }
      ]
    ]
  ])
}

# Convert flattened list into a map for for_each iteration
resource "azurerm_role_assignment" "mapped_roles" {
  for_each             = { for item in locals.assignments : item.key => item }
  scope                = "/subscriptions/.../resourceGroups/${each.value.rg}"
  role_definition_name = each.value.role
  principal_id         = "00000000-0000-0000-0000-000000000000" # Target Group Object ID
}

```
### 8. How do you perform automated resource post-validation using Terraform custom postcondition blocks?
 * **Answer:** Declare a postcondition block inside a resource's lifecycle configuration to validate runtime state outputs returned by Azure APIs right after resource provisioning completes.
 * **Example:** Ensuring an Azure Storage Account requires HTTPS traffic after creation:
```hcl
resource "azurerm_storage_account" "st" {
  name                     = "stappdata001"
  resource_group_name      = "rg-data"
  location                 = "East US"
  account_tier             = "Standard"
  account_replication_type = "LRS"

  lifecycle {
    postcondition {
      condition     = self.enable_https_traffic_only == true
      error_message = "Security compliance failure: Storage Account must enforce HTTPS traffic only."
    }
  }
}

```

---

Here is a **tenth, expanded set** of unique Terraform and Azure interview questions and code examples. This set contains **12 detailed, scenario-based questions** spanning Basic, Intermediate, and Advanced topics.
## Basic Level Questions
### 1. How do you declare and manage multiple Azure Storage Containers inside a single Storage Account using for_each?
 * **Answer:** Rather than writing repetitive azurerm_storage_container blocks, pass a set(string) or map into for_each. This creates containers with predictable state addresses and allows easy management via simple variable lists.
 * **Example:**
```hcl
variable "container_names" {
  type    = set(string)
  default = ["data", "logs", "backups"]
}

resource "azurerm_storage_container" "containers" {
  for_each              = var.container_names
  name                  = each.key
  storage_account_name  = "stappdata001"
  container_access_type = "private"
}

```
### 2. How do you configure resource removal behavior using the terraform destroy prevent lifecycle rule?
 * **Answer:** You place prevent_destroy = true inside the lifecycle block of a critical resource. If anyone attempts a terraform destroy or an apply that forces recreation, Terraform rejects the plan before making any Azure API requests.
 * **Example:**
```hcl
resource "azurerm_mssql_database" "db" {
  name             = "sqldb-production"
  server_id        = "/subscriptions/.../servers/sql-prod"
  collation        = "SQL_Latin1_General_CP1_CI_AS"
  max_size_gb      = 100
  sku_name         = "S1"

  lifecycle {
    prevent_destroy = true
  }
}

```
### 3. What is the difference between var inputs and local definitions when configuring Azure Resource tags?
 * **Answer:**
   * **var (Input Variables):** Values supplied from outside the module (e.g., via CLI, .tfvars, or pipeline variables) to allow customization.
   * **local (Local Values):** Calculated expressions, merged maps, or internal constants defined inside the module to avoid code duplication.
 * **Example:** Merging external variables with calculated local values for tagging:
```hcl
variable "environment" {
  type    = string
  default = "prod"
}

locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
    CreatedDate = "2026-08-06"
  }
}

resource "azurerm_resource_group" "rg" {
  name     = "rg-app-${var.environment}"
  location = "East US"
  tags     = locals.common_tags
}

```
### 4. How do you suppress sensitive values in outputs so they aren't printed to stdout during terraform apply?
 * **Answer:** Mark the output definition with sensitive = true. Terraform continues saving the true value in the state file, but masks it in terminal outputs and pipeline logs.
 * **Example:**
```hcl
resource "azurerm_storage_account" "st" {
  name                     = "stappdata001"
  resource_group_name      = "rg-app"
  location                 = "East US"
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

output "storage_primary_access_key" {
  value       = azurerm_storage_account.st.primary_access_key
  sensitive   = true # Prevents displaying key in CLI/Pipeline execution logs
}

```
## Intermediate Level Questions
### 5. How do you configure Azure Virtual Network Peering bi-directionally between two VNets in Terraform?
 * **Answer:** Azure VNet Peering is non-transitive and unidirectional. To connect two VNets fully, you must construct **two** azurerm_virtual_network_peering resources—one from VNet A to VNet B, and another from VNet B to VNet A.
 * **Example:**
```hcl
# Peering 1: Hub to Spoke
resource "azurerm_virtual_network_peering" "hub_to_spoke" {
  name                      = "peer-hub-to-spoke"
  resource_group_name       = "rg-hub"
  virtual_network_name      = "vnet-hub"
  remote_virtual_network_id = "/subscriptions/.../virtualNetworks/vnet-spoke"
  allow_virtual_network_access = true
  allow_forwarded_traffic      = true
}

# Peering 2: Spoke to Hub
resource "azurerm_virtual_network_peering" "spoke_to_hub" {
  name                      = "peer-spoke-to-hub"
  resource_group_name       = "rg-spoke"
  virtual_network_name      = "vnet-spoke"
  remote_virtual_network_id = "/subscriptions/.../virtualNetworks/vnet-hub"
  allow_virtual_network_access = true
}

```
### 6. How do you attach a custom Route Table to an existing Azure Subnet using Terraform?
 * **Answer:** Provision the Route Table (azurerm_route_table), define individual routes (azurerm_route), and link it to the target subnet using azurerm_subnet_route_table_association.
 * **Example:**
```hcl
resource "azurerm_route_table" "rt" {
  name                = "rt-spoke-to-firewall"
  location            = "East US"
  resource_group_name = "rg-networking"

  route {
    name                   = "dg-to-azure-firewall"
    address_prefix         = "0.0.0.0/0"
    next_hop_type          = "VirtualAppliance"
    next_hop_in_ip_address = "10.0.0.4" # Azure Firewall Private IP
  }
}

resource "azurerm_subnet_route_table_association" "assoc" {
  subnet_id      = "/subscriptions/.../subnets/snet-app"
  route_table_id = azurerm_route_table.rt.id
}

```
### 7. How do you provision Azure Key Vault Access Policies dynamically for multiple Principal IDs?
 * **Answer:** Rather than declaring hardcoded policy blocks, use azurerm_key_vault_access_policy separately with for_each bound to a map of identities and permissions.
 * **Example:**
```hcl
variable "keyvault_access_map" {
  type = map(list(string))
  default = {
    "00000000-0000-0000-0000-000000000001" = ["Get", "List"]
    "00000000-0000-0000-0000-000000000002" = ["Get", "List", "Set", "Delete"]
  }
}

resource "azurerm_key_vault_access_policy" "policies" {
  for_each     = var.keyvault_access_map
  key_vault_id = "/subscriptions/.../vaults/kv-prod"
  tenant_id    = "11111111-1111-1111-1111-111111111111"
  object_id    = each.key

  secret_permissions = each.value
}

```
### 8. How do you configure custom Input Variable validations using regex patterns for Azure Naming compliance?
 * **Answer:** Add a validation block inside the variable declaration. Terraform tests the variable's value against standard functions or regular expressions before attempting to generate execution plans.
 * **Example:** Restricting Azure Storage Account names to lowercase letters and numbers (3 to 24 characters):
```hcl
variable "storage_account_name" {
  type        = string
  description = "Globally unique name for Azure Storage Account."

  validation {
    condition     = can(regex("^[a-z0-9]{3,24}$", var.storage_account_name))
    error_message = "Storage Account names must be between 3 and 24 characters, lowercase, and contain numbers or letters only."
  }
}

```
## Advanced Level Questions
### 9. How do you configure cross-region state replication or multi-region data storage using Terraform?
 * **Answer:** Pair an primary resource with secondary data endpoints using azurerm provider aliases or built-in geo-replication arguments (such as geo_match_definition or geo_replication).
 * **Example:** Deploying a Geo-Redundant (RA-GRS) Azure Storage Account with custom replication parameters:
```hcl
resource "azurerm_storage_account" "primary_storage" {
  name                     = "stappdataprod001"
  resource_group_name      = "rg-data-prod"
  location                 = "East US"
  account_tier             = "Standard"
  account_replication_type = "RAGRS" # Read-Access Geo-Redundant Storage

  blob_properties {
    versioning_enabled = true # Supports point-in-time recovery
  }
}

```
### 10. How do you implement custom validation on a Data Source using precondition blocks?
 * **Answer:** Use a precondition block inside a lifecycle block. If an existing Azure resource fetched via a data block doesn't meet requirements (e.g., missing expected tags or insufficient IP ranges), execution stops immediately.
 * **Example:** Validating that an existing Azure VNet has the correct tags before placing new workloads into it:
```hcl
data "azurerm_virtual_network" "existing_vnet" {
  name                = "vnet-shared-prod"
  resource_group_name = "rg-networking"

  lifecycle {
    precondition {
      condition     = lookup(self.tags, "ComplianceStatus", "") == "Approved"
      error_message = "Target Virtual Network is not marked as 'Approved' in Azure Tags."
    }
  }
}

```
### 11. How do you write declarative import blocks in HCL (Terraform 1.5+) to onboard unmanaged Azure resources safely?
 * **Answer:** Instead of running imperatively typed terraform import commands, declare import HCL blocks in code specifying the resource address and target Azure Resource Manager (ARM) ID.
 * **Example:**
```hcl
# 1. Target Resource HCL Definition
resource "azurerm_resource_group" "legacy_rg" {
  name     = "rg-legacy-app"
  location = "East US"
}

# 2. Declarative Import Block
import {
  to = azurerm_resource_group.legacy_rg
  id = "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-legacy-app"
}

```
### 12. How do you deploy an Azure Private Endpoint with Automated Private DNS Zone Registration in a modular pipeline?
 * **Answer:** Connect azurerm_private_endpoint directly to the target Azure PaaS resource ID (e.g., Key Vault or SQL Server), and include the private_dns_zone_group block to handle record registration automatically.
 * **Example:**
```hcl
resource "azurerm_private_endpoint" "kv_pe" {
  name                = "pe-kv-prod"
  location            = "East US"
  resource_group_name = "rg-security"
  subnet_id           = "/subscriptions/.../subnets/snet-private-endpoints"

  private_service_connection {
    name                           = "psc-kv-prod"
    private_connection_resource_id = "/subscriptions/.../vaults/kv-prod"
    is_manual_connection           = false
    subresource_names              = ["vault"]
  }

  private_dns_zone_group {
    name                 = "dns-group-kv"
    private_dns_zone_ids = ["/subscriptions/.../privateDnsZones/privatelink.vaultcore.azure.net"]
  }
}

```

---

Here is an **eleventh set** of 10 detailed, scenario-focused Terraform and Microsoft Azure interview questions with practical HCL code examples.
## Basic Level Questions
### 1. How do you declare and manage Provider Aliases for deploying resources across multiple Azure Regions in the same codebase?
 * **Answer:** * Define a primary provider without an alias and one or more secondary providers using the alias argument.
   * Resources declare which region they target by explicitly referencing the provider = azurerm.<alias_name> meta-argument.
 * **Example:**
```hcl
# Primary Azure Region (East US)
provider "azurerm" {
  features {}
}

# Secondary Azure Region (West US) for Disaster Recovery
provider "azurerm" {
  alias    = "dr"
  features {}
  location = "West US"
}

# Resource Group created in West US
resource "azurerm_resource_group" "rg_dr" {
  provider = azurerm.dr
  name     = "rg-app-dr-westus"
  location = "West US"
}

```
### 2. What is the ignore_changes lifecycle rule, and how do you prevent Terraform from overriding Azure Auto-Scaling modifications?
 * **Answer:** * Azure App Services, Virtual Machine Scale Sets, or AKS Clusters frequently modify attributes (e.g., instance counts or tags) dynamically via auto-scaling rules or policy engines.
   * The ignore_changes lifecycle meta-argument instructs Terraform to ignore drift on specified attributes during terraform plan and apply operations.
 * **Example:**
```hcl
resource "azurerm_linux_virtual_machine_scale_set" "vmss" {
  name                = "vmss-app-prod"
  resource_group_name = "rg-app"
  location            = "East US"
  sku                 = "Standard_DS1_v2"
  instances           = 2 # Initial baseline count

  # Prevents Terraform from overwriting changes made by Azure Auto-Scale settings
  lifecycle {
    ignore_changes = [
      instances,
      tags["LastAutoScaled"]
    ]
  }
}

```
### 3. How do you pass local script outputs or calculated files into Terraform using the local_file resource?
 * **Answer:** * You can generate output artifacts (like custom configuration files or environment setup scripts) dynamically and write them to disk using the local_file resource from the local provider.
 * **Example:** Writing rendered Azure VM boot diagnostics or deployment metadata to a local file:
```hcl
resource "local_file" "deployment_summary" {
  content  = <<EOT
  Deployment Summary for Azure Environment:
  VNet ID: ${azurerm_virtual_network.vnet.id}
  Subnet ID: ${azurerm_subnet.snet.id}
  Timestamp: ${timestamp()}
  EOT
  filename = "${path.module}/outputs/deployment-info.txt"
}

```
## Intermediate Level Questions
### 4. How do you create an Azure Key Vault Private Endpoint and integrate it with an Azure Private DNS Zone using Terraform?
 * **Answer:** You link an azurerm_private_endpoint to the Key Vault ID, attach it to a designated private subnet, and reference an azurerm_private_dns_zone_group pointing to privatelink.vaultcore.azure.net.
 * **Example:**
```hcl
resource "azurerm_private_dns_zone" "kv_dns" {
  name                = "privatelink.vaultcore.azure.net"
  resource_group_name = "rg-networking"
}

resource "azurerm_private_endpoint" "kv_pe" {
  name                = "pe-keyvault-prod"
  location            = "East US"
  resource_group_name = "rg-security"
  subnet_id           = "/subscriptions/.../subnets/snet-private-endpoints"

  private_service_connection {
    name                           = "psc-kv-prod"
    private_connection_resource_id = azurerm_key_vault.kv.id
    is_manual_connection           = false
    subresource_names              = ["vault"]
  }

  private_dns_zone_group {
    name                 = "dns-group-kv"
    private_dns_zone_ids = [azurerm_private_dns_zone.kv_dns.id]
  }
}

```
### 5. How do you enforce custom Tag standards on Azure resources using HCL variable validations?
 * **Answer:** Define a map variable for resource tags and attach a validation block using HCL functions like contains() or keys() to verify required keys exist before plan execution.
 * **Example:**
```hcl
variable "mandatory_tags" {
  type = map(string)

  validation {
    condition     = contains(keys(var.mandatory_tags), "Environment") && contains(keys(var.mandatory_tags), "CostCenter")
    error_message = "All Azure resources must contain both 'Environment' and 'CostCenter' tag keys."
  }
}

```
### 6. How do you configure Azure Event Grid Subscriptions using Terraform to trigger Azure Functions?
 * **Answer:** Use azurerm_eventgrid_event_subscription to route system events (such as Blob Storage file creations) directly to an Azure Function or Event Hub endpoint.
 * **Example:**
```hcl
resource "azurerm_eventgrid_event_subscription" "storage_events" {
  name                  = "evgs-blob-created"
  scope                 = azurerm_storage_account.st.id
  included_event_types  = ["Microsoft.Storage.BlobCreated"]

  azure_function_endpoint {
    function_id = "${azurerm_linux_function_app.func.id}/functions/ProcessBlob"
  }
}

```
### 7. How do you implement Azure User-Assigned Identities with AKS Workload Identity using Terraform?
 * **Answer:** Enable oidc_issuer_enabled and workload_identity_enabled on the AKS cluster. Create an azurerm_user_assigned_identity and establish a federated identity credential (azurerm_federated_identity_credential) linked to the Kubernetes service account namespace.
 * **Example:**
```hcl
# 1. Enable OIDC & Workload Identity on AKS
resource "azurerm_kubernetes_cluster" "aks" {
  name                = "aks-prod"
  location            = "East US"
  resource_group_name = "rg-aks"
  dns_prefix          = "aksprod"

  oidc_issuer_enabled       = true
  workload_identity_enabled = true

  default_node_pool {
    name       = "system"
    node_count = 2
    vm_size    = "Standard_DS2_v2"
  }
  identity { type = "SystemAssigned" }
}

# 2. Establish Federated Identity Credential
resource "azurerm_federated_identity_credential" "aks_federated" {
  name                = "fed-aks-app"
  resource_group_name = "rg-aks"
  audience            = ["api://AzureADTokenExchange"]
  issuer              = azurerm_kubernetes_cluster.aks.oidc_issuer_url
  subject             = "system:serviceaccount:default:app-service-account"
  parent_id           = azurerm_user_assigned_identity.app_identity.id
}

```
## Advanced Level Questions
### 8. How do you implement automated multi-region deployment with fallback routing using Azure Front Door in Terraform?
 * **Answer:** Declare an azurerm_cdn_frontdoor_profile along with an origin group containing primary and secondary region endpoints. Configure health probe settings so Front Door automatically routes traffic to the secondary region if the primary fails.
 * **Example:**
```hcl
# Azure Front Door Origin Group with Failover Health Probe
resource "azurerm_cdn_frontdoor_origin_group" "og" {
  name                     = "og-global-web"
  cdn_frontdoor_profile_id = azurerm_cdn_frontdoor_profile.profile.id

  health_probe {
    protocol            = "Https"
    request_type        = "HEAD"
    probe_method        = "GET"
    path                = "/health"
    interval_in_seconds = 15
  }

  load_balancing {
    additional_latency_in_milliseconds = 50
    sample_size                        = 4
    successful_samples_required        = 2
  }
}

# Primary Region Origin (East US)
resource "azurerm_cdn_frontdoor_origin" "primary" {
  name                          = "origin-primary-eastus"
  cdn_frontdoor_origin_group_id = azurerm_cdn_frontdoor_origin_group.og.id
  enabled                       = true
  host_name                     = "app-primary-eastus.azurewebsites.net"
  priority                      = 1 # Primary target
  weight                        = 1000
}

# Secondary Region Origin (West US - Failover)
resource "azurerm_cdn_frontdoor_origin" "secondary" {
  name                          = "origin-secondary-westus"
  cdn_frontdoor_origin_group_id = azurerm_cdn_frontdoor_origin_group.og.id
  enabled                       = true
  host_name                     = "app-secondary-westus.azurewebsites.net"
  priority                      = 2 # Secondary fallback
  weight                        = 1000
}

```
### 9. How do you refactor monolithic Terraform code into modules without recreating existing production resources?
 * **Answer:** * Write the child module code containing the target resources.
   * Define moved blocks in the root configuration mapping the old standalone resource addresses to their new modular addresses.
   * Execute terraform plan to confirm that Terraform registers a structural state move rather than a destroy/recreate operation.
 * **Example:**
```hcl
# Standalone definition previously used:
# resource "azurerm_storage_account" "legacy_st" { ... }

# New Child Module Call:
module "storage_module" {
  source               = "./modules/storage"
  storage_account_name = "stappdataprod001"
  resource_group_name  = "rg-data"
}

# Refactoring Directive: Moves state non-destructively
moved {
  from = azurerm_storage_account.legacy_st
  to   = module.storage_module.azurerm_storage_account.this
}

```
### 10. How do you execute conditional data fetching using data sources with dynamic parameters in Azure?
 * **Answer:** Use ternary logic or count inside a data source block to conditionally query Azure resources only when certain conditions are met.
 * **Example:** Fetching an existing Virtual Network only if an external network ID is provided, otherwise skipping the data lookup:
```hcl
variable "existing_vnet_name" {
  type    = string
  default = ""
}

data "azurerm_virtual_network" "vnet_lookup" {
  count               = var.existing_vnet_name != "" ? 1 : 0
  name                = var.existing_vnet_name
  resource_group_name = "rg-shared-networking"
}

locals {
  # Resolves ID from Data Source if provided, otherwise defaults to local resource
  target_vnet_id = var.existing_vnet_name != "" ? data.azurerm_virtual_network.vnet_lookup[0].id : azurerm_virtual_network.local_vnet[0].id
}

```

---

Here is a **twelfth set** of 10 scenario-based Terraform and Azure interview questions with HCL code examples.
## Basic Level Questions
### 1. What is the difference between terraform destroy and setting count = 0 on an Azure resource block?
 * **Answer:**
   * **terraform destroy:** Destroys **all** infrastructure managed by the state file.
   * **count = 0:** Removes only the **specific resource block** assigned count = 0 during the next terraform apply, while leaving all other state-managed Azure resources completely untouched.
 * **Example:**
```hcl
variable "deploy_bastion" {
  type    = bool
  default = false # Set to false to remove Bastion without destroying the VNet
}

resource "azurerm_bastion_host" "bastion" {
  count               = var.deploy_bastion ? 1 : 0
  name                = "bas-shared-prod"
  location            = "East US"
  resource_group_name = "rg-networking"

  ip_configuration {
    name                 = "bastion-ip-config"
    subnet_id            = "/subscriptions/.../subnets/AzureBastionSubnet"
    public_ip_address_id = "/subscriptions/.../publicIPAddresses/pip-bastion"
  }
}

```
### 2. How do you convert complex object outputs into localized JSON files using jsonencode() in HCL?
 * **Answer:** When external deployment scripts require output parameters (such as database endpoints or subnet IDs), you can format local expressions or outputs into structured JSON strings using jsonencode().
 * **Example:**
```hcl
locals {
  app_config = {
    environment = "production"
    database = {
      server_fqdn = "sql-app-prod.database.windows.net"
      db_name     = "sqldb-orders"
    }
    features_enabled = ["logging", "auto-scaling", "cdn"]
  }
}

resource "local_file" "app_settings" {
  content  = jsonencode(locals.app_config)
  filename = "${path.module}/appsettings.json"
}

```
### 3. How do you handle string case formatting and transformations in Azure naming tags?
 * **Answer:** Azure resource properties (like Storage Accounts) enforce strict lowercase alphanumeric constraints. You can use native HCL string functions like lower(), replace(), and substr() inside local variables to sanitize inputs automatically.
 * **Example:**
```hcl
variable "raw_project_name" {
  type    = string
  default = "My_Project-App 01"
}

locals {
  # Converts "My_Project-App 01" -> "myprojectapp01"
  clean_storage_name = lower(replace(replace(var.raw_project_name, "_", ""), "-", ""))
}

resource "azurerm_storage_account" "st" {
  name                     = "st${locals.clean_storage_name}"
  resource_group_name      = "rg-storage"
  location                 = "East US"
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

```
## Intermediate Level Questions
### 4. How do you attach a Network Security Group (NSG) to an Azure Subnet using the modern dedicated association resource?
 * **Answer:** In current versions of the azurerm provider, embedding security rules directly inside the azurerm_subnet resource is discouraged because it can cause state conflicts. Instead, create an independent azurerm_network_security_group and link it to the subnet using azurerm_subnet_network_security_group_association.
 * **Example:**
```hcl
resource "azurerm_network_security_group" "nsg" {
  name                = "nsg-app-prod"
  location            = "East US"
  resource_group_name = "rg-networking"

  security_rule {
    name                       = "AllowHTTPS"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "443"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

resource "azurerm_subnet_network_security_group_association" "nsg_assoc" {
  subnet_id                 = "/subscriptions/.../subnets/snet-app"
  network_security_group_id = azurerm_network_security_group.nsg.id
}

```
### 5. How do you configure Azure Key Vault Auto-Rotation policies for encryption keys using Terraform?
 * **Answer:** Define the key using azurerm_key_vault_key and add a nested rotation_policy block specifying the time intervals for automatic key generation and expiry alerts.
 * **Example:**
```hcl
resource "azurerm_key_vault_key" "auto_key" {
  name         = "cmk-storage-key"
  key_vault_id = "/subscriptions/.../vaults/kv-security-prod"
  key_type     = "RSA"
  key_size     = 2048

  key_opts = [
    "decrypt", "encrypt", "sign", "unwrapKey", "verify", "wrapKey"
  ]

  rotation_policy {
    automatic {
      time_before_expiry = "P30D" # Rotate 30 days before key expiry
    }

    expire_after         = "P90D" # Set total key lifetime to 90 days
    notify_before_expiry = "P15D" # Trigger Azure Monitor alert 15 days before expiry
  }
}

```
### 6. How do you enforce private network access on an Azure Container Registry (ACR) using Terraform?
 * **Answer:** Disable public network access on the azurerm_container_registry resource (public_network_access_enabled = false) and attach a Private Endpoint (azurerm_private_endpoint) targeting the subresource registry.
 * **Example:**
```hcl
resource "azurerm_container_registry" "acr" {
  name                          = "acrsharedappprod001"
  resource_group_name           = "rg-container-services"
  location                      = "East US"
  sku                           = "Premium" # Required for Private Endpoints
  public_network_access_enabled = false    # Disables public internet access
}

resource "azurerm_private_endpoint" "acr_pe" {
  name                = "pe-acr-prod"
  location            = "East US"
  resource_group_name = "rg-container-services"
  subnet_id           = "/subscriptions/.../subnets/snet-private-endpoints"

  private_service_connection {
    name                           = "psc-acr-prod"
    private_connection_resource_id = azurerm_container_registry.acr.id
    is_manual_connection           = false
    subresource_names              = ["registry"]
  }
}

```
### 7. How do you execute time_sleep delays between interdependent Azure resource creations?
 * **Answer:** * Certain Azure APIs (e.g., Entra ID RBAC role propogation or Managed Identity assignments) take up to 60–90 seconds to fully propagate across regions.
   * You can insert an explicit delay using the time_sleep resource from the hashicorp/time provider before executing downstream dependent resources.
 * **Example:**
```hcl
resource "azurerm_user_assigned_identity" "identity" {
  name                = "id-aks-keda"
  location            = "East US"
  resource_group_name = "rg-aks"
}

# Waits 60 seconds after identity creation to allow Entra ID propagation
resource "time_sleep" "wait_60_seconds" {
  depends_on      = [azurerm_user_assigned_identity.identity]
  create_duration = "60s"
}

# Executes assignment only after the sleep duration completes
resource "azurerm_role_assignment" "role" {
  depends_on           = [time_sleep.wait_60_seconds]
  scope                = "/subscriptions/.../resourceGroups/rg-data"
  role_definition_name = "Storage Blob Data Contributor"
  principal_id         = azurerm_user_assigned_identity.identity.principal_id
}

```
## Advanced Level Questions
### 8. How do you provision an Azure Application Gateway with Path-Based Routing and SSL Termination using Terraform?
 * **Answer:** Configure azurerm_application_gateway by defining frontend_port, backend_address_pool, http_listener, and request_routing_rule blocks with PathBasedRouting type.
 * **Example:**
```hcl
resource "azurerm_application_gateway" "appgw" {
  name                = "agw-web-prod"
  resource_group_name = "rg-networking"
  location            = "East US"

  sku {
    name     = "Standard_v2"
    tier     = "Standard_v2"
    capacity = 2
  }

  gateway_ip_configuration {
    name      = "ip-config"
    subnet_id = "/subscriptions/.../subnets/AppGatewaySubnet"
  }

  frontend_port {
    name = "port_80"
    port = 80
  }

  frontend_ip_configuration {
    name                 = "public-ip-config"
    public_ip_address_id = "/subscriptions/.../publicIPAddresses/pip-agw"
  }

  backend_address_pool {
    name = "default-backend-pool"
  }

  backend_http_settings {
    name                  = "http-settings"
    cookie_based_affinity = "Disabled"
    port                  = 80
    protocol              = "Http"
    request_timeout       = 20
  }

  http_listener {
    name                           = "http-listener"
    frontend_ip_configuration_name = "public-ip-config"
    frontend_port_name             = "port_80"
    protocol                       = "Http"
  }

  request_routing_rule {
    name                       = "routing-rule"
    rule_type                  = "Basic"
    http_listener_name         = "http-listener"
    backend_address_pool_name  = "default-backend-pool"
    backend_http_settings_name = "http-settings"
    priority                   = 100
  }
}

```
### 9. How do you implement dynamic diagnostic settings across multiple Azure resources using azurerm_monitor_diagnostic_setting?
 * **Answer:** Iterate over a map or set of resource IDs using for_each on azurerm_monitor_diagnostic_setting to stream metric and log categories into a central Log Analytics Workspace.
 * **Example:**
```hcl
variable "target_resource_ids" {
  type = map(string)
  default = {
    "keyvault" = "/subscriptions/.../vaults/kv-prod"
    "storage"  = "/subscriptions/.../storageAccounts/stprod001"
  }
}

resource "azurerm_monitor_diagnostic_setting" "diag" {
  for_each                   = var.target_resource_ids
  name                       = "diag-${each.key}"
  target_resource_id         = each.value
  log_analytics_workspace_id = "/subscriptions/.../workspaces/law-central-logs"

  enabled_log {
    category_group = "allLogs"
  }

  metric {
    category = "AllMetrics"
    enabled  = true
  }
}

```
### 10. How do you automate Terraform Plan output review in GitHub Actions using Workflow Artifacts and Comment Bot automation?
 * **Answer:**
   1. In the Pull Request workflow, execute terraform plan -out=tfplan.
   2. Convert the binary plan to JSON: terraform show -json tfplan > plan.json.
   3. Run script checks or policy evaluations (e.g., checkov or opa) against plan.json.
   4. Post a sanitized summary of proposed changes directly as a comment on the Pull Request using GitHub Script actions.
 * **Example GitHub Actions Step:**
```yaml
- name: Terraform Plan Output
  run: |
    terraform plan -no-color -out=tfplan
    terraform show -no-color tfplan > plan_summary.txt

- name: Comment Plan on PR
  uses: actions/github-script@v6
  with:
    script: |
      const fs = require('fs');
      const plan = fs.readFileSync('plan_summary.txt', 'utf8');
      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: `### Terraform Execution Plan Summary:\n\`\`\`hcl\n${plan.substring(0, 6000)}\n\`\`\``
      })

```


