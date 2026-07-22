# Terraform Interview Questions:

<details>
<summary>
## Scenario 1: The Production Drift & Lockout Incident
You are an Infrastructure Engineer at a healthcare enterprise.

During an automated CI/CD pipeline deployment to the production environment, a network glitch kills the runner process right in the middle of executing terraform apply.

Ten minutes later, the incident response team alerts you to two distinct problems:

- Subsequent pipeline runs are failing immediately with an "Error acquiring the state lock" error in your remote backend.

- A database administrator manually changed an RDS security group directly via the Cloud Console during the outage to fix a    connectivity issue, introducing infrastructure drift.

## Question:

How do you safely resolve the state lock issue, and what exact steps do you take to detect, evaluate, and handle the manual security group drift without accidentally destroying or breaking production traffic?

</summary><br><b>
