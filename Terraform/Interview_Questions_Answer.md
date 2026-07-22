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

Path B: Revert the Manual Fix (Reconcile Reality to Code)
If the manual change was a temporary workaround or violated security policies, keep your code as-is and run:
```bash
Bash
terraform apply
```
Terraform will automatically strip away the unauthorized manual rule and bring the security group back into compliance with your code.

</b></details>
