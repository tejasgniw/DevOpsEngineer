# Questions I expect them to ask

Prepare strong answers for these:

## Architecture

### How would you design this solution?

Before choosing the implementation, I'd clarify what an environment actually contains, how isolated each environment needs to be, how frequently environments are created and reset, how long they're expected to live, and whether sales users need to trigger the lifecycle themselves or if DevOps owns that process.

Then:

Assuming each environment is self-contained and needs to support one application version, I'd separate the solution into image creation, infrastructure provisioning, configuration/data initialization and lifecycle management.

```
                 GitHub
                    │
                    ▼
             GitHub Actions
                    │
        ┌───────────┴───────────┐
        │                       │
      Packer                 Terraform
        │                       │
        ▼                       ▼
   Golden Image             Azure / AVS (Azure VMWare solution)
                                │
                     ┌──────────┴──────────┐
                     │                     │
                  Windows VM             DB
                     │                PostgreSQL/
                     │                  MySQL
                     └──────────┬──────────┘
                                │
                          Demo Environment
```


Lifecycle:

```
CREATE → CONFIGURE → VALIDATE → READY

             ↓

           RESET

             ↓

RESTORE BASELINE → VALIDATE → READY

             ↓

           DESTROY
```


### How would you create 10–20 environments?

Almost the same above. However, in case of the infrastructure, I would use a unique workspaces for each environment. Terraform workspaces allow you to maintain separate state files within the same configuration.  They are useful when managing multiple environments (like dev, staging, prod) without duplicating code.

Workspaces work well for simple environment separation, but become unwieldy in complex multi-account setups.

### How would you make environments isolated?

- I’d isolate environments using separate network segments/subnets and security rules.
- Each environment gets its own VM, application configuration, and database.
- Secrets and permissions would be environment-specific using something like Azure Key Vault.
- Terraform + GitHub Actions would be environment-aware, ensuring an operation on Demo-07 only affects Demo-07.
- I’d choose the isolation level based on the required security, connectivity, and cost, not over-engineer it.

```
Azure / AVS
    │
    ├── Demo-01 → VM + DB
    ├── Demo-02 → VM + DB
    ├── Demo-03 → VM + DB
    └── ...Demo-20 → VM + DB
```

### How would you handle different versions of the application?

- I’d maintain a versioned golden image for each supported application version.
- Each version would have a corresponding database baseline/backup.
- Terraform would take the application version as an input and provision the correct environment.
- GitHub Actions would validate the version and orchestrate the deployment/reset.
- This keeps all four versions reproducible, isolated, and easy to maintain.

```
GitHub Actions
      │
      ├── v1 → Packer Image + DB Baseline → Demo-01
      ├── v2 → Packer Image + DB Baseline → Demo-02
      ├── v3 → Packer Image + DB Baseline → Demo-03
      └── v4 → Packer Image + DB Baseline → Demo-04
```


### How would you make the environment reproducible?

- I’d define the infrastructure entirely as Terraform code with reusable modules.
- Use Packer to create versioned, standardized Windows/application images.
- Maintain a versioned database baseline for each application version.
- GitHub Actions orchestrates the full process with validated inputs and automated health checks.
- The goal is that CREATE or RESET always converges to the same known-good state.

```
GitHub Actions
      │
      ├── Terraform → Infrastructure
      ├── Packer    → Windows/App Image
      └── DB Backup → Known-good Data
              │
              ▼
        Identical Environment
```

### How would you implement reset?

For me, the reset operation is the most important part of the design. I'd want it to be deterministic and idempotent. Whether the user runs it once or repeats it, the environment should converge to a known-good baseline. I'd also build validation into the workflow so the user gets a clear success or failure result rather than having to determine manually whether the environment is healthy.

```
RESET Request
     │
     ▼
Stop App → Reset/Restore DB → Restore Config → Start App
                                      │
                                      ▼
                              Health Checks → READY
```

### What's the difference between reset, stop/start and destroy?

- Stop/Start: Keeps the VM and data intact; simply powers the environment off/on.
- Reset: Keeps the infrastructure but restores the application + database to a known-good baseline.
- Destroy: Terraform removes the environment's VMs, networking, storage, and other infrastructure.
- I’d use Stop/Start for temporary savings, Reset for a fresh demo/test, and Destroy when the environment is no longer needed.

```
STOP/START → Power state
RESET      → Restore known-good state
DESTROY    → Remove infrastructure
```

### How would you prevent users from accidentally destroying environments?

- CI level: Keep destroy as a separate protected job; require manual approval and restrict it to Maintainers, not Developers.
- Terraform level: Use separate workspaces/state files per environment, with remote state + locking to prevent accidental cross-environment changes.
- IAM level: Apply least-privilege Azure RBAC/managed identities; the CI identity should only have the permissions required for its job.
- Multiple layers: Protected branches/environments, PR reviews, Terraform plan before apply/destroy, and audit logs.
- Defense in depth: Even if someone bypasses one control, the other layers should prevent an unintended production/demo environment destruction.

## Terraform

### How do you structure Terraform modules?

- I’d create small, reusable modules around logical components like networking, VM, and database.
- A higher-level demo-environment module composes those modules into one complete environment.
- Environment-specific values are passed through variables, rather than duplicating Terraform code.
- Keep state separate per environment and use remote state with locking.
- This gives us reusability, consistency, and easy scaling from 1 to 20 environments.

```
Terraform Root
      │
      ├── modules/
      │    ├── network/
      │    ├── windows-vm/
      │    ├── database/
      │    └── demo-environment/
      │
      └── environments/
           ├── demo-01/
           ├── demo-02/
           └── demo-03/
```

### How do you manage Terraform state?

- I’d keep Terraform state remote, using Azure Storage Blob rather than local state.
- Use separate state per environment so Demo-01 changes cannot affect Demo-02.
- Use state locking to prevent concurrent Terraform operations.
- Restrict access to state through Azure RBAC/managed identity.
- Enable versioning/backups so state can be recovered if it’s accidentally modified or corrupted.

### How do you handle secrets?

- I would never store secrets in Git, Terraform code, or plaintext pipeline variables.
- Store them in Azure Key Vault with encryption, access policies/RBAC, and auditing.
- GitHub Actions authenticates using OIDC/Managed Identity rather than long-lived credentials where possible.
- Grant each environment least-privilege access to only the secrets it needs.
- Rotate secrets regularly and avoid exposing them in logs, outputs, or Terraform state.

Github CI/CD variables, Terraform variables as sensitive but ideally Azure Key vault.

### How do you manage multiple environments?

In case of the infrastructure, I would use a unique workspaces for each environment. Terraform workspaces allow you to maintain separate state files within the same configuration.  They are useful when managing multiple environments (like dev, staging, prod) without duplicating code.

Workspaces work well for simple environment separation, but become unwieldy in complex multi-account setups.

### How do you handle Terraform drift?

Drift detection refers to the situation when the actual infrastructure state diverges from the state defined in Terraform’s configuration. This can happen when manual changes are made outside of Terraform, like updates in the cloud provider’s console or other automation tools. 

Terraform can detect drift by running terraform plan, which compares the current state from the state file with the real infrastructure. 

- If drift is detected, you should revert the manual changes to match the Terraform configuration, update the configuration to reflect the new desired state, and run terraform apply to bring the infrastructure back into alignment with the configuration.

### What happens if Terraform apply fails halfway through?

When a terraform apply fails halfway through, Terraform immediately stops processing downstream resources and does not roll back. Unlike some orchestration tools that automatically clean up on failure, Terraform embraces a "partial success" reality

- No Automatic Rollback: Terraform will not delete the infrastructure it just created.
- State File Updates: Before exiting, Terraform updates your state file with the resources that were successfully created or modified up to the point of failure. The state file represents a snapshot of this "halfway" reality.
- Execution Halts: Any resources that depend on the failed resource, or were scheduled later in the dependency graph, are skipped entirely.
- State Unlocks: Terraform safely unlocks your backend state storage so you can run subsequent commands.


## Packer

### Why Packer?

- Packer automates the creation of consistent, repeatable VM images.
- It avoids manually configuring every Windows VM and ensures all environments start from the same baseline.

### What is a golden image?

- A golden image is a tested, approved VM image containing the OS, patches, dependencies, agents, and required application components.
- It becomes the known-good starting point for new environments.

### How would you version images?

Semantic versioning

### How would you test an image before promoting it?

- Deploy a temporary VM from the image and run automated smoke tests(Smoke testing is a quick, preliminary evaluation of a software build designed to verify its basic stability and core functionality before conducting deep, rigorous testing.): OS readiness, required services, application startup, connectivity, and security checks.
- Only promote the image to the approved repository if all tests pass.

## CI/CD
- GitLab CI vs GitHub Actions?
- What is a self-hosted runner?
### How would you secure the runner?

Use Ephemeral Runners
- Destroy after each job: Configure runners to automatically spin up for a single job and terminate immediately after completion.
- Prevent persistence: Ephemeral environments ensure that any malicious modifications or artifacts left by a compromised build are wiped out.
- Run as non-root: Ensure the runner application executes under a restricted, unprivileged user account rather than root.
- Minimize installed tools: Keep the software inventory on the runner machine strictly limited to what is required for your builds.

- How would you implement manual approval?
- How would you trigger workflows through the GitHub API?

## Windows
- How comfortable are you with PowerShell?
- How would you automate Windows configuration?
### How would you troubleshoot a Windows VM that isn't responding?

Check vCenter and ESXi Host Health
- Review host alarms: Open the vSphere Client in your AVS private cloud to see if the underlying ESXi host has hardware, memory, or storage alerts.
- Verify cluster resource contention: Check if CPU or memory utilization is maxed out on the cluster level, which can freeze VM scheduling.
- Perform a graceful restart or reset: If the console is completely frozen, use vSphere to send an operating system shutdown command or issue a Reset if the guest OS is entirely unresponsive.

## Database

### How would you automate PostgreSQL/MySQL backup and restore?

- Create a version-specific known-good baseline using pg_dump / mysqldump or native snapshots depending on size and RTO/RPO (Recovery Time Objective (RTO) and Recovery Point Objective (RPO))
- Store backups securely with versioning, encryption, retention, then automate restore through GitHub Actions/PowerShell.

### How would you validate that a restore succeeded?

- Don't rely only on the restore command succeeding.
- Validate DB connectivity → expected schema/tables → known baseline data → application health check before marking the environment READY.

### How would you make database reset repeatable?

- Make the reset idempotent: regardless of the previous state, restore the same known-good baseline.
- Use a version-specific DB baseline so v1 always resets to v1 data, v2 to v2, etc.

```
RESET → Stop App → Restore Baseline → Start App → Validate → READY
```

## Security
- How would you integrate Polaris into CI/CD?
- How would you handle critical vulnerabilities?
- How would you protect credentials/secrets?
### How would you secure the VMs?

```
VM
├── Hardened Golden Image
├── Network/NSG Rules
├── Least-Privilege IAM
├── Secrets → Key Vault
└── Patching + Monitoring
```
- Start with a hardened Packer golden image: patched OS, unnecessary services disabled, security baseline applied.
- Use NSGs/firewalls to allow only required traffic; avoid unnecessary public exposure.
- Apply least-privilege identities and keep credentials/secrets in Key Vault.
- Enable EDR/Defender, vulnerability scanning, logging and monitoring.
- Automate patching and image refreshes so environments don't drift from the approved security baseline.