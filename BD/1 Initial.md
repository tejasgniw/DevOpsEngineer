# Your preparation strategy

I’d prepare in 5 layers, in this order:

- Understand the BD architecture/use case
- Translate AWS → Azure/AVS
- Build a small POC
- Prepare the Windows/PowerShell + DB + GitHub Actions gaps
- Practice architecture + behavioral answers

## He probably doesn't need you to explain what terraform plan does. He'll be evaluating:

- Can you understand the business problem?
- Can you design a solution that is simple, reliable and maintainable?
- Do you understand automation trade-offs?
- Can you operate in a Windows/Azure environment despite your AWS background?
- Can you communicate with technical and non-technical stakeholders?
- Do you think about security, reliability, cost and operational ownership?
- Can you take an ambiguous problem and turn it into an implementation plan?

## Preparation ratio

| Area                                    | Prep time |
| --------------------------------------- | --------: |
| BD project / architecture               |   **30%** |
| Your experience + stories               |   **25%** |
| Terraform / Packer / CI/CD architecture |   **15%** |
| Azure / AVS conceptual knowledge        |   **10%** |
| Windows / PowerShell / DB               |   **10%** |
| Security / AI / GitHub API              |    **5%** |
| Questions for director                  |    **5%** |


## 45 minute interview

```
0–5 min
Introduction / background

5–15 min
Your experience + projects

15–30 min
Technical / architecture discussion

30–38 min
Problem-solving / behavioral / scenarios

38–45 min
Your questions
```


## 1. Your opening needs to be extremely strong

I'm a DevOps Engineer with a little over four years of experience, primarily working with AWS, Kubernetes, Terraform and GitLab CI/CD. Most of my work has been around automating infrastructure and deployment processes, improving reliability and security, and building reusable infrastructure and CI/CD patterns.

In my current role, I've been working in a large enterprise environment where we're standardizing and automating deployment processes across Windows and Linux environments. Before that, I worked extensively with AWS infrastructure, Terraform, Kubernetes and security automation.

What attracted me to this opportunity is that it's an infrastructure automation project — creating reproducible environments, automating their lifecycle and making them simple enough that users don't need DevOps intervention every time they need a test or demo environment.


## 2. Your biggest advantage is actually Terraform + automation

### Tell me about your Terraform experience.

I've used Terraform beyond just provisioning individual resources. I've worked with reusable modules, remote state, environment-specific configurations and CI/CD-driven deployments. One thing I've learned is that the challenge with Terraform isn't really creating infrastructure — it's designing it so that multiple engineers and environments can use it safely and consistently.

Then give a concrete example from EXFO.


## 3. Expect him to challenge your architecture

### You have 20 demo environments. How would you design this?

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

## 4. The word I'd emphasize: idempotency (Idempotency is the property of an operation that means it can be applied multiple times without changing the result beyond the initial application.)

They don't just want infrastructure provisioning. Someone who isn't a DevOps engineer should be able to operate it safely.

### Idempotency

If someone clicks:

```
RESET
RESET
RESET
```

the environment should still end up in the same known-good state.

Not:

```
RESET
  ↓
some weird partial state
  ↓
DevOps engineer required
```

CREATE:
Image → VM → DB Restore → Configure → Validate → READY

RESET:
Stop App → DB Restore → Config Reset → Start App → Validate → READY

For me, the reset operation is the most important part of the design. I'd want it to be deterministic and idempotent. Whether the user runs it once or repeats it, the environment should converge to a known-good baseline. I'd also build validation into the workflow so the user gets a clear success or failure result rather than having to determine manually whether the environment is healthy.


## 5. Be ready for the obvious challenge:

### Why do we need Terraform AND Packer?

I'd separate image creation from infrastructure provisioning. Packer creates and versions the golden Windows image with the required OS-level dependencies and application prerequisites. Terraform then provisions the actual environment using that image. That avoids reinstalling and configuring everything from scratch every time we create an environment.

### Why not just use PowerShell?

PowerShell can absolutely handle configuration, but I wouldn't want the entire environment lifecycle to depend on imperative scripts. I'd use PowerShell where it makes sense for Windows configuration and application-specific operations, while Terraform handles infrastructure state and Packer handles image creation.


## 6. Expect an AWS → Azure question

### You've mainly worked with AWS. How comfortable are you with Azure?

AWS has definitely been my primary production cloud, but I don't see the core infrastructure concepts as a major gap. I've already worked extensively with Terraform, networking, IAM, compute, databases, security and CI/CD. The Azure-specific services and AVS architecture are what I'd need to learn, rather than the underlying infrastructure concepts.

For AVS specifically, I understand that we're dealing with a VMware environment hosted within Azure, so I'd expect the implementation to involve both Azure networking/integration and VMware concepts. I wouldn't claim production AVS experience that I don't have, but I would be comfortable ramping up on that layer.


## 7. Your GitLab experience is another obvious comparison

### You use GitLab. We use GitHub Actions. How transferable is that?

Very transferable. The syntax and ecosystem are different, but the CI/CD concepts are the same — workflows, runners, secrets, artifacts, approvals, environment-specific deployments and pipeline controls. I've also worked with GitLab runners and infrastructure automation, so the runner architecture isn't new to me.

I'd probably spend more time understanding how your organization has structured GitHub Actions and self-hosted runners than learning CI/CD concepts from scratch.


## 8. Prepare for a “what would you do first?” question

### You join us tomorrow. What's the first thing you would do?

I'd first understand the current manual process end-to-end. I'd want to see how an environment is currently created, what makes an environment different between the four application versions, how databases are initialized, what parts are currently manual, and where failures typically occur.

From there I'd identify the stable pieces that can become reusable images/modules and build a small vertical slice — probably one environment from creation through validation — before scaling it to 10 or 20 environments.


## 9. He may test whether you over-engineer

You would be tempted to design:

```
Azure
+
AVS
+
Terraform
+
Packer
+
GitHub Actions
+
API Gateway
+
Lambda
+
Kubernetes
+
ArgoCD
+
Kafka
+
...
```

I'd avoid introducing additional platform components unless the requirements justify them. The goal is to make the demo environment lifecycle simple and reliable, not to build another platform that DevOps has to maintain.


## 10. Have one strong story for each category

### Story 1 — Automation

Your GitLab CI/CD work at CACIB.

Manual → standardized → automated.

### Story 2 — Terraform / infrastructure complexity

Your EXFO Terraform experience.

Reusable infrastructure + multiple environments + state management.

Your Terraform state recovery experience is particularly useful.

###  Story 3 — Security

Your IaC/SAST/container/dependency scanning + Trivy work.

Security integrated into pipeline rather than manual security review.

### Story 4 — Reliability / incident

Use your Terraform state issue or another production problem.

Problem → investigation → recovery → improvement.

### Story 5 — Ambiguity / improvement

Your AWS resource-tagging/orphaned-resource example is useful.

Identified an operational gap → audited → cleaned up → implemented ownership/tagging improvement.


## 11. The director may care about your engineering philosophy

### What does good DevOps mean to you?

For me, good DevOps is reducing the amount of manual operational work while making the system safer and more predictable. It's not just automation for the sake of automation. I look at repeatability, observability, security, failure handling and how easy it is for another engineer to operate the system.

### What makes automation reliable?

Idempotency, clear ownership, validation, error handling, good logging and having a known rollback or recovery path. I also try to make failures explicit rather than allowing a pipeline to report success when the environment is actually only partially configured.

### What would you automate first?

The repetitive and deterministic parts of the process. I'd first understand where people are spending time and where human error is occurring, then automate those parts while keeping appropriate approval points for destructive operations.


## 12. Ask the Director questions that sound like a future teammate

### Question 1 — Current pain

Where is the biggest pain in the current process today — environment provisioning, application configuration, database reset, or maintaining the different application versions?

### Question 2 — Definition of success

If I joined and we looked back six months later, what would make you say this project was a success?

### Question 3 — Architecture

How much of the existing environment is already automated today, and where do you see the biggest gap that this person would own?

### Question 4 — Team

How is the responsibility split between the DevOps team, application team and the people consuming these demo environments?

### Question 5 — Technical direction

Is the long-term goal primarily to automate the current VMware/AVS model, or do you see the architecture eventually moving toward a different cloud-native approach?


## 13. One thing I would NOT do

Don't spend your preparation time trying to become: “Azure DevOps Engineer with 5 years of AVS experience.”

You aren't. And a technically experienced Director will probably detect that very quickly.

Instead, become:

“A strong DevOps engineer who understands the problem deeply, has proven Terraform/CI/CD/cloud automation experience, and can transfer those skills into Azure/AVS.” That's believable.


## Your final positioning

He doesn't know everything about AVS yet, but he understands infrastructure automation, Terraform, CI/CD, security and reliability. He thinks about the actual business problem, not just tools. He can probably ramp up on our Azure/Windows environment.

That's the win.

And given your existing AWS + Terraform + GitLab experience, that is a much more realistic target than trying to cram Azure into your head before Tuesday.