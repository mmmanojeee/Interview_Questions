# Terraform Interview Questions:

<details>
<summary>
  
### Scenario 1: The Production Drift & Lockout Incident
You are an Infrastructure Engineer at a healthcare enterprise.

During an automated CI/CD pipeline deployment to the production environment, a network glitch kills the runner process right in the middle of executing terraform apply.

Ten minutes later, the incident response team alerts you to two distinct problems:

-  Subsequent pipeline runs are failing immediately with an "Error acquiring the state lock" error in your remote backend.
-  A database administrator manually changed an RDS security group directly via the Cloud Console during the outage to fix a    connectivity issue, introducing infrastructure drift.

### Question:

How do you safely resolve the state lock issue, and what exact steps do you take to detect, evaluate, and handle the manual security group drift without accidentally destroying or breaking production traffic?

</summary><br><b>

### Step 1: Resolving the State Lock

When the pipeline crashed mid-run, Terraform didn't get the chance to release the lock on your state backend (e.g., AWS DynamoDB or Terraform Cloud).

### How to fix it:

Verify no active processes are running: Check your CI/CD pipeline to ensure no hidden runner job is still actively trying to apply changes.

Retrieve the Lock ID: Read the failure logs from the pipeline. Terraform will explicitly output a Lock ID (e.g., Lock Info: ID: 52a1b184-c89d-...).

Force unlock the state: Run the force unlock command locally or through a controlled pipeline task:
``` bash
Bash
terraform force-unlock <LOCK-ID>
```
### 🔖Safety Note: Never force unlock unless you are 100% sure no other team member or process is actively running terraform apply. Dual writes can permanently corrupt state files.

### Step 2: Detecting and Handling the Drift

Now that the lock is cleared, you have a situation where the Cloud Console state (what actually exists in AWS) no longer matches the Terraform State or your Terraform Code.

### How to handle it safely:

1. Detect the Drift (Dry Run)
First, run a non-destructive plan to see exactly what Terraform sees:

``` bash
Bash
terraform plan
```
Terraform will automatically perform a refresh phase, comparing your remote state, your local .tf code, and the live cloud infrastructure. It will flag the manual security group rule as a discrepancy.

2. Evaluate the Manual Change
Speak with the DBA or incident team. Ask: "Is this manual security group rule a permanent requirement, or was it a temporary emergency fix?"

3. Take Action (Choose Path A or Path B)
Path A: Keep the Manual Fix (Reconcile Code to Reality)
If the security group rule change was necessary and needs to stay, update your .tf code to include the new security group rule. Then run terraform plan again. Once the plan shows No changes. Infrastructure matches configuration, you're synced up cleanly without touching live traffic.

**Path B**: Revert the Manual Fix (Reconcile Reality to Code)
If the manual change was a temporary workaround or violated security policies, keep your code as-is and run:
```bash
Bash
terraform apply
```
Terraform will automatically strip away the unauthorized manual rule and bring the security group back into compliance with your code.

</b></details>

<details>
  <summary>
    
### Scenario 2: The Module Refactoring Dilemma
    
Ready for the next scenario?

Your team has a massive, single-file Terraform codebase (main.tf) that manages a production Kubernetes cluster, its node groups, VPC, and database. It's becoming unmaintainable.

You are tasked with refactoring this codebase by moving the existing aws_db_instance.postgres resource out of main.tf and into a reusable local module named module.database.

### Question:

If you simply cut and paste the resource code into the new module directory and run terraform plan, what will Terraform try to do to your production database, and what specific Terraform feature or block should you use to refactor the code cleanly without causing downtime or recreation? 
  </summary><br><b>
    Here is what happens under the hood and how to handle it safely.

**What Terraform Tries to Do** (The Trap)
If you simply cut and paste the resource code from `main.tf` into `module.database` and run `terraform plan`, Terraform will try to DESTROY your production database and CREATE a new one.

**Why does this happen?**
Terraform tracks resources in its state file by their address (their exact hierarchical path in code):

`Old Address in State: aws_db_instance.postgres`

`New Address in Code: module.database.aws_db_instance.postgres`

Because Terraform looks at the state file and sees that `aws_db_instance.postgres` is missing from the root file, it schedules it for deletion. Then, seeing a brand new `module.database.aws_db_instance.postgres` block, it schedules a creation.

For a production database, this means data loss and major downtime.

**The Solution**: Refactoring with the moved Block
In modern Terraform (version 1.1+), you use a moved block directly in your HCL code to tell Terraform: "Hey, this isn't a new resource—we just renamed its address."

**Step 1**: Update Your Code
Move the resource code into your new module as planned, and then add a moved block at the root level of your code:

Terraform
moved {
  from = aws_db_instance.postgres
  to   = module.database.aws_db_instance.postgres
}
**Step 2**: Run terraform plan
When you run terraform plan, instead of seeing 1 to add, 1 to destroy, Terraform reads the moved block and outputs:
```text
Plaintext
# module.database.aws_db_instance.postgres has moved to module.database.aws_db_instance.postgres
Plan: 0 to add, 0 to change, 0 to destroy.
```
**Step 3**: Run terraform apply

Executing terraform apply updates the internal state file to point to the new address without touching the live database in AWS. Zero downtime, zero risk!

**Legacy Method Note**: Before Terraform 1.1 introduced moved blocks, engineers had to manually run CLI commands like `terraform state mv aws_db_instance.postgres module.database.aws_db_instance.postgres`. Modern declarative moved blocks are vastly preferred because they can be committed to Git and executed automatically in team CI/CD pipelines.
  </b>
</details>

<details>
  <summary>
    
### Scenario 3: The Secret Leaks & State File Exposure
Let's try one more scenario.

You are provisioning an RDS Database and a TLS Certificate. You set the database password using a variable var.db_password, which is marked as sensitive = true.

When a developer runs `terraform apply`, Terraform outputs db_password = (sensitive value) in the terminal, keeping it hidden from the logs.

**Question:**

Even though the variable is marked `sensitive = true` and hidden from terminal outputs, is the database password stored in plaintext inside the .tfstate file? If yes, how do you secure this credential in production?

  </summary><br><b>
    
Here is the straightforward truth about how Terraform handles sensitive data.

1. **Is the secret in plaintext in (.tfstate)?**
 YES, absolutely.

Marking a variable or output as `sensitive = true` only hides it from the terminal console and CI/CD logs. It does NOT encrypt the data inside the `.tfstate` file.

If someone reads your `terraform.tfstate` JSON file, they will see the database password, private keys, API tokens, and connection strings written in clear, unencrypted text.

2. **How to Secure Credentials in Production**
Because the state file will naturally contain secrets, production security relies on protecting access to the state file and avoiding hardcoded secrets in Terraform code.

**Strategy A:** Secure the State File at Rest

*Remote Encrypted Backends*: Store state in a remote backend like AWS S3 with SSE-KMS encryption enabled at rest.

*Strict IAM Policies*: Limit access to the S3 bucket using strict IAM policies, ensuring only the CI/CD service account (and minimal authorized personnel) can read the bucket.

Enable Bucket Versioning: Keep versioning turned on in S3 so you can rollback if state gets corrupted.

**Strategy B:** Avoid Passing Raw Secrets into Terraform
Instead of generating or passing passwords into Terraform variables, delegate secret management to a dedicated secret service:

**Auto-generate Secrets in AWS Secrets Manager / HashiCorp Vault:**
Have Terraform provision a secret container in AWS Secrets Manager, but let Secrets Manager auto-generate a random password.

**Use Dynamic Secrets via HashiCorp Vault Provider:**
Use the Vault provider to fetch short-lived dynamic database credentials at runtime, so static passwords aren't stored long-term in state.

Use Identity-Based Access (IAM / OIDC):
Where possible, avoid passwords entirely. Use IAM roles, IAM database authentication, or OIDC tokens so resources communicate using temporary, short-lived tokens instead of permanent passwords.
  </b>
</details>

<details>
  <summary>
    
### Scenario 4 (Azure Edition): The List Iteration & Resource Destruction Risk

You are tasked with deploying a set of Azure Storage Accounts for 4 different environments: dev, staging, qa, and prod.

Instead of copying and pasting the azurerm_storage_account block 4 times, you decide to define a list variable in `variables.tf:`

``` Terraform

variable "environments" {
  type    = list(string)
  default = ["dev", "staging", "qa", "prod"]
}
```
To create these Storage Accounts, you need to iterate over this list in your HCL code.

**Question:**

Should you use count (referencing count.index) or `for_each`?

What unexpected behavior occurs if you used count and a team member removes "staging" from the middle of that variable list next week?

  </summary>
  <br>
  <b>

Here is the exact explanation of why `for_each` is the industry standard and what goes wrong under the hood when using count.

The Danger of Using `count` (Index-Based Tracking) When you use count, Terraform tracks resources internally in the state file using numeric array indices **(0, 1, 2, 3)**
- `azurerm_storage_account.st[0]` $\rightarrow$ dev
- `azurerm_storage_account.st[1]` $\rightarrow$ staging
- `azurerm_storage_account.st[2]` $\rightarrow$ qa
- `azurerm_storage_account.st[3]` $\rightarrow$ prod

  What happens when "staging" is removed? If your variable list changes to ["dev", "qa", "prod"]:
  - `azurerm_storage_account.st[0]` is still dev (No change).
  - `azurerm_storage_account.st[1]` is now qa! (Terraform compares index 1 in state—which was staging—to index 1 in code—which is now qa. It sees a name change and schedules staging to be destroyed and replaced with qa).
  - `azurerm_storage_account.st[2]` is now prod! (Terraform destroys the old qa at index 2 and replaces it with prod).
  - `azurerm_storage_account.st[3]` no longer exists in code, so Terraform destroys the existing production Storage Account.
    
    **The Result:** Removing an item from the middle of a count list causes a cascading destruction and recreation of all subsequent resources, resulting in catastrophic data loss in production!Why `for_each` Solves This (Key-Based Tracking) When you convert your list into a `set/map` and use `for_each`, Terraform tracks resources by their explicit key string rather than an index number:

  ``` Terraform
  resource "azurerm_storage_account" "st" {
  for_each             = toset(var.environments)
  name                 = "st${each.key}appdata2026"
  resource_group_name  = azurerm_resource_group.rg.name
  location             = azurerm_resource_group.rg.location
  account_tier         = "Standard"
  account_replication_type = "LRS"
  }
```

Terraform addresses these resources in the state file as:

- azurerm_storage_account.st["dev"]
- azurerm_storage_account.st["staging"]
- azurerm_storage_account.st["qa"]
- azurerm_storage_account.st["prod"]

If you remove "staging" from the list, Terraform looks at the keys in state versus code. It sees that ["dev"], ["qa"], and ["prod"] are completely untouched. It only deletes azurerm_storage_account.st["staging"], leaving all other environments completely safe.

  </b>
</details>

