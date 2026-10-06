# Introduction

I'm a DevOps Engineer with a little over four years of experience, primarily working with AWS, Kubernetes, Terraform and GitLab CI/CD. Most of my work has been around automating infrastructure and deployment processes, improving reliability and security, and building reusable infrastructure and CI/CD patterns.

In my current role, I've been working in a large enterprise environment where we're standardizing and automating deployment processes across Windows and Linux environments. Before that, I worked extensively with AWS infrastructure, Terraform, Kubernetes and security automation.

What attracted me to this opportunity is that it's an infrastructure automation project — creating reproducible environments, automating their lifecycle and making them simple enough that users don't need DevOps intervention every time they need a test or demo environment.

# Common

## Tell me about yourself

### Who are you?

- I’m a DevOps Engineer with almost five years of experience **working on cloud-native & on-prem  platforms**, primarily on AWS and Kubernetes, with strong exposure to Terraform, GitLab CI/CD, and GitOps using Argo CD.

### What was your primary focus?

- My work has mostly focused on **building** and **operating** dev, QA & production **platforms** — not just deploying infrastructure, but making sure it’s reliable, secure, observable, and easy to operate at scale.

#### What did you do?

- Over the last few years, I’ve **deployed cloud infra**, **supported critical workloads** across multiple **environments**, **worked closely with** security teams on DevSecOps automation, and helped reduce operational toil by standardizing infrastructure as code, CI/CD workflows, and platform-level observability.

### What are you looking for?

- I’m now looking a DevOps opportunity with new challenges where **security**, **reliability**, and **platform stability** are first-class concerns — which is why this DevOps role at Intact really **caught my attention**.


## Why BD

- Different Industry (Medical tech company)
- Interesting Project (Infra automation, Software controlling baggage robots)
- Fascination + Relevant experience (Always fascinated by different industries and since my experience correlates to the requirement.)

What attracts me to BD is the combination of a large enterprise environment, cloud engineering, automation, and the opportunity to work on platforms that is integrated with the Pharma industry.

So I see myself as a great fit because I can bring both sides: hands-on cloud and DevOps engineering experience, combined with experience working within large enterprise environments with strict governance and security requirements.


## Why this DevOps role?

This role is interesting to me because it matches very closely with the type of DevOps work I enjoy doing.

What attracted me to this opportunity is that it's an infrastructure automation project — creating reproducible environments, automating their lifecycle and making them simple enough that users don't need DevOps intervention every time they need a test or demo environment. That is exactly the direction I want to continue developing in my DevOps career.


## GitHub Actions — this will probably be one of your biggest questions

My strongest hands-on CI/CD experience is with GitLab CI/CD, but I've worked extensively with the underlying concepts that GitHub Actions uses.

I've designed reusable pipelines, worked with runners, artifacts, secrets, environment-specific deployments, security scanning, approvals and deployment automation.

So I don't see the transition from GitLab CI/CD to GitHub Actions as learning CI/CD from scratch. The main learning curve is the GitHub Actions syntax, ecosystem and specific implementation patterns.

For example, I understand concepts such as workflows, jobs, steps, runners, secrets, artifacts, environments, reusable workflows and conditional execution. I’m confident I can become productive with the GitHub-specific implementation very quickly.

Brief: My primary production experience is GitLab CI/CD. I have not had the same depth of production ownership with GitHub Actions, but I understand the CI/CD architecture and I'm comfortable adapting to the platform.


## Why should we hire you?

I think I bring a good combination of hands-on cloud engineering and enterprise experience.

I have 4.5 years of DevOps experience, with strong hands-on work in AWS, Terraform, Kubernetes, CI/CD, security and automation. At EXFO, I worked extensively on AWS and Kubernetes platforms, including EKS, S3, VPC, IAM, Lambda and Terraform-based infrastructure.

In my current role at Crédit Agricole CIB, I’m working in a highly governed financial-services environment, where security, standardization, change management and reliability are very important.

I’m also someone who likes to automate repetitive work and improve existing processes rather than simply maintain them.

The main area where I would need to adapt is the specific CI/CD ecosystem, because my strongest experience is GitLab rather than GitHub Actions. But the underlying concepts are very familiar to me, and I’m confident I can transfer that experience quickly.

Overall, I think I can bring strong AWS and infrastructure automation experience while fitting well into an enterprise team and contributing proactively.


## How do you handle pressure?

**Situation:**
In my previous role at EXFO, I had situations where a CI/CD pipeline or infrastructure issue was affecting a deployment, and there was pressure to restore the environment quickly because other teams were depending on it.

**Task:**
My responsibility was to identify the root cause, restore the service or deployment as quickly and safely as possible, and keep the relevant team members informed without making the situation worse.

**Action:**
When I’m under pressure, I try to stay structured rather than immediately making changes. First, I assess the impact and prioritize what needs to be restored. Then I check the most relevant logs, pipeline information, Kubernetes or AWS status, and recent changes to narrow down the issue.

I communicate clearly with the affected team about what I know, what I’m investigating, and the expected next steps. Once I identify the issue, I apply the safest remediation, validate the result, and monitor the system afterward.

After the immediate issue is resolved, I also look at the root cause and whether we can prevent the same problem through automation, monitoring, documentation, or a change to the deployment process.

**Result:**
This approach has helped me resolve issues without making decisions based purely on pressure. It also allows me to keep stakeholders informed while focusing on restoring service first and preventing recurrence afterward.

So, overall, I handle pressure by **staying calm, prioritizing impact, communicating clearly, troubleshooting systematically, and focusing on both immediate recovery and long-term prevention.**


Brief: When I’m under pressure, I stay structured: assess the impact, prioritize, communicate, resolve, and then prevent recurrence.

## Individual Leadership contribution

- Image tag issue (Tag version wasn't changed manually), found it and automated a semantic release versions in the project.

- Security lead: Led the vulnerability management for the whole project during the releases. Created a tool that uses Trivy and Grype to detect and mitigate the vulnerabilities using SAST, IAC-SAST, Kubesec scans over the projects.

- AWS: One of the service in the exchange platform was faulty on a Friday and it was critical as one of our clients busiest day. Jumped in, assessed the impact, troubleshooted, verified the logs over elastic search and found that the DB in RDS is full and was throwing issues.

- Terraform: Initially the AWS infra for dev was not being tagged properly for the services being deployed in AWS. Thus, a lot of orphaned resources were left around. Did an audit to make a list of all the resources being used and cleared the orphaned resources. Also tagged them using the Environment, thus we know who the resource belongs to.

Situation:
In one of our AWS development environments, resources were not being consistently tagged according to the services or environments they belonged to. As a result, it became difficult to identify resource ownership, and some orphaned resources were left behind after deployments or changes.

Task:
I took the initiative to audit the AWS environment, identify which resources were actually being used, determine the ownership of those resources, and clean up resources that were no longer required.

Action:
I created an inventory of the AWS resources and mapped them back to the services and environments using them. I then identified orphaned or unused resources and coordinated their cleanup.

After the cleanup, I improved the tagging approach so that resources were consistently tagged with information such as Environment, Service/Project, and Owner. Since most of the infrastructure was managed through Terraform, I also looked at making the tags part of the Terraform configuration rather than relying on manual tagging.

A further improvement would be to enforce mandatory tags through Terraform validation or policy controls during CI/CD, so that infrastructure without the required tags cannot be deployed. We could also use AWS cost allocation tags and periodic reporting to identify resources without proper ownership and improve cost visibility.

Result:
This gave us a much cleaner AWS environment, made resource ownership easier to identify, reduced orphaned resources, and improved our ability to manage cloud costs.

More importantly, the improvement was not just cleaning up the existing problem. The goal was to make tagging part of the infrastructure provisioning process so the same issue would be less likely to happen again.