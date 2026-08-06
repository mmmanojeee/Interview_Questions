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

---


Here is a **thirteenth set** of 10 scenario-based Terraform and Azure interview questions with practical HCL code examples.
## Basic Level Questions
### 1. What is the difference between path.module, path.root, and path.cwd in Terraform HCL?
 * **Answer:**
   * **path.module:** Returns the filesystem path to the module directory where the expression is evaluated. Essential inside reusable child modules to reference local template files or scripts relative to that module.
   * **path.root:** Returns the filesystem path to the root module directory where execution started.
   * **path.cwd:** Returns the current working directory from which the terraform command was run.
 * **Example:** Loading a local initialization script located inside a child module:
```hcl
# Inside modules/app_service/main.tf
resource "azurerm_linux_web_app" "app" {
  name                = "app-orders-prod"
  resource_group_name = "rg-orders"
  location            = "East US"
  service_plan_id     = var.service_plan_id

  site_config {
    # Path relative to child module location, regardless of where terraform apply is executed
    app_command_line = file("${path.module}/scripts/startup.sh")
  }
}

```
### 2. How do you configure an Azure Storage Account Blob Lifecycle Management policy using Terraform?
 * **Answer:** Use azurerm_storage_management_policy to define automated rules that transition unaccessed blobs to cooler storage tiers (Cool/Archive) or delete them after a defined number of days.
 * **Example:** Moving blobs to Cool storage after 30 days and deleting them after 365 days:
```hcl
resource "azurerm_storage_management_policy" "retention" {
  storage_account_id = azurerm_storage_account.st.id

  rule {
    name    = "archive-and-delete-old-blobs"
    enabled = true

    filter {
      prefix_match = ["container-logs/"]
      blob_types   = ["blockBlob"]
    }

    actions {
      base_blob {
        tier_to_cool_after_days_since_modification_greater_than    = 30
        tier_to_archive_after_days_since_modification_greater_than = 90
        delete_after_days_since_modification_greater_than          = 365
      }
    }
  }
}

```
### 3. How do you extract and export Terraform outputs in JSON format (terraform output -json) for automated pipeline ingestion?
 * **Answer:** Running terraform output -json outputs all state outputs as a single, unformatted JSON object. This allows downstream script engines (like PowerShell, Python, or Azure CLI steps) to parse infrastructural data programmatically without needing direct state access.
 * **Example:**
```bash
# Export state outputs to a JSON file
terraform output -json > outputs.json

# Parse specific value using jq in CI/CD pipeline
STORAGE_KEY=$(jq -r '.storage_primary_access_key.value' outputs.json)

```
## Intermediate Level Questions
### 4. How do you manage Azure SQL Database Elastic Pools dynamically using Terraform?
 * **Answer:** Create an azurerm_mssql_elasticpool to define shared resource limits (e.g., eDTUs or vCores). Next, link individual azurerm_mssql_database resources to the pool using the elastic_pool_id argument instead of assigning isolated compute tiers.
 * **Example:**
```hcl
# Create shared Elastic Pool
resource "azurerm_mssql_elasticpool" "pool" {
  name                = "sqlelasticpool-prod"
  resource_group_name = "rg-database"
  location            = "East US"
  server_name         = azurerm_mssql_server.sql.name
  license_type        = "LicenseIncluded"

  sku {
    name     = "GP_Gen5"
    tier     = "GeneralPurpose"
    family   = "Gen5"
    capacity = 4 # Total vCores for the pool
  }

  per_database_settings {
    min_capacity = 0.25
    max_capacity = 2.0
  }
}

# Attach database to Elastic Pool
resource "azurerm_mssql_database" "db" {
  name            = "sqldb-orders"
  server_id       = azurerm_mssql_server.sql.id
  elastic_pool_id = azurerm_mssql_elasticpool.pool.id
}

```
### 5. How do you configure an Azure Standard Load Balancer with Outbound SNAT Rules using Terraform?
 * **Answer:** Provision an azurerm_lb with a Dedicated Outbound Frontend IP Configuration and associate an azurerm_lb_outbound_rule to grant VMs inside internal backend pools secure outbound internet connectivity without exposing inbound ports.
 * **Example:**
```hcl
resource "azurerm_lb" "lb" {
  name                = "lb-outbound-prod"
  location            = "East US"
  resource_group_name = "rg-networking"
  sku                 = "Standard" # Standard SKU required for outbound rules

  frontend_ip_configuration {
    name                 = "outbound-public-ip"
    public_ip_address_id = azurerm_public_ip.pip.id
  }
}

resource "azurerm_lb_backend_address_pool" "bap" {
  name            = "backend-pool-vms"
  loadbalancer_id = azurerm_lb.lb.id
}

resource "azurerm_lb_outbound_rule" "snat_rule" {
  name                    = "outbound-snat-rule"
  loadbalancer_id         = azurerm_lb.lb.id
  protocol                = "All"
  backend_address_pool_id = azurerm_lb_backend_address_pool.bap.id

  frontend_ip_configuration {
    name = "outbound-public-ip"
  }
}

```
### 6. How do you implement Azure Virtual WAN (vWAN) with Virtual Hubs and Spoke VNet Connections in Terraform?
 * **Answer:** Create an azurerm_virtual_wan, establish an azurerm_virtual_hub inside a specific region, and peer regional spoke VNets to the hub using azurerm_virtual_hub_connection.
 * **Example:**
```hcl
# 1. Declare Global Virtual WAN
resource "azurerm_virtual_wan" "vwan" {
  name                = "vwan-global-enterprise"
  resource_group_name = "rg-networking-global"
  location            = "East US"
}

# 2. Deploy Regional Virtual Hub
resource "azurerm_virtual_hub" "hub_eastus" {
  name                = "vhub-eastus"
  resource_group_name = "rg-networking-global"
  location            = "East US"
  virtual_wan_id      = azurerm_virtual_wan.vwan.id
  address_prefix      = "10.100.0.0/23"
}

# 3. Connect Spoke VNet to Virtual Hub
resource "azurerm_virtual_hub_connection" "spoke_connection" {
  name                      = "vhub-conn-spoke-app"
  virtual_hub_id            = azurerm_virtual_hub.hub_eastus.id
  remote_virtual_network_id = "/subscriptions/.../virtualNetworks/vnet-spoke-app"
}

```
### 7. How do you provision Azure Key Vault Access Policies for Service Principals without exposing secrets in code?
 * **Answer:** Fetch the Service Principal's Object ID dynamically using the azuread_service_principal data source (from the azuread provider) and pass that object ID into azurerm_key_vault_access_policy.
 * **Example:**
```hcl
# Lookup Service Principal by Application ID
data "azuread_service_principal" "sp" {
  application_id = "00000000-0000-0000-0000-000000000000"
}

resource "azurerm_key_vault_access_policy" "sp_policy" {
  key_vault_id = "/subscriptions/.../vaults/kv-sec-prod"
  tenant_id    = "11111111-1111-1111-1111-111111111111"
  object_id    = data.azuread_service_principal.sp.object_id

  secret_permissions = ["Get", "List"]
}

```
## Advanced Level Questions
### 8. How do you integrate Azure API Management (APIM) with Key Vault to bind custom SSL domain certificates in Terraform?
 * **Answer:** Grant the APIM System-Assigned Identity read access on the Key Vault, fetch the target certificate secret using azurerm_key_vault_certificate, and attach it to the hostname_configuration block of azurerm_api_management.
 * **Example:**
```hcl
resource "azurerm_api_management" "apim" {
  name                = "apim-enterprise-prod"
  location            = "East US"
  resource_group_name = "rg-api"
  publisher_name      = "Enterprise IT"
  publisher_email     = "admin@enterprise.com"
  sku_name            = "Developer_1"

  identity {
    type = "SystemAssigned"
  }

  hostname_configuration {
    proxy {
      default_ssl_binding = true
      host_name           = "api.enterprise.com"
      key_vault_id        = "https://kv-prod.vault.azure.net/secrets/custom-api-cert"
    }
  }
}

```
### 9. How do you migrate Terraform state between two different Azure Storage Accounts seamlessly without losing state tracking?
 * **Answer:**
   1. Update the backend "azurerm" block in main.tf to target the **new** Storage Account or container name.
   2. Run terraform init -migrate-state.
   3. Terraform will detect the backend configuration change, verify access to both backends, and automatically copy the current state file to the new location while maintaining lock safety.
 * **Example Command:**
```bash
# Step 1: Update main.tf with new storage account backend settings
# Step 2: Initialize migration
terraform init -migrate-state

```
### 10. How do you deploy preview or unsupported Azure features using the Microsoft azapi_resource provider alongside azurerm?
 * **Answer:** Use the azapi_resource block from the azure/azapi provider. Specify the type (ARM API resource type and version) and pass resource properties as a structured HCL map or JSON payload using body.
 * **Example:** Provisioning a preview Azure feature before it exists in azurerm:
```hcl
terraform {
  required_providers {
    azapi = {
      source  = "azure/azapi"
      version = "~> 1.0"
    }
  }
}

provider "azapi" {}

# Deploying an ARM resource using raw REST API schema
resource "azapi_resource" "preview_feature" {
  type      = "Microsoft.ContainerInstance/containerGroups@2023-05-01"
  name      = "aci-preview-app"
  location  = "East US"
  parent_id = "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-app"

  body = {
    properties = {
      containers = [
        {
          name = "web"
          properties = {
            image = "mcr.microsoft.com/azuredocs/aci-helloworld"
            resources = {
              requests = {
                cpu    = 1.0
                memoryInGB = 1.5
              }
            }
            ports = [{ port = 80 }]
          }
        }
      ]
      osType        = "Linux"
      ipAddress     = { type = "Public", ports = [{ port = 80, protocol = "TCP" }] }
    }
  }
}

```

---

Here is a **fourteenth set** of 10 scenario-focused Terraform and Microsoft Azure interview questions complete with practical HCL code examples.
## Basic Level Questions
### 1. How do you execute post-provisioning scripts inside an Azure Virtual Machine without using local-exec provisioners?
 * **Answer:** Instead of using Terraform provisioners (which bypass state tracking), use the azurerm_virtual_machine_extension resource to invoke the **Custom Script Extension** natively through Azure Resource Manager.
 * **Example:**
```hcl
resource "azurerm_virtual_machine_extension" "custom_script" {
  name                 = "vm-bootstrap-script"
  virtual_machine_id   = azurerm_linux_virtual_machine.vm.id
  publisher            = "Microsoft.Azure.Extensions"
  type                 = "CustomScript"
  type_handler_version = "2.1"

  settings = <<SETTINGS
    {
        "commandToExecute": "echo 'Bootstrap complete' > /tmp/init.log"
    }
SETTINGS
}

```
### 2. How do you enforce Immutability (WORM policy) on an Azure Storage Container using Terraform?
 * **Answer:** Configure azurerm_storage_container_immutability_policy to block modification or deletion of blobs within a container for a specified retention period.
 * **Example:**
```hcl
resource "azurerm_storage_container" "audit_logs" {
  name                  = "audit-compliance-logs"
  storage_account_name  = azurerm_storage_account.st.name
  container_access_type = "private"
}

resource "azurerm_storage_container_immutability_policy" "worm_policy" {
  storage_container_resource_manager_id = azurerm_storage_container.audit_logs.resource_manager_id
  immutability_period_in_days            = 365
  protected_append_blobs_automated_policy_enabled = true
}

```
### 3. How do you look up regional location display names dynamically using Azure data sources?
 * **Answer:** Use the azurerm_location data source to fetch metadata for a region (such as display names or pairing regions) without hardcoding regional strings.
 * **Example:**
```hcl
data "azurerm_location" "eastus" {
  location = "eastus"
}

output "location_display_name" {
  value = data.azurerm_location.eastus.display_name # Outputs "East US"
}

output "paired_location" {
  value = data.azurerm_location.eastus.paired_location # Outputs "westus"
}

```
## Intermediate Level Questions
### 4. How do you configure Azure Application Insights linked to a central Log Analytics Workspace using Terraform?
 * **Answer:** Declare an azurerm_log_analytics_workspace and link it to azurerm_application_insights using the workspace_id property.
 * **Example:**
```hcl
resource "azurerm_log_analytics_workspace" "law" {
  name                = "law-central-prod"
  location            = "East US"
  resource_group_name = "rg-ops"
  sku                 = "PerGB2018"
}

resource "azurerm_application_insights" "app_insights" {
  name                = "appi-orderservice-prod"
  location            = "East US"
  resource_group_name = "rg-ops"
  workspace_id        = azurerm_log_analytics_workspace.law.id
  application_type    = "web"
}

```
### 5. How do you attach an Azure NAT Gateway to multiple subnets for outbound IP egress control using Terraform?
 * **Answer:** Create an azurerm_nat_gateway, associate a Static Public IP (azurerm_public_ip), and link target subnets using azurerm_subnet_nat_gateway_association.
 * **Example:**
```hcl
resource "azurerm_nat_gateway" "nat" {
  name                = "nat-gw-prod"
  location            = "East US"
  resource_group_name = "rg-networking"
  sku_name            = "Standard"
}

resource "azurerm_subnet_nat_gateway_association" "assoc_app" {
  subnet_id      = "/subscriptions/.../subnets/snet-app"
  nat_gateway_id = azurerm_nat_gateway.nat.id
}

```
### 6. How do you provision an Azure Cosmos DB SQL Database with Autoscale settings using Terraform?
 * **Answer:** Define an azurerm_cosmosdb_account and create an azurerm_cosmosdb_sql_database configured with an autoscale_settings block.
 * **Example:**
```hcl
resource "azurerm_cosmosdb_account" "cosmos" {
  name                = "cosmos-orders-prod"
  location            = "East US"
  resource_group_name = "rg-data"
  offer_type          = "Standard"
  kind                = "GlobalDocumentDB"

  consistency_policy {
    consistency_level = "Session"
  }

  geo_location {
    location          = "East US"
    failover_priority = 0
  }
}

resource "azurerm_cosmosdb_sql_database" "db" {
  name                = "sqldb-orders"
  resource_group_name = azurerm_cosmosdb_account.cosmos.resource_group_name
  account_name        = azurerm_cosmosdb_account.cosmos.name

  autoscale_settings {
    max_throughput = 4000 # Scales dynamically between 400 RU/s and 4000 RU/s
  }
}

```
### 7. How do you link a single Azure Private DNS Zone across multiple Virtual Networks using for_each?
 * **Answer:** Maintain a map or list of VNet IDs and iterate using for_each inside azurerm_private_dns_zone_virtual_network_link.
 * **Example:**
```hcl
variable "vnet_ids" {
  type = map(string)
  default = {
    "hub"   = "/subscriptions/.../virtualNetworks/vnet-hub"
    "spoke" = "/subscriptions/.../virtualNetworks/vnet-spoke-app"
  }
}

resource "azurerm_private_dns_zone" "dns" {
  name                = "privatelink.database.windows.net"
  resource_group_name = "rg-networking"
}

resource "azurerm_private_dns_zone_virtual_network_link" "links" {
  for_each              = var.vnet_ids
  name                  = "link-${each.key}"
  resource_group_name   = azurerm_private_dns_zone.dns.resource_group_name
  private_dns_zone_name = azurerm_private_dns_zone.dns.name
  virtual_network_id    = each.value
}

```
## Advanced Level Questions
### 8. How do you create Custom Azure Policy Definitions and trigger Policy Remediation Tasks via Terraform?
 * **Answer:** Define custom governance rules using azurerm_policy_definition, assign them via azurerm_resource_group_policy_assignment, and trigger automatic remediation of existing non-compliant resources using azurerm_resource_group_policy_remediation.
 * **Example:**
```hcl
# 1. Custom Policy Definition
resource "azurerm_policy_definition" "enforce_https" {
  name         = "enforce-storage-https"
  policy_type  = "Custom"
  mode         = "Indexed"
  display_name = "Enforce HTTPS for Storage Accounts"

  policy_rule = jsonencode({
    if = {
      field  = "type"
      equals = "Microsoft.Storage/storageAccounts"
    }
    then = {
      effect = "Modify"
      details = {
        roleDefinitionIds = ["/providers/Microsoft.Authorization/roleDefinitions/17d101d3-bc11-4c22-b94e-af69f357809b"]
        operations = [
          {
            operation = "addOrReplace"
            field     = "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly"
            value     = true
          }
        ]
      }
    }
  })
}

# 2. Policy Remediation Task
resource "azurerm_resource_group_policy_remediation" "remediate_storage" {
  name                 = "remediate-storage-https"
  resource_group_id    = "/subscriptions/.../resourceGroups/rg-data"
  policy_assignment_id = azurerm_resource_group_policy_assignment.assign.id
}

```
### 9. How do you implement Ephemeral Variables or Write-Only Values to prevent secrets from being stored in Terraform State?
 * **Answer:** Modern Terraform features support write-only inputs or ephemeral resources that process credentials during execution without recording raw secret strings inside the JSON state file.
 * **Example:** Defining ephemeral parameters in modern HCL:
```hcl
# Input variable marked as ephemeral (Terraform 1.10+)
variable "database_password" {
  type      = string
  ephemeral = true # Prevents value from being persisted to state file
}

resource "azurerm_key_vault_secret" "db_secret" {
  name         = "sql-admin-pass"
  value        = var.database_password
  key_vault_id = "/subscriptions/.../vaults/kv-prod"
}

```
### 10. How do you configure Disaster Recovery VM Replication using Azure Site Recovery (ASR) in Terraform?
 * **Answer:** Configure azurerm_site_recovery_fabric in primary and secondary regions, set up an azurerm_site_recovery_protection_container, and initiate disk replication via azurerm_site_recovery_replicated_vm.
 * **Example:**
```hcl
resource "azurerm_site_recovery_replicated_vm" "vm_replication" {
  name                                      = "asr-vm-replication"
  resource_group_name                       = "rg-recovery-vault"
  recovery_vault_name                       = azurerm_recovery_services_vault.vault.name
  source_recovery_fabric_name               = "primary-fabric-eastus"
  source_vm_id                              = azurerm_linux_virtual_machine.primary_vm.id
  target_recovery_fabric_id                 = azurerm_site_recovery_fabric.secondary.id
  target_resource_group_id                  = "/subscriptions/.../resourceGroups/rg-dr-westus"
  target_zone                               = "1"

  managed_disk {
    disk_id                    = azurerm_linux_virtual_machine.primary_vm.os_disk[0].managed_disk_id
    staging_storage_account_id = azurerm_storage_account.asr_cache.id
    target_resource_group_id   = "/subscriptions/.../resourceGroups/rg-dr-westus"
    target_disk_type           = "Premium_LRS"
    target_replica_disk_type   = "Premium_LRS"
  }
}

```

---

Here is a **fifteenth set** of 10 scenario-focused Terraform and Microsoft Azure interview questions with practical HCL code examples.
## Basic Level Questions
### 1. What is terraform graph, and how can you use it to visualize Azure resource dependencies?
 * **Answer:** * terraform graph: Generates a visual representation of the dependency tree defined in your Terraform configuration or state file in Graphviz DOT format.
   * **Use Case:** Helps teams analyze complex resource creation chains (e.g., verifying that Network Security Groups and Subnets are constructed before Virtual Machines) or identify unintended circular dependencies.
 * **Example Command:**
```bash
# Generate a dependency graph and convert it to an image file using Graphviz
terraform graph | dot -Tpng > azure_infrastructure_graph.png

```
### 2. How do override.tf and *_override.tf files behave in a Terraform configuration?
 * **Answer:**
   * Files named override.tf or ending in _override.tf are loaded **last** by Terraform.
   * Rather than replacing standard .tf files, they selectively merge into and override arguments defined in existing resource or module declarations.
   * **Best Practice:** Used primarily for emergency hotfixes, local developer testing, or temporarily changing specific Azure settings without modifying the main codebase.
 * **Example:** Overriding an Azure App Service SKU tier locally:
```hcl
# override.tf
resource "azurerm_service_plan" "app_plan" {
  # Overrides sku_name to a cheaper tier for local sandbox testing
  sku_name = "B1"
}

```
### 3. How do you construct string templates and multi-line strings in HCL using Heredoc syntax?
 * **Answer:** * Use the Heredoc syntax (<<EOT ... EOT) or stripped Heredoc (<<-EOT ... EOT) to declare multi-line string blocks (e.g., cloud-init user-data scripts or JSON parameters) cleanly.
   * Stripped syntax (<<-EOT) automatically removes leading whitespace indentation, preserving clean code formatting.
 * **Example:** Passing a custom cloud-init script into an Azure Linux Virtual Machine:
```hcl
locals {
  custom_data_script = <<-EOT
    #!/bin/bash
    apt-get update -y
    apt-get install -y nginx
    systemctl enable --now nginx
  EOT
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                = "vm-web-prod"
  resource_group_name = "rg-web"
  location            = "East US"
  size                = "Standard_B2s"
  admin_username      = "azureuser"

  # Encodes multi-line script for Azure custom data
  custom_data = base64encode(locals.custom_data_script)

  # ... Network interface & disk configs ...
}

```
## Intermediate Level Questions
### 4. How do you configure Azure Web App Regional VNet Integration using Terraform?
 * **Answer:** Provision the Linux or Windows Web App along with a designated Azure Subnet containing the Microsoft.Web/serverFarms delegation. Connect the app to the subnet using azurerm_app_service_virtual_network_swift_connection.
 * **Example:**
```hcl
# Delegated Subnet for App Service VNet Integration
resource "azurerm_subnet" "vnet_integration_subnet" {
  name                 = "snet-app-vnet-integration"
  resource_group_name  = "rg-networking"
  virtual_network_name = "vnet-spoke"
  address_prefixes     = ["10.1.2.0/24"]

  delegation {
    name = "appservice-delegation"
    service_delegation {
      name    = "Microsoft.Web/serverFarms"
      actions = ["Microsoft.Network/virtualNetworks/subnets/action"]
    }
  }
}

# Link Web App to Delegated Subnet
resource "azurerm_app_service_virtual_network_swift_connection" "vnet_integration" {
  app_service_id = azurerm_linux_web_app.app.id
  subnet_id      = azurerm_subnet.vnet_integration_subnet.id
}

```
### 5. How do you manage Azure Storage Accounts with multiple Private Endpoints targeting different sub-resources (blob, file, queue, table)?
 * **Answer:** Each Azure Storage sub-resource requires its own distinct azurerm_private_endpoint resource and specific Private DNS Zone target (e.g., privatelink.blob.core.windows.net, privatelink.file.core.windows.net).
 * **Example:** Creating Private Endpoints for both Blob and File services on a single Storage Account using for_each:
```hcl
variable "storage_subresources" {
  type = map(string)
  default = {
    "blob" = "privatelink.blob.core.windows.net"
    "file" = "privatelink.file.core.windows.net"
  }
}

resource "azurerm_private_endpoint" "st_endpoints" {
  for_each            = var.storage_subresources
  name                = "pe-st-${each.key}-prod"
  location            = "East US"
  resource_group_name = "rg-data"
  subnet_id           = "/subscriptions/.../subnets/snet-private-endpoints"

  private_service_connection {
    name                           = "psc-st-${each.key}"
    private_connection_resource_id = azurerm_storage_account.st.id
    is_manual_connection           = false
    subresource_names              = [each.key] # Target 'blob' or 'file'
  }
}

```
### 6. How do you provision a standalone Azure Web Application Firewall (WAF) Policy and attach it to an Application Gateway in Terraform?
 * **Answer:** Define an independent azurerm_web_application_firewall_policy with custom rules (e.g., IP rate limiting or Geo-blocking). Next, link the policy to the azurerm_application_gateway by assigning its ID to the firewall_policy_id property.
 * **Example:**
```hcl
# Standalone Azure WAF Policy
resource "azurerm_web_application_firewall_policy" "waf_policy" {
  name                = "waf-policy-global"
  resource_group_name = "rg-security"
  location            = "East US"

  policy_settings {
    enabled = true
    mode    = "Prevention"
  }

  managed_rules {
    managed_rule_set {
      type    = "OWASP"
      version = "3.2"
    }
  }
}

# Attach WAF Policy to Application Gateway
resource "azurerm_application_gateway" "appgw" {
  name                = "agw-web-prod"
  resource_group_name = "rg-networking"
  location            = "East US"
  firewall_policy_id  = azurerm_web_application_firewall_policy.waf_policy.id

  # ... Gateway configuration ...
}

```
### 7. How do you configure soft-delete and purge protection on Azure Key Vault using Terraform?
 * **Answer:** Set soft_delete_retention_days (between 7 and 90 days) and set purge_protection_enabled = true. Additionally, specify the provider behavior in the features { key_vault { ... } } block to manage behavior when running terraform destroy.
 * **Example:**
```hcl
provider "azurerm" {
  features {
    key_vault {
      # Retains soft-deleted Key Vaults upon destroy instead of forcing immediate purge
      purge_soft_delete_on_destroy = false
    }
  }
}

resource "azurerm_key_vault" "kv" {
  name                       = "kv-sec-prod-001"
  location                   = "East US"
  resource_group_name        = "rg-sec"
  tenant_id                  = "00000000-0000-0000-0000-000000000000"
  sku_name                   = "standard"
  soft_delete_retention_days = 90
  purge_protection_enabled   = true # Prevents permanent purge during retention window
}

```
## Advanced Level Questions
### 8. How do you mock Azure provider dependencies in terraform test without creating real cloud infrastructure?
 * **Answer:** Modern terraform test supports mock providers using override_provider blocks inside .tftest.hcl files. This validates resource configurations, conditions, and dynamic variable transformations without executing live calls against Azure APIs.
 * **Example:**
```hcl
# tests/unit_test.tftest.hcl
override_provider {
  azurerm = {} # Mocks azurerm API responses locally
}

run "validate_vnet_creation_logic" {
  command = plan

  assert {
    condition     = azurerm_virtual_network.vnet.address_space[0] == "10.0.0.0/16"
    error_message = "VNet address space configuration must equal 10.0.0.0/16."
  }
}

```
### 9. How do you provision Azure Monitor Managed Service for Prometheus and Managed Grafana using Terraform?
 * **Answer:** Deploy azurerm_monitor_workspace (Prometheus backend) along with azurerm_dashboard_grafana. Establish a data source connection between Grafana and the Azure Monitor Workspace using Azure RBAC role assignments (Monitoring Reader).
 * **Example:**
```hcl
# 1. Azure Monitor Workspace (Prometheus)
resource "azurerm_monitor_workspace" "prometheus" {
  name                = "prom-workspace-prod"
  resource_group_name = "rg-ops"
  location            = "East US"
}

# 2. Azure Managed Grafana
resource "azurerm_dashboard_grafana" "grafana" {
  name                = "grafana-dashboard-prod"
  resource_group_name = "rg-ops"
  location            = "East US"
  sku                 = "Standard"

  identity {
    type = "SystemAssigned"
  }
}

# 3. Assign Monitoring Reader permissions to Grafana Identity
resource "azurerm_role_assignment" "grafana_prom_reader" {
  scope                = azurerm_monitor_workspace.prometheus.id
  role_definition_name = "Monitoring Reader"
  principal_id         = azurerm_dashboard_grafana.grafana.identity[0].principal_id
}

```
### 10. How do you implement Run Triggers and Remote Workspaces in Terraform Cloud/Enterprise when orchestrating multi-layer Azure deployments?
 * **Answer:** * Split infrastructure into isolated workspaces (e.g., 1-networking, 2-database, 3-app-compute).
   * Use **Run Triggers** in Terraform Cloud to automatically chain execution workflows (e.g., applying updates to 1-networking automatically triggers a plan in 2-database).
   * Pass upstream resource attributes across workspaces using the terraform_remote_state data source pointed at the remote org/workspace.
 * **Example:**
```hcl
# App Compute Workspace reading outputs from Remote Networking Workspace
data "terraform_remote_state" "networking_remote" {
  backend = "remote"

  config = {
    organization = "enterprise-corp"
    workspaces = {
      name = "1-networking-prod"
    }
  }
}

resource "azurerm_linux_web_app" "app" {
  name                = "app-orders-prod"
  resource_group_name = "rg-app"
  location            = "East US"
  service_plan_id     = "/subscriptions/.../servicePlans/asp-prod"

  # Accesses VNet subnet output from the remote networking workspace
  virtual_network_subnet_id = data.terraform_remote_state.networking_remote.outputs.app_subnet_id
}

```
Would you like to explore any specific Azure domain in more detail, such as **AKS / Containers**, **Enterprise Governance & Policies**, or **Hub-and-Spoke Networking**?

---

Here is a **sixteenth set** of 10 scenario-focused Terraform and Microsoft Azure interview questions and answers, complete with practical HCL code examples.
## Basic Level Questions
### 1. What is the difference between terraform taint (deprecated) and terraform apply -replace?
 * **Answer:**
   * **terraform taint (Legacy):** Interactively marked a resource as degraded directly inside the state file. On the next execution, Terraform would destroy and recreate that specific resource.
   * **terraform apply -replace (Modern Standard):** Instructs Terraform to force-replace a specific resource during a single execution run *without* permanently mutating the state file beforehand.
 * **Example:**
   ```bash
   # Force recreation of a specific Azure Virtual Machine during apply
   terraform apply -replace="azurerm_linux_virtual_machine.app_vm"
   
   ```
### 2. How do you reference an Azure Managed Disk's ID when attaching it to an existing VM?
 * **Answer:** Define the disk using azurerm_managed_disk and link it to the VM using the dedicated azurerm_virtual_machine_data_disk_attachment resource. This prevents inline disk configuration changes from forcing the recreation of the underlying Virtual Machine.
 * **Example:**
   ```hcl
   resource "azurerm_managed_disk" "data_disk" {
     name                 = "disk-data-prod"
     location             = "East US"
     resource_group_name  = "rg-compute"
     storage_account_type = "Premium_LRS"
     create_option        = "Empty"
     disk_size_gb         = 128
   }
   
   resource "azurerm_virtual_machine_data_disk_attachment" "disk_attach" {
     managed_disk_id    = azurerm_managed_disk.data_disk.id
     virtual_machine_id = azurerm_linux_virtual_machine.vm.id
     lun                = "10"
     caching            = "ReadWrite"
   }
   
   ```
### 3. How do you use the can() function in local variables for safe attribute validation?
 * **Answer:** The can() function evaluates an expression and returns true if no errors occur, and false if an error (like a missing key or null reference) is raised. It is commonly used in locals or input validations to handle optional nested properties cleanly.
 * **Example:**
   ```hcl
   variable "vnet_config" {
     type    = any
     default = { address_space = ["10.0.0.0/16"] }
   }
   
   locals {
     # Validates if custom DNS servers were provided without throwing a runtime error
     has_custom_dns = can(var.vnet_config.dns_servers[0])
   }
   
   ```
## Intermediate Level Questions
### 4. How do you deploy an Azure Function App with a Managed Identity and Key Vault reference in its App Settings?
 * **Answer:** Create an azurerm_linux_function_app with a System-Assigned Managed Identity enabled. Grant that identity access to Key Vault secrets via RBAC (Key Vault Secrets User). Then, use the native @Microsoft.KeyVault(...) syntax inside app_settings.
 * **Example:**
   ```hcl
   resource "azurerm_linux_function_app" "func" {
     name                = "func-processor-prod"
     resource_group_name = "rg-apps"
     location            = "East US"
     service_plan_id     = azurerm_service_plan.plan.id
   
     storage_account_name       = azurerm_storage_account.st.name
     storage_account_access_key = azurerm_storage_account.st.primary_access_key
   
     identity {
       type = "SystemAssigned"
     }
   
     app_settings = {
       # App pulls secret directly from Key Vault at runtime
       "DbConnectionString" = "@Microsoft.KeyVault(SecretUri=https://kv-prod.vault.azure.net/secrets/db-conn/)"
     }
   }
   
   ```
### 5. How do you configure Azure Service Bus Topics and Subscriptions with dynamic Filters using Terraform?
 * **Answer:** Provision an azurerm_servicebus_topic, create an azurerm_servicebus_subscription, and attach custom SQL or Correlation filtering rules via azurerm_servicebus_subscription_rule.
 * **Example:**
   ```hcl
   resource "azurerm_servicebus_topic" "orders" {
     name         = "sbt-orders-events"
     namespace_id = azurerm_servicebus_namespace.sb.id
   }
   
   resource "azurerm_servicebus_subscription" "sub_priority" {
     name               = "sub-priority-orders"
     topic_id           = azurerm_servicebus_topic.orders.id
     max_delivery_count = 10
   }
   
   resource "azurerm_servicebus_subscription_rule" "rule_priority" {
     name            = "rule-high-priority"
     subscription_id = azurerm_servicebus_subscription.sub_priority.id
     filter_type     = "SqlFilter"
     sql_filter      = "User.Priority = 'High'"
   }
   
   ```
### 6. How do you enforce Private Endpoint deployments for Azure SQL Database using Terraform?
 * **Answer:** Set public_network_access_enabled = false on azurerm_mssql_server to disable public connectivity. Then, bind an azurerm_private_endpoint targeting the sqlServer subresource to a private subnet.
 * **Example:**
   ```hcl
   resource "azurerm_mssql_server" "sql" {
     name                         = "sql-orders-prod"
     resource_group_name          = "rg-data"
     location                     = "East US"
     version                      = "12.0"
     administrator_login          = "sqladmin"
     administrator_login_password = "ComplexPassword123!"
   
     public_network_access_enabled = false # Blocks all public access
   }
   
   resource "azurerm_private_endpoint" "sql_pe" {
     name                = "pe-sql-prod"
     location            = "East US"
     resource_group_name = "rg-data"
     subnet_id           = "/subscriptions/.../subnets/snet-private"
   
     private_service_connection {
       name                           = "psc-sql-prod"
       private_connection_resource_id = azurerm_mssql_server.sql.id
       is_manual_connection           = false
       subresource_names              = ["sqlServer"]
     }
   }
   
   ```
### 7. How do you write cross-field validation rules using precondition blocks inside a Resource block?
 * **Answer:** Use a precondition block inside the resource's lifecycle block. This allows you to evaluate expressions that compare multiple variables or resource parameters before Terraform sends API execution calls to Azure.
 * **Example:**
   ```hcl
   variable "sku_tier" { default = "Premium" }
   variable "enable_zone_redundancy" { default = true }
   
   resource "azurerm_service_plan" "asp" {
     name                = "asp-web-prod"
     resource_group_name = "rg-web"
     location            = "East US"
     os_type             = "Linux"
     sku_name            = "P1v2"
   
     lifecycle {
       precondition {
         condition     = var.enable_zone_redundancy == false || var.sku_tier == "Premium"
         error_message = "Zone Redundancy can only be enabled when sku_tier is set to 'Premium'."
       }
     }
   }
   
   ```
## Advanced Level Questions
### 8. How do you structure an Enterprise Hub-and-Spoke Network Architecture with Azure Firewall routing in Terraform?
 * **Answer:**
   1. Build the Hub VNet containing AzureFirewallSubnet and deploy azurerm_firewall.
   2. Build Spoke VNets and create two-way VNet Peerings (azurerm_virtual_network_peering) between Hub and Spokes.
   3. Create an azurerm_route_table with a 0.0.0.0/0 default route pointing to the Azure Firewall's private IP as the next hop (VirtualAppliance).
   4. Associate the Route Table with all Spoke Subnets.
 * **Example:**
   ```hcl
   resource "azurerm_route_table" "spoke_rt" {
     name                = "rt-spoke-to-firewall"
     location            = "East US"
     resource_group_name = "rg-hub-networking"
   
     route {
       name                   = "default-to-firewall"
       address_prefix         = "0.0.0.0/0"
       next_hop_type          = "VirtualAppliance"
       next_hop_in_ip_address = azurerm_firewall.fw.ip_configuration[0].private_ip_address
     }
   }
   
   resource "azurerm_subnet_route_table_association" "spoke_assoc" {
     subnet_id      = azurerm_subnet.spoke_app_subnet.id
     route_table_id = azurerm_route_table.spoke_rt.id
   }
   
   ```
### 9. How do you implement Zero-Downtime Blue/Green deployments for Azure Virtual Machine Scale Sets (VMSS) using custom capacity rolling updates?
 * **Answer:** Configure azurerm_linux_virtual_machine_scale_set with an upgrade_mode = "Rolling" policy and couple it with rolling_upgrade_policy settings. Use lifecycle { create_before_destroy = true } on dependent load balancer configurations to prevent traffic drops during updates.
 * **Example:**
   ```hcl
   resource "azurerm_linux_virtual_machine_scale_set" "vmss" {
     name                = "vmss-web-prod"
     resource_group_name = "rg-compute"
     location            = "East US"
     sku                 = "Standard_D2s_v3"
     instances           = 3
     upgrade_mode        = "Rolling"
   
     rolling_upgrade_policy {
       max_batch_instance_percent              = 20
       max_unhealthy_instance_percent          = 20
       max_unhealthy_upgraded_instance_percent = 5
       pause_time_between_batches              = "PT0S"
     }
   
     # ... Network and OS Profile configurations ...
   }
   
   ```
### 10. How do you configure Terraform OIDC Federated Authentication for GitHub Actions without using long-lived Azure Client Secrets?
 * **Answer:**
   1. Create a User-Assigned Identity or Entra App Registration.
   2. Create an azurerm_federated_identity_credential linking the GitHub repository, environment, or pull-request subject claim to the Azure identity.
   3. Configure the azurerm provider block to use use_oidc = true.
   4. In the GitHub Actions workflow, pass client-id, tenant-id, and subscription-id via the azure/login action using short-lived tokens.
 * **Example:**
   ```hcl
   resource "azurerm_user_assigned_identity" "github_oidc" {
     name                = "id-github-actions-prod"
     location            = "East US"
     resource_group_name = "rg-sec"
   }
   
   resource "azurerm_federated_identity_credential" "github_federated" {
     name                = "fed-github-main-branch"
     resource_group_name = "rg-sec"
     audience            = ["api://AzureADTokenExchange"]
     issuer              = "https://token.actions.githubusercontent.com"
     subject             = "repo:my-org/my-repo:ref:refs/heads/main"
     parent_id           = azurerm_user_assigned_identity.github_oidc.id
   }
   
   ```

---


Here is a **seventeenth set** of brand-new, scenario-based Terraform and Microsoft Azure interview questions and answers. To help you learn like a student, every answer breaks down **"The Concept"** in simple terms first, followed by **"The Interview Answer"**, and ends with a practical **HCL Code Example**.
## Basic Level Questions
### 1. What is the difference between count.index and each.key in Terraform loops?
 * **The Concept:** Think of count.index like a numbered list ([0, 1, 2]) and each.key like a labeled map ({"dev": "East US", "prod": "West US"}). If you delete item #1 from a numbered list, everything after it shifts positions, causing Terraform to recreate resources. With each.key, every item keeps its unique name forever.
 * **The Interview Answer:** * count.index is used inside resources configured with the count meta-argument to reference the current integer index.
   * each.key (and each.value) is used inside resources configured with for_each to reference the current map key or set member.
   * **Best Practice:** Use for_each and each.key when managing distinct cloud infrastructure (like Azure Subnets or Resource Groups) to maintain stable resource addresses in the state file.
 * **HCL Example:**
```hcl
variable "resource_groups" {
  type    = map(string)
  default = {
    "rg-networking-prod" = "East US"
    "rg-compute-prod"    = "East US2"
  }
}

resource "azurerm_resource_group" "rg" {
  for_each = var.resource_group_name # Using for_each over a map
  name     = each.key               # Evaluates to "rg-networking-prod", etc.
  location = each.value             # Evaluates to "East US", etc.
}

```
### 2. How do you configure Storage Account Network Rules to allow traffic from Azure Services while blocking the public internet?
 * **The Concept:** By default, Azure Storage Accounts accept traffic from any IP on the internet. To secure them, you flip the default action to "Deny" and create exceptions (bypass rules) for trusted Microsoft services like Azure Backup or Azure Monitor.
 * **The Interview Answer:** You configure the network_rules block inside azurerm_storage_account. Set default_action = "Deny" and add bypass = ["AzureServices"].
 * **HCL Example:**
```hcl
resource "azurerm_storage_account" "secure_st" {
  name                     = "stdataappprod001"
  resource_group_name      = "rg-data"
  location                 = "East US"
  account_tier             = "Standard"
  account_replication_type = "LRS"

  network_rules {
    default_action = "Deny"             # Blocks all public internet traffic by default
    bypass         = ["AzureServices"]  # Allows trusted internal Azure platform services
  }
}

```
### 3. What is the azurerm_client_config Data Source, and why is it used so frequently?
 * **The Concept:** When running Terraform in a CI/CD pipeline, you often need to know *who* is running the pipeline (the Tenant ID, Subscription ID, or Service Principal Object ID) to assign permissions dynamically without hardcoding them.
 * **The Interview Answer:** The azurerm_client_config data source queries the current authenticated Azure session. It retrieves metadata like tenant_id, subscription_id, client_id, and object_id directly from the active provider authentication context.
 * **HCL Example:**
```hcl
# Reads the active authentication context
data "azurerm_client_config" "current" {}

resource "azurerm_key_vault" "kv" {
  name                = "kv-app-prod"
  location            = "East US"
  resource_group_name = "rg-sec"
  sku_name            = "standard"
  tenant_id           = data.azurerm_client_config.current.tenant_id # Dynamically uses current tenant ID
}

```
## Intermediate Level Questions
### 4. How do you configure an Azure Container App Environment with an internal Virtual Network using Terraform?
 * **The Concept:** Azure Container Apps run microservices inside serverless containers. To keep them isolated from the internet, you deploy them inside an internal azurerm_container_app_environment linked to a dedicated custom subnet.
 * **The Interview Answer:** Create a dedicated subnet, set internal_load_balancer_enabled = true inside azurerm_container_app_environment, and supply the infrastructure_subnet_id.
 * **HCL Example:**
```hcl
resource "azurerm_container_app_environment" "env" {
  name                       = "cae-microservices-prod"
  location                   = "East US"
  resource_group_name        = "rg-apps"
  infrastructure_subnet_id   = "/subscriptions/.../subnets/snet-containerapps"
  internal_load_balancer_enabled = true # Restricts access to internal VNet only
}

resource "azurerm_container_app" "app" {
  name                         = "ca-orders-api"
  container_app_environment_id = azurerm_container_app_environment.env.id
  resource_group_name          = "rg-apps"
  revision_mode                = "Single"

  template {
    container {
      name   = "api"
      image  = "mcr.microsoft.com/azuredocs/aci-helloworld:latest"
      cpu    = 0.5
      memory = "1.0Gi"
    }
  }
}

```
### 5. How do you use terraform_remote_state with Azure Blob Storage backends safely in enterprise environments?
 * **The Concept:** When Team B needs information produced by Team A (like a Subnet ID created by the Networking team), Team B reads Team A's state file in read-only mode.
 * **The Interview Answer:** terraform_remote_state acts as a data source that connects to an external state file stored in an Azure Storage Account container. It exposes the outputs published by that root module while preventing accidental modifications to the remote state.
 * **HCL Example:**
```hcl
# Reads the Networking Team's remote state file stored in Azure Blob Storage
data "terraform_remote_state" "networking" {
  backend = "azurerm"
  config = {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "sttfstateacct"
    container_name       = "tfstate"
    key                  = "networking.prod.tfstate"
  }
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                = "vm-app-prod"
  resource_group_name = "rg-compute"
  location            = "East US"
  size                = "Standard_B2s"

  network_interface_ids = [
    azurerm_network_interface.nic.id
  ]
  # ... VM OS & Disk settings ...
}

resource "azurerm_network_interface" "nic" {
  name                = "nic-app-prod"
  location            = "East US"
  resource_group_name = "rg-compute"

  ip_configuration {
    name                          = "internal"
    # Consumes output from remote state file
    subnet_id                     = data.terraform_remote_state.networking.outputs.app_subnet_id
    private_ip_address_allocation = "Dynamic"
  }
}

```
### 6. How do you provision Azure Event Hubs with Auto-Inflate (Auto-Scaling) enabled?
 * **The Concept:** An Event Hub stream can suddenly get overwhelmed by high data volume. Instead of manually increasing throughput units during an outage, you enable "Auto-Inflate" so Azure automatically scales up throughput units when traffic spikes.
 * **The Interview Answer:** In azurerm_eventhub_namespace, set auto_inflate_enabled = true and specify maximum_throughput_units.
 * **HCL Example:**
```hcl
resource "azurerm_eventhub_namespace" "eh_namespace" {
  name                = "ehns-telemetry-prod"
  location            = "East US"
  resource_group_name = "rg-streaming"
  sku                 = "Standard"
  capacity            = 1 # Baseline Throughput Units

  auto_inflate_enabled     = true # Enables dynamic scaling
  maximum_throughput_units = 10   # Sets upper limit for auto-scaling
}

resource "azurerm_eventhub" "hub" {
  name                = "eh-logs-stream"
  namespace_name      = azurerm_eventhub_namespace.eh_namespace.name
  resource_group_name = "rg-streaming"
  partition_count     = 4
  message_retention   = 1
}

```
### 7. How do you enforce compliance on Azure Private DNS Zones using terraform_data triggers?
 * **Answer:**
 * **The Concept:** Sometimes you need a task or script to run **only when a specific value changes** (like forcing a cache clear script when a DNS record IP changes). terraform_data holds arbitrary values and triggers lifecycle actions whenever those tracked values change.
 * **The Interview Answer:** Use terraform_data (introduced in Terraform 1.4) to replace legacy null_resource. Track attributes in the triggers_replace argument to force dependent provisioning triggers whenever target values drift.
 * **HCL Example:**
```hcl
resource "azurerm_private_dns_a_record" "db_dns" {
  name                = "db"
  zone_name           = "privatelink.database.windows.net"
  resource_group_name = "rg-networking"
  ttl                 = 300
  records             = ["10.0.1.25"]
}

# Native replacement for null_resource
resource "terraform_data" "dns_cache_flush_trigger" {
  triggers_replace = [
    azurerm_private_dns_a_record.db_dns.records # Fires action whenever IP record changes
  ]

  provisioner "local-exec" {
    command = "echo DNS IP updated to ${azurerm_private_dns_dns_a_record.db_dns.records[0]} >> /tmp/dns_changes.log"
  }
}

```
## Advanced Level Questions
### 8. How do you implement zero-downtime Blue/Green deployments for Azure App Services using Deployment Slots and Traffic Routing?
 * **The Concept:** You deploy new code to a hidden "staging" slot first. You test it, and if it passes, you tell Azure App Service to swap the staging slot with production instantly, resulting in zero user downtime.
 * **The Interview Answer:** Define an azurerm_linux_web_app (Production) and an azurerm_linux_web_app_slot (Staging). Use the routing_rule parameter inside the production app's site_config to split live user traffic gradually (e.g., sending 10% of users to Staging for testing) before issuing a full slot swap.
 * **HCL Example:**
```hcl
resource "azurerm_service_plan" "asp" {
  name                = "asp-web-prod"
  resource_group_name = "rg-web"
  location            = "East US"
  os_type             = "Linux"
  sku_name            = "P1v2"
}

# Production Primary App
resource "azurerm_linux_web_app" "production" {
  name                = "app-store-prod"
  resource_group_name = "rg-web"
  location            = "East US"
  service_plan_id     = azurerm_service_plan.asp.id

  site_config {
    # Routes 10% of live production traffic to the 'staging' slot for canary testing
    traffic_routing {
      action = "Reroute"
      name   = "staging"
      weight = 10
    }
  }
}

# Staging Slot
resource "azurerm_linux_web_app_slot" "staging" {
  name           = "staging"
  app_service_id = azurerm_linux_web_app.production.id

  site_config {}
}

```
### 9. How do you manage Azure Resource Group-level Role Assignments dynamically using for_each and custom maps?
 * **The Concept:** Assigning permissions manually for 10 different teams across 5 resource groups leads to duplicated code. Instead, you define a single map structure that binds target roles to group object IDs and loop over it.
 * **The Interview Answer:** Construct a structured map variable containing security principal IDs, target roles, and resource group scopes. Use for_each on azurerm_role_assignment and generate deterministic UUID names using uuidv5() to ensure idempotent role creation.
 * **HCL Example:**
```hcl
variable "rbac_mappings" {
  type = map(object({
    principal_id = string
    role_name    = string
    rg_id        = string
  }))
  default = {
    "dev_team_reader" = {
      principal_id = "00000000-0000-0000-0000-000000000001"
      role_name    = "Reader"
      rg_id        = "/subscriptions/0000.../resourceGroups/rg-dev"
    },
    "ops_team_contributor" = {
      principal_id = "00000000-0000-0000-0000-000000000002"
      role_name    = "Contributor"
      rg_id        = "/subscriptions/0000.../resourceGroups/rg-ops"
    }
  }
}

resource "azurerm_role_assignment" "assignments" {
  for_each             = var.rbac_mappings
  scope                = each.value.rg_id
  role_definition_name = each.value.role_name
  principal_id         = each.value.principal_id

  # Generates a deterministic UUID to prevent duplicate role assignment collisions
  name = uuidv5("url", "${each.value.rg_id}-${each.value.principal_id}-${each.value.role_name}")
}

```
### 10. How do you write unit tests in .tftest.hcl to assert that an Azure Storage Account blocks HTTP traffic?
 * **The Concept:** Before running code in a production environment, unit tests run terraform plan against test conditions to guarantee that mandatory security parameters (like forcing HTTPS encryption) are enabled.
 * **The Interview Answer:** Use native terraform test framework files (.tftest.hcl). Define a run block executing command = plan combined with an assert block evaluating the state of azurerm_storage_account.st.supports_https_traffic_only.
 * **HCL Example:**
```hcl
# tests/security_compliance.tftest.hcl

# Mock provider setup to avoid real cloud deployment costs
override_provider {
  azurerm = {}
}

run "verify_https_enforcement" {
  command = plan # Evaluates plan output logic

  assert {
    condition     = azurerm_storage_account.st.supports_https_traffic_only == true
    error_message = "Security Violation: Storage account must enforce HTTPS traffic."
  }

  assert {
    condition     = azurerm_storage_account.st.min_tls_version == "TLS1_2"
    error_message = "Security Violation: Minimum TLS version must be configured to TLS1_2."
  }
}

```

---

Here is an **eighteenth set** of brand-new, scenario-based Terraform and Microsoft Azure interview questions and answers, designed in the student-friendly format with **The Concept**, **The Interview Answer**, and an **HCL Code Example**.
## Basic Level Questions
### 1. What is the difference between an azurerm Resource declaration and an azurerm Data Source lookup?
 * **The Concept:** Think of a **Resource** as buying a brand-new house—you define its features, and Terraform constructs it for you in Azure. A **Data Source** is like looking up an address in a phone book—the house already exists, and you just want to find its address or details so you can send mail there.
 * **The Interview Answer:** * resource "azurerm_...": Tells Terraform to create, manage, and track a *new* Azure infrastructure component in the state file.
   * data "azurerm_...": Tells Terraform to perform a *read-only* query against Azure Resource Manager (ARM) APIs to fetch attributes of a pre-existing resource without managing its lifecycle.
 * **HCL Example:**
```hcl
# Data Source: Looking up an existing Resource Group
data "azurerm_resource_group" "existing_rg" {
  name = "rg-shared-infrastructure-prod"
}

# Resource: Creating a NEW Storage Account inside that existing Resource Group
resource "azurerm_storage_account" "new_storage" {
  name                     = "stappdata002"
  resource_group_name      = data.azurerm_resource_group.existing_rg.name # References data lookup
  location                 = data.azurerm_resource_group.existing_rg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

```
### 2. What is the templatefile() function, and how is it used to pass variables into Azure scripts?
 * **The Concept:** Imagine writing a form letter with blank spots like "Hello [Name]". The templatefile() function reads a script file on your computer, replaces all the [Name] placeholders with real Terraform variables, and hands the completed script to your Azure Virtual Machine.
 * **The Interview Answer:** templatefile(path, vars) reads an external file containing template syntax (${var_name}) and renders its content by substituting specified local or input variables. It is the standard way to dynamically inject parameters into Azure Linux Custom Data scripts or Windows PowerShell cloud-init scripts.
 * **HCL Example:**
```hcl
# File: scripts/userdata.sh.tftpl
# #!/bin/bash
# echo "Environment: ${env_name}" > /tmp/env.txt
# echo "Database Endpoint: ${db_host}" >> /tmp/env.txt

resource "azurerm_linux_virtual_machine" "vm" {
  name                = "vm-web-prod"
  resource_group_name = "rg-app"
  location            = "East US"
  size                = "Standard_B2s"

  # Renders the template script with live Terraform variables
  custom_data = base64encode(
    templatefile("${path.module}/scripts/userdata.sh.tftpl", {
      env_name = "production"
      db_host  = "sql-db.database.windows.net"
    })
  )

  # ... Network interface and OS disk configuration ...
}

```
### 3. What is the purpose of the azurerm_subnet_service_endpoint_storage_policy in Azure networking?
 * **The Concept:** A Service Endpoint lets your private subnet talk directly to Azure Storage. But what if an attacker inside your network tries to copy data to their own personal storage account? A Storage Service Endpoint Policy acts like a guard rail, allowing traffic *only* to authorized enterprise storage accounts.
 * **The Interview Answer:** An Azure Storage Service Endpoint Policy restricts outbound Service Endpoint traffic from a subnet to specific Azure Storage accounts. This prevents data exfiltration by blocking access to unauthorized storage accounts over internal networking paths.
 * **HCL Example:**
```hcl
# Service Endpoint Policy allowing access ONLY to enterprise storage
resource "azurerm_subnet_service_endpoint_storage_policy" "policy" {
  name                = "policy-allow-prod-storage-only"
  resource_group_name = "rg-networking"
  location            = "East US"

  definition {
    name        = "allow-enterprise-storage"
    description = "Allows access only to the production storage account"
    service_resources = [
      "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-data/providers/Microsoft.Storage/storageAccounts/stproddata001"
    ]
  }
}

```
## Intermediate Level Questions
### 4. How do you configure Azure Key Vault to use Azure RBAC for authorization instead of Access Policies?
 * **The Concept:** Traditional Key Vaults use "Access Policies" where you explicitly list permissions for each user or app inside the Key Vault itself. "Azure RBAC Mode" replaces this by letting you manage Key Vault access at the Resource Group or Subscription level using standard Azure roles like "Key Vault Secrets User".
 * **The Interview Answer:** Enable enable_rbac_authorization = true on azurerm_key_vault. This disables local Key Vault Access Policies and allows authorization to be managed via azurerm_role_assignment using built-in roles like "Key Vault Secrets Officer" or "Key Vault Secrets User".
 * **HCL Example:**
```hcl
resource "azurerm_key_vault" "kv" {
  name                      = "kv-security-prod-001"
  location                  = "East US"
  resource_group_name       = "rg-sec"
  tenant_id                 = "00000000-0000-0000-0000-000000000000"
  sku_name                  = "standard"
  enable_rbac_authorization = true # Switches authorization to Azure RBAC
}

# Assign secrets reading access via RBAC
resource "azurerm_role_assignment" "kv_user" {
  scope                = azurerm_key_vault.kv.id
  role_definition_name = "Key Vault Secrets User"
  principal_id         = "11111111-1111-1111-1111-111111111111" # Managed Identity Object ID
}

```
### 5. How do you configure Static Website Hosting on an Azure Storage Account using Terraform?
 * **The Concept:** You don't always need a heavy web server to host a single-page app (like React or Angular). An Azure Storage Account can serve static HTML, CSS, and JS files directly to the web at a fraction of the cost.
 * **The Interview Answer:** Use the static_website block inside azurerm_storage_account. Specify the index_document (e.g., index.html) and error_404_document (e.g., 404.html). Terraform automatically enables the $web storage container.
 * **HCL Example:**
```hcl
resource "azurerm_storage_account" "website" {
  name                     = "ststaticwebprod001"
  resource_group_name      = "rg-web"
  location                 = "East US"
  account_tier             = "Standard"
  account_replication_type = "LRS"

  # Enables static web hosting capabilities
  static_website {
    index_document     = "index.html"
    error_404_document = "404.html"
  }
}

output "web_endpoint" {
  value = azurerm_storage_account.website.primary_web_endpoint # Public web URL
}

```
### 6. How do you create an Azure Private Link Service to securely expose an internal service to consumer VNets?
 * **The Concept:** Imagine your company builds an API that other partner companies need to access. Instead of exposing your API to the public internet, you create a "Private Link Service" behind your load balancer so partners can connect to it securely through their own private VNet.
 * **The Interview Answer:** Provision an azurerm_private_link_service attached to the Frontend IP Configuration of a Standard Internal Azure Load Balancer (azurerm_lb). Consumer subscriptions can then attach Private Endpoints targeting this service.
 * **HCL Example:**
```hcl
resource "azurerm_private_link_service" "pls" {
  name                = "pls-provider-service"
  location            = "East US"
  resource_group_name = "rg-provider"

  # Link to Internal Standard Load Balancer Frontend IP
  load_balancer_frontend_ip_configuration_ids = [
    "/subscriptions/.../loadBalancers/lb-internal/frontendIPConfigurations/ip-config-internal"
  ]

  nat_ip_configuration {
    name      = "pls-nat-config"
    primary   = true
    subnet_id = "/subscriptions/.../subnets/snet-pls-nat"
  }
}

```
### 7. How do you configure Azure Log Analytics Workspace Data Retention policies dynamically using Terraform?
 * **The Concept:** Compliance requirements might say "keep production logs for 365 days, but development logs for only 30 days." You can set log retention rules inside your code based on variables.
 * **The Interview Answer:** Set retention_in_days inside azurerm_log_analytics_workspace. You can also configure table-level overrides using azurerm_log_analytics_workspace_table to keep specific high-value audit tables longer than standard operational logs.
 * **HCL Example:**
```hcl
variable "env" {
  type    = string
  default = "prod"
}

resource "azurerm_log_analytics_workspace" "law" {
  name                = "law-central-${var.env}"
  location            = "East US"
  resource_group_name = "rg-ops"
  sku                 = "PerGB2018"
  
  # Dynamic retention based on environment
  retention_in_days   = var.env == "prod" ? 365 : 30
}

```
## Advanced Level Questions
### 8. How do you write integration tests in .tftest.hcl that actually provision and destroy real Azure resources (command = apply)?
 * **The Concept:** Unit tests only inspect code locally (command = plan). Integration tests run command = apply to physically create real resources in a sandbox Azure subscription, run test assertions to verify they work, and then destroy everything automatically.
 * **The Interview Answer:** In .tftest.hcl, write a run block with command = apply. Specify assertions comparing actual runtime output attributes from Azure. Terraform provisions the resources, runs the assertions, and teardowns the test stack cleanly upon completion.
 * **HCL Example:**
```hcl
# tests/integration_test.tftest.hcl

run "deploy_and_verify_storage_account" {
  command = apply # Actually creates resources in Azure test subscription!

  assert {
    condition     = azurerm_storage_account.st.primary_location == "eastus"
    error_message = "Storage Account was not deployed to the primary expected region."
  }

  assert {
    condition     = startswith(azurerm_storage_account.st.primary_blob_endpoint, "https://")
    error_message = "Storage Account primary endpoint is not serving over secure HTTPS."
  }
}

```
### 9. How do you handle Azure Provider authentication using Azure Managed Identity when running Terraform inside an Azure VM build agent?
 * **The Concept:** Instead of storing Service Principal credentials or client secrets on a build agent machine (where they could be stolen), you give the build agent VM a System-Assigned Managed Identity and tell Terraform "just use the identity of the VM you are running on".
 * **The Interview Answer:** Set use_msi = true in the provider "azurerm" block and supply subscription_id and tenant_id. Omit client_id and client_secret. Terraform automatically queries the Azure Instance Metadata Service (IMDS) endpoint (http://169.254.169.254) to fetch tokens securely.
 * **HCL Example:**
```hcl
provider "azurerm" {
  features {}

  use_msi         = true # Uses Managed Identity of the host build agent VM
  subscription_id = "00000000-0000-0000-0000-000000000000"
  tenant_id       = "11111111-1111-1111-1111-111111111111"
}

```
### 10. How do you create an Azure Subscription-level Policy Remediation Task using Terraform?
 * **The Concept:** You assigned a policy saying "all Storage Accounts must disable public access." But what about 50 old storage accounts created years ago? A Policy Remediation Task actively scans the subscription and fixes non-compliant existing resources automatically.
 * **The Interview Answer:** Provision an azurerm_subscription_policy_remediation targeting a specific subscription_policy_assignment_id. Azure Resource Manager evaluates non-compliant resources and applies remediation definitions without manual intervention.
 * **HCL Example:**
```hcl
# 1. Subscription Policy Assignment
resource "azurerm_subscription_policy_assignment" "assign_https" {
  name                 = "assign-enforce-https"
  subscription_id      = "/subscriptions/00000000-0000-0000-0000-000000000000"
  policy_definition_id = "/providers/Microsoft.Authorization/policyDefinitions/40474edd-0b44-4b9a-a4d1-6b0bea3f1b03"
}

# 2. Automated Remediation Task for existing infrastructure
resource "azurerm_subscription_policy_remediation" "remediate_existing" {
  name                 = "remediate-old-storage-accounts"
  subscription_id      = "/subscriptions/00000000-0000-0000-0000-000000000000"
  policy_assignment_id = azurerm_subscription_policy_assignment.assign_https.id
}

```

---

Here is a **nineteenth set** of brand-new, scenario-based Terraform and Microsoft Azure interview questions and answers. Each question follows the student-friendly format with **The Concept**, **The Interview Answer**, and an **HCL Code Example**.
## Basic Level Questions
### 1. How do you configure a System-Assigned vs User-Assigned Managed Identity on an Azure Linux VM?
 * **The Concept:** Think of a **System-Assigned Identity** as a driver's license tied to a specific car—if the car is scrapped, the license vanishes with it. A **User-Assigned Identity** is like a physical keycard—you create it separately, and you can hand it to one car, multiple cars, or transfer it whenever you like.
 * **The Interview Answer:** * **System-Assigned:** Defined directly inside the azurerm_linux_virtual_machine resource using identity { type = "SystemAssigned" }. Azure manages its lifecycle automatically.
   * **User-Assigned:** Created independently using azurerm_user_assigned_identity and attached to the VM via identity { type = "UserAssigned", identity_ids = [...] }.
 * **HCL Example:**
```hcl
# 1. Independent User-Assigned Identity
resource "azurerm_user_assigned_identity" "custom_id" {
  name                = "id-vm-app-prod"
  location            = "East US"
  resource_group_name = "rg-compute"
}

# 2. Linux VM with BOTH System-Assigned and User-Assigned Identities
resource "azurerm_linux_virtual_machine" "vm" {
  name                = "vm-app-prod"
  resource_group_name = "rg-compute"
  location            = "East US"
  size                = "Standard_B2s"
  admin_username      = "azureuser"

  # Configures both identity types simultaneously
  identity {
    type         = "SystemAssigned, UserAssigned"
    identity_ids = [azurerm_user_assigned_identity.custom_id.id]
  }

  # ... Network interface and OS disk settings ...
}

```
### 2. How do you filter and transform lists using HCL for expressions with if conditions?
 * **The Concept:** Imagine you have a basket containing apples, oranges, and bananas. A for expression with an if clause lets you say: "Go through the basket, pick *only* the apples, and convert their names to UPPERCASE."
 * **The Interview Answer:** HCL for expressions allow you to iterate over lists or maps, transform attributes, and filter elements based on boolean conditions (if). This is commonly used to extract specific IP prefixes or public subnets from complex variable objects.
 * **HCL Example:**
```hcl
variable "subnets" {
  type = map(object({
    cidr      = string
    is_public = bool
  }))
  default = {
    "snet-web" = { cidr = "10.0.1.0/24", is_public = true }
    "snet-app" = { cidr = "10.0.2.0/24", is_public = false }
    "snet-db"  = { cidr = "10.0.3.0/24", is_public = false }
  }
}

locals {
  # Filters map to pick ONLY private subnets and extracts their CIDRs into a list
  private_subnet_cidrs = [
    for subnet_name, subnet_obj in var.subnets : subnet_obj.cidr
    if subnet_obj.is_public == false
  ] # Evaluates to ["10.0.2.0/24", "10.0.3.0/24"]
}

```
### 3. What is the difference between azurerm_key_vault_secret and azurerm_key_vault_key in Terraform?
 * **The Concept:** An **Azure Key Vault Secret** is like storing a plain piece of paper inside a safe (e.g., a database connection string or API token). An **Azure Key Vault Key** is like putting a master key-cutting machine inside the safe—it performs hardware-based mathematical encryption (RSA/EC) without ever revealing the private key.
 * **The Interview Answer:** * azurerm_key_vault_secret: Stores string data (passwords, certificates, API tokens) as plain text in Key Vault.
   * azurerm_key_vault_key: Generates and manages cryptographic key pairs (RSA, Elliptic Curve) used for Customer-Managed Key (CMK) disk encryption, data signing, and decryption.
 * **HCL Example:**
```hcl
# Secret: Stores textual database connection string
resource "azurerm_key_vault_secret" "db_conn" {
  name         = "sql-connection-string"
  value        = "Server=sql.database.windows.net;Database=db;User=admin;"
  key_vault_id = "/subscriptions/.../vaults/kv-prod"
}

# Key: Generates RSA 2048-bit Cryptographic Key for Disk Encryption
resource "azurerm_key_vault_key" "cmk_key" {
  name         = "cmk-disk-encryption-key"
  key_vault_id = "/subscriptions/.../vaults/kv-prod"
  key_type     = "RSA"
  key_size     = 2048

  key_opts = ["decrypt", "encrypt", "unwrapKey", "wrapKey"]
}

```
## Intermediate Level Questions
### 4. How do you configure an Azure Backup Vault Policy for VMs using Terraform?
 * **The Concept:** Creating a Recovery Services Vault gives you a backup location, but a **Backup Policy** defines the schedule—like "take a snapshot every day at 2:00 AM and keep weekly backups for 30 days." You then link that policy to specific Virtual Machines.
 * **The Interview Answer:** Provision an azurerm_recovery_services_vault, define schedule parameters using azurerm_backup_policy_vm, and link target Virtual Machines to the policy using azurerm_backup_protected_vm.
 * **HCL Example:**
```hcl
# 1. Recovery Services Vault
resource "azurerm_recovery_services_vault" "vault" {
  name                = "rsv-backup-prod"
  location            = "East US"
  resource_group_name = "rg-backup"
  sku                 = "Standard"
}

# 2. Daily VM Backup Policy
resource "azurerm_backup_policy_vm" "policy" {
  name                = "policy-daily-vm-backup"
  resource_group_name = "rg-backup"
  recovery_vault_name = azurerm_recovery_services_vault.vault.name

  backup {
    frequency = "Daily"
    time      = "02:00"
  }

  retention_daily {
    count = 14 # Retains daily backups for 14 days
  }
}

# 3. Attach Policy to Azure VM
resource "azurerm_backup_protected_vm" "protected_vm" {
  resource_group_name = "rg-backup"
  recovery_vault_name = azurerm_recovery_services_vault.vault.name
  source_vm_id        = "/subscriptions/.../virtualMachines/vm-app-prod"
  backup_policy_id    = azurerm_backup_policy_vm.policy.id
}

```
### 5. How do you configure Azure Logic Apps (Consumption) with custom HTTP triggers using Terraform?
 * **The Concept:** An Azure Logic App is a visual workflow builder. In Terraform, you write the workflow JSON structure into an azurerm_logic_app_workflow and attach a trigger (like an HTTP endpoint) so external systems can kick off the automation.
 * **The Interview Answer:** Declare an azurerm_logic_app_workflow and use azurerm_logic_app_trigger_custom to define incoming triggers (e.g., JSON schema payloads received over HTTP) using valid JSON syntax.
 * **HCL Example:**
```hcl
resource "azurerm_logic_app_workflow" "workflow" {
  name                = "logic-alert-processor-prod"
  location            = "East US"
  resource_group_name = "rg-automation"
}

# Define HTTP Post Trigger receiving Webhook JSON payloads
resource "azurerm_logic_app_trigger_custom" "http_trigger" {
  name         = "When_HTTP_Request_Is_Received"
  logic_app_id = azurerm_logic_app_workflow.workflow.id

  body = jsonencode({
    type = "Request"
    kind = "Http"
    inputs = {
      schema = {
        type = "object"
        properties = {
          alertName   = { type = "string" }
          severity    = { type = "string" }
        }
      }
    }
  })
}

```
### 6. How do you configure an Azure Event Hub Schema Registry using Terraform?
 * **The Concept:** When microservices stream millions of events per second, they need to agree on data formatting (e.g., Avro or JSON schemas). The Schema Registry acts as a centralized contract validator ensuring producers send correctly structured events.
 * **The Interview Answer:** Provision an azurerm_eventhub_namespace (Standard/Premium tier) and construct an azurerm_eventhub_namespace_schema_group specifying the schema_type (e.g., Avro) and schema_compatibility rules.
 * **HCL Example:**
```hcl
resource "azurerm_eventhub_namespace" "eh_ns" {
  name                = "ehns-streaming-prod"
  location            = "East US"
  resource_group_name = "rg-data"
  sku                 = "Standard"
}

resource "azurerm_eventhub_namespace_schema_group" "schema_group" {
  name                 = "schema-group-orders"
  namespace_id         = azurerm_eventhub_namespace.eh_ns.id
  schema_compatibility = "Forward" # Enforces forward schema evolution compatibility
  schema_type          = "Avro"
}

```
### 7. How do you enforce Entra ID (Azure AD) Only Authentication on Azure SQL Database using Terraform?
 * **The Concept:** Traditional SQL servers use local usernames and passwords (like sa / sqladmin), which can be brute-forced or leaked. "Entra ID Only Auth" disables traditional password logins entirely and forces every user and application to authenticate via Microsoft Entra ID tokens or Managed Identities.
 * **The Interview Answer:** In azurerm_mssql_server, configure an azuread_administrator block specifying the admin object ID, and set azuread_authentication_only = true.
 * **HCL Example:**
```hcl
resource "azurerm_mssql_server" "sql" {
  name                         = "sql-secure-prod-001"
  resource_group_name          = "rg-data"
  location                     = "East US"
  version                      = "12.0"
  
  # Disables local SQL admin logins; enforces Microsoft Entra ID authentication exclusively
  azuread_authentication_only = true

  azuread_administrator {
    login_username              = "Entra SQL Admins"
    object_id                   = "00000000-0000-0000-0000-000000000000" # Group Object ID
    azuread_authentication_only = true
  }
}

```
## Advanced Level Questions
### 8. How do you configure Azure Route Server to dynamically peer with a Network Virtual Appliance (NVA) using Terraform?
 * **The Concept:** In a hub-and-spoke network, updating static route tables across 50 spoke subnets every time a new network opens is painful. **Azure Route Server** uses BGP (Border Gateway Protocol) to exchange routes dynamically between your firewalls/NVAs and all spokes without manual route tables.
 * **The Interview Answer:** Create a dedicated subnet named RouteServerSubnet, provision azurerm_route_server, and configure BGP peering with the firewall/NVA using azurerm_route_server_bgp_connection.
 * **HCL Example:**
```hcl
# Subnet MUST be named 'RouteServerSubnet'
resource "azurerm_subnet" "routeserver_subnet" {
  name                 = "RouteServerSubnet"
  resource_group_name  = "rg-networking-hub"
  virtual_network_name = "vnet-hub"
  address_prefixes     = ["10.0.4.0/24"]
}

# Azure Route Server Instance
resource "azurerm_route_server" "rs" {
  name                 = "routeserver-hub"
  resource_group_name  = "rg-networking-hub"
  location             = "East US"
  subnet_id            = azurerm_subnet.routeserver_subnet.id
  public_ip_address_id = "/subscriptions/.../publicIPAddresses/pip-rs"
}

# BGP Peering Connection to NVA / Firewall
resource "azurerm_route_server_bgp_connection" "nva_peer" {
  name            = "bgp-peer-nva-firewall"
  route_server_id = azurerm_route_server.rs.id
  peer_asn        = 65001        # NVA Autonomous System Number
  peer_ip         = "10.0.1.4"   # NVA Private IP Address
}

```
### 9. How do you deploy an Azure Confidential Virtual Machine (AMD SEV-SNP) using Terraform?
 * **The Concept:** Standard VMs encrypt data on disk and in transit, but memory (RAM) is unencrypted while running. **Confidential VMs** encrypt data *in memory* using hardware-enforced cryptographic boundaries (AMD SEV-SNP or Intel TDX), protecting processing data from hypervisors or cloud admins.
 * **The Interview Answer:** In azurerm_linux_virtual_machine, select a Confidential VM SKU size (e.g., Standard_DC2as_v5), enable vtpm_enabled and secure_boot_enabled, and set security_encryption_type = "DiskWithVMGuestState".
 * **HCL Example:**
```hcl
resource "azurerm_linux_virtual_machine" "confidential_vm" {
  name                = "cvm-secure-compute"
  resource_group_name = "rg-sec-compute"
  location            = "East US"
  size                = "Standard_DC2as_v5" # Confidential VM SKU
  admin_username      = "azureuser"

  # Enables Confidential Hardware Security features
  vtpm_enabled        = true
  secure_boot_enabled = true

  os_disk {
    caching                  = "ReadWrite"
    storage_account_type     = "Premium_LRS"
    security_encryption_type = "DiskWithVMGuestState" # Enforces Confidential Disk Encryption
  }

  # ... Network interface and OS image reference settings ...
}

```
### 10. How do you perform Multi-Tenant Azure Authentication across separate Entra ID Tenants using Provider Aliases?
 * **The Concept:** Imagine an MSP (Managed Service Provider) managing infrastructure for Client A (Tenant A) and Client B (Tenant B). A single Terraform script can deploy resources into both clients simultaneously using multi-tenant provider aliases.
 * **The Interview Answer:** Declare two provider "azurerm" blocks with distinct alias identifiers. Supply the specific tenant_id and subscription_id for each tenant. Bind individual resource blocks to their respective tenant provider using provider = azurerm.<alias_name>.
 * **HCL Example:**
```hcl
# Provider for Tenant A (Client Alpha)
provider "azurerm" {
  alias           = "tenant_a"
  tenant_id       = "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
  subscription_id = "11111111-1111-1111-1111-111111111111"
  features {}
}

# Provider for Tenant B (Client Beta)
provider "azurerm" {
  alias           = "tenant_b"
  tenant_id       = "bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb"
  subscription_id = "22222222-2222-2222-2222-222222222222"
  features {}
}

# Resource deployed into Tenant A
resource "azurerm_resource_group" "rg_a" {
  provider = azurerm.tenant_a
  name     = "rg-client-alpha-prod"
  location = "East US"
}

# Resource deployed into Tenant B
resource "azurerm_resource_group" "rg_b" {
  provider = azurerm.tenant_b
  name     = "rg-client-beta-prod"
  location = "West US"
}

```

---

Here is a **twentieth set** of brand-new, scenario-based Terraform and Microsoft Azure interview questions and answers. Each question follows the student-friendly format with **The Concept**, **The Interview Answer**, and an **HCL Code Example**.
## Basic Level Questions
### 1. Why should you reference resource outputs (e.g., azurerm_resource_group.rg.name) instead of hardcoding resource group names?
 * **The Concept:** If you hardcode "rg-app-prod" everywhere in your script, Terraform doesn't know which resource needs to be created first. When you reference azurerm_resource_group.rg.name, Terraform automatically builds a dependency tree and knows: *"I must finish building the Resource Group before I try to build the Virtual Network inside it."*
 * **The Interview Answer:** Referencing resource attributes creates an **implicit dependency**. It ensures Terraform builds resources in the correct order without requiring explicit depends_on blocks, prevents runtime provisioning errors, and allows dynamic parameter updates across configurations.
 * **HCL Example:**
```hcl
resource "azurerm_resource_group" "rg" {
  name     = "rg-app-prod"
  location = "East US"
}

resource "azurerm_virtual_network" "vnet" {
  name                = "vnet-app-prod"
  # Implicit dependency: Terraform automatically waits for the Resource Group to be created first
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  address_space       = ["10.0.0.0/16"]
}

```
### 2. How do you use the join() and split() functions in HCL for Azure configurations?
 * **The Concept:** Think of join() like gluing a list of words together with a comma to make a single sentence. Think of split() like cutting a sentence at every comma to turn it back into an array list.
 * **The Interview Answer:** * join(separator, list) converts a list of strings into a single delimited string (e.g., creating a comma-separated list of IP addresses for Azure Firewall rules).
   * split(separator, string) breaks a delimited string into a list of substrings (e.g., parsing a subscription ID out of an Azure Resource Manager ID string).
 * **HCL Example:**
```hcl
locals {
  ip_list = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]

  # Converts list into a single comma-separated string: "10.0.1.0/24,10.0.2.0/24,10.0.3.0/24"
  joined_ips = join(",", locals.ip_list)

  # Splits an Azure Resource ID string to extract the Resource Group name
  sample_resource_id = "/subscriptions/0000/resourceGroups/rg-data/providers/Microsoft.Storage/storageAccounts/st01"
  rg_name            = split("/", locals.sample_resource_id)[4] # Evaluates to "rg-data"
}

```
### 3. What is the difference between running terraform apply directly vs. using terraform plan -out=tfplan?
 * **The Concept:** Running terraform apply directly is like ordering food at a restaurant and letting the chef decide modifications on the fly. Saving a plan file (-out=tfplan) is like taking a snapshot of the exact order contract so that when it goes to the kitchen, nothing can change between the moment you reviewed it and the moment it's cooked.
 * **The Interview Answer:** terraform plan -out=tfplan saves the calculated execution plan into an encrypted binary file. Running terraform apply tfplan guarantees that Terraform executes **only** the exact changes that were reviewed and approved during the plan phase, eliminating race conditions or state drift between planning and applying in CI/CD pipelines.
 * **HCL Example Execution Commands:**
```bash
# Step 1: Generate and lock execution plan to a file
terraform plan -out=tfplan

# Step 2: Apply the exact saved plan file in automated pipeline
terraform apply tfplan

```
## Intermediate Level Questions
### 4. How do you configure Azure Key Vault Network ACLs (Firewall) using Terraform?
 * **The Concept:** By default, anyone on the internet can reach your Key Vault's public login endpoint. Key Vault Network ACLs act like a security gate, allowing connections **only** from authorized IP addresses or specific Azure Virtual Network subnets.
 * **The Interview Answer:** Configure the network_acls block inside azurerm_key_vault. Set default_action = "Deny", define trusted IP addresses in ip_rules, and pass authorized subnet IDs into virtual_network_subnet_ids.
 * **HCL Example:**
```hcl
resource "azurerm_key_vault" "kv" {
  name                = "kv-sec-prod-002"
  location            = "East US"
  resource_group_name = "rg-security"
  tenant_id           = "00000000-0000-0000-0000-000000000000"
  sku_name            = "standard"

  network_acls {
    default_action             = "Deny" # Blocks all unauthorized internet traffic
    bypass                     = ["AzureServices"] # Allows trusted Azure platform services
    ip_rules                   = ["203.0.113.5/32"] # Developer Office IP
    virtual_network_subnet_ids = ["/subscriptions/.../subnets/snet-app"] # Authorized Subnet
  }
}

```
### 5. How do you configure Azure Service Bus Authorization Rules (SAS Keys) in Terraform?
 * **The Concept:** You don't want your frontend app to have full administrative control over your entire Service Bus messaging system. Service Bus Authorization Rules let you generate granular Shared Access Signature (SAS) keys that grant *only* Send rights to producers and *only* Listen rights to consumers.
 * **The Interview Answer:** Use azurerm_servicebus_namespace_authorization_rule or azurerm_servicebus_topic_authorization_rule to define explicit send, listen, or manage rights. Retrieve primary connection strings using sensitive outputs for application configurations.
 * **HCL Example:**
```hcl
resource "azurerm_servicebus_namespace" "sb" {
  name                = "sb-messaging-prod"
  location            = "East US"
  resource_group_name = "rg-integration"
  sku                 = "Standard"
}

# Authorization Rule granting ONLY 'Send' permissions for Publisher apps
resource "azurerm_servicebus_namespace_authorization_rule" "publisher_rule" {
  name         = "rule-app-publisher"
  namespace_id = azurerm_servicebus_namespace.sb.id

  send   = true
  listen = false
  manage = false
}

output "publisher_connection_string" {
  value     = azurerm_servicebus_namespace_authorization_rule.publisher_rule.primary_connection_string
  sensitive = true # Protects SAS connection string in CLI logs
}

```
### 6. How do you configure Azure Virtual Machine Availability Sets vs. Availability Zones in Terraform?
 * **The Concept:** An **Availability Set** spreads VMs across racks (fault domains) inside a *single* datacenter. An **Availability Zone** spreads VMs across entirely *separate physical datacenters* miles apart within the same region.
 * **The Interview Answer:**
   * **Availability Zone:** Defined directly on the VM block using zone = "1" (or zones = ["1", "2"] for scale sets).
   * **Availability Set:** Defined by creating an azurerm_availability_set and passing its ID into availability_set_id on the VM block.
 * **HCL Example:**
```hcl
# Option A: Deploying VM into an Availability Zone (Separate Datacenter)
resource "azurerm_linux_virtual_machine" "vm_zone" {
  name                = "vm-zone-prod"
  resource_group_name = "rg-compute"
  location            = "East US"
  size                = "Standard_B2s"
  zone                = "1" # Deploys directly into Availability Zone 1

  # ... Network interface and disk settings ...
}

# Option B: Deploying VM into an Availability Set (Separate Racks inside same Datacenter)
resource "azurerm_availability_set" "avset" {
  name                        = "avset-app-prod"
  location                    = "East US"
  resource_group_name         = "rg-compute"
  platform_fault_domain_count = 2
}

resource "azurerm_linux_virtual_machine" "vm_avset" {
  name                 = "vm-avset-prod"
  resource_group_name  = "rg-compute"
  location             = "East US"
  size                 = "Standard_B2s"
  availability_set_id  = azurerm_availability_set.avset.id # Linked to Availability Set

  # ... Network interface and disk settings ...
}

```
### 7. How do you provision an Azure Cognitive Services Account (Azure OpenAI / AI Services) using Terraform?
 * **The Concept:** To build AI-driven applications on Azure, you need an Azure AI Services account endpoint. In Terraform, you define the account SKU and deploy specific AI models (like GPT-4) as deployments inside that account.
 * **The Interview Answer:** Provision azurerm_cognitive_account specifying kind = "OpenAI" or kind = "CognitiveServices". Create individual AI model deployments inside the account using azurerm_cognitive_deployment.
 * **HCL Example:**
```hcl
resource "azurerm_cognitive_account" "openai" {
  name                = "oai-enterprise-prod-001"
  location            = "East US"
  resource_group_name = "rg-ai"
  kind                = "OpenAI"
  sku_name            = "S0"
}

# Deploys a specific model instance inside the OpenAI account
resource "azurerm_cognitive_deployment" "gpt4" {
  name                 = "gpt-4-deployment"
  cognitive_account_id = azurerm_cognitive_account.openai.id

  model {
    format  = "OpenAI"
    name    = "gpt-4"
    version = "0613"
  }

  scale {
    type     = "Standard"
    capacity = 10 # Tokens-per-minute scale unit capacity
  }
}

```
## Advanced Level Questions
### 8. How do you implement Azure Management Group Hierarchies for enterprise landing zones using Terraform?
 * **The Concept:** Big enterprises don't just dump all subscriptions in one flat pile. They create a tree hierarchy of Management Groups (e.g., Root -> Platform -> Identity / Connectivity / Workloads) so policy rules and RBAC permissions inherit down the tree automatically.
 * **The Interview Answer:** Use azurerm_management_group to construct parent-child hierarchy structures. Link individual Azure subscriptions to their target management group node using subscription_ids or azurerm_management_group_subscription_association.
 * **HCL Example:**
```hcl
# Parent Management Group: Platform
resource "azurerm_management_group" "parent_platform" {
  display_name = "Platform-Management-Group"
  name         = "mg-platform"
}

# Child Management Group: Connectivity (inherits permissions from Platform)
resource "azurerm_management_group" "child_connectivity" {
  display_name               = "Connectivity-Management-Group"
  name                       = "mg-connectivity"
  parent_management_group_id = azurerm_management_group.parent_platform.id

  # Connects target Networking subscription to this child node
  subscription_ids = [
    "00000000-0000-0000-0000-000000000000"
  ]
}

```
### 9. How do you handle zero-downtime upgrades for an Azure Virtual Network Gateway (VPN Gateway) using Terraform?
 * **The Concept:** Changing a VPN Gateway SKU or IP configuration can cause a network outage if done incorrectly. By deploying the VPN Gateway in **Active-Active** mode with two public IPs and using create_before_destroy, Terraform ensures one tunnel stays active while updating the other.
 * **The Interview Answer:** Configure azurerm_virtual_network_gateway with active_active = true and provide two distinct ip_configuration blocks. Apply lifecycle { create_before_destroy = true } to guarantee that replacement gateways are fully provisioned before old gateway connections tear down.
 * **HCL Example:**
```hcl
resource "azurerm_virtual_network_gateway" "vpn_gw" {
  name                = "vnet-gw-prod"
  location            = "East US"
  resource_group_name = "rg-networking"

  type     = "Vpn"
  vpn_type = "RouteBased"
  active_active = true # Enables Active-Active Dual Tunnel High Availability
  sku      = "VpnGw2"

  ip_configuration {
    name                          = "vnetGatewayConfig1"
    public_ip_address_id          = "/subscriptions/.../publicIPAddresses/pip-vnet-gw-1"
    private_ip_address_allocation = "Dynamic"
    subnet_id                     = "/subscriptions/.../subnets/GatewaySubnet"
  }

  ip_configuration {
    name                          = "vnetGatewayConfig2"
    public_ip_address_id          = "/subscriptions/.../publicIPAddresses/pip-vnet-gw-2"
    private_ip_address_allocation = "Dynamic"
    subnet_id                     = "/subscriptions/.../subnets/GatewaySubnet"
  }

  lifecycle {
    create_before_destroy = true # Prevents VPN downtime during Gateway updates
  }
}

```
### 10. How do you write Open Policy Agent (OPA/Rego) policies to validate Terraform Azure plan JSON files in CI/CD pipelines?
 * **The Concept:** Static analysis checks your raw .tf files, but **OPA/Rego** inspects the calculated tfplan.json output *after* variables and dynamic loops have evaluated. This lets you enforce policy gates like: *"Reject any deployment where a Storage Account has public access enabled."*
 * **The Interview Answer:** Convert the binary execution plan to JSON using terraform show -json tfplan > plan.json. Run OPA policy rules against plan.json in your CI/CD pipeline stage to evaluate resource change actions (create, update) against enterprise governance policies.
 * **Rego Policy Example (policy.rego):**
```rego
package main

# Rule: Deny deployment if any newly created Storage Account allows public access
deny[msg] {
    # Iterates over planned resource changes in plan.json
    resource := input.resource_changes[_]
    resource.type == "azurerm_storage_account"
    
    # Checks if action is create or update
    resource.change.actions[_] == "create"
    
    # Condition: Public access enabled
    resource.change.after.public_network_access_enabled == true
    
    msg := sprintf("Policy Violation: Storage Account '%v' must have public_network_access_enabled set to false.", [resource.name])
}

```
```bash
# CI/CD Pipeline Execution Commands:
terraform plan -out=tfplan
terraform show -json tfplan > plan.json
opa eval --data policy.rego --input plan.json "data.main.deny"

```
