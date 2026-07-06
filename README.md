# AWS Automation
 
One-click AWS operations via GitHub Actions — no console logins, no stored credentials, no surprise bills.
 
This repo is my automation toolbox for managing personal AWS infrastructure. It started with a simple problem: a t3.small left running overnight is the most expensive way to learn about AWS billing. Now starting and stopping my EC2 instance is a button press.
 
## How It Works
 
Workflows authenticate to AWS via **GitHub OIDC federation** — GitHub proves its identity to AWS and assumes a narrowly-scoped IAM role for the duration of the run. No access keys exist anywhere: not in this repo, not in GitHub secrets, not on disk.
 
```
workflow_dispatch (manual trigger)
        │
        ▼
GitHub issues OIDC token ──► AWS STS validates trust policy ──► temporary credentials
        │                     (scoped to this exact repo)         (expire in minutes)
        ▼
aws ec2 start-instances / stop-instances
        (IAM role permits only these two actions, on one instance)
```
 
## Workflows
 
| Workflow | Trigger | What it does |
|---|---|---|
| **EC2 Control** | Manual (`workflow_dispatch`) | Start or stop a specific EC2 instance via a dropdown choice |
 
More automations will land here as needs appear (scheduled stop at night, budget-triggered shutdown, AMI backups).
 
## Usage
 
1. Go to **Actions → EC2 Control → Run workflow**
2. Pick `start` or `stop` from the dropdown
3. Run
## Security Design
 
- **OIDC over access keys** — temporary, per-run credentials; nothing long-lived to leak or rotate
- **Trust policy scoped to this repository** — no other repo (including forks) can assume the role
- **Least-privilege IAM** — the role permits exactly `ec2:StartInstances` and `ec2:StopInstances` on a single instance ARN, nothing else
- **All identifiers in GitHub secrets** — region, role ARN, and instance ID are configuration, not code
- **Manual trigger only** — `workflow_dispatch` requires write access to the repo; strangers can't press the buttons
## Related
 
This toolbox supports [DevFocus](https://github.com/OLekgetho/DevFocus) — a developer productivity platform.
 
