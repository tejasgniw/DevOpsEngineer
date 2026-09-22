You already have a strong match on AWS, Terraform, Kubernetes/EKS, CI/CD, enterprise governance/security, Python/Bash, monitoring, production troubleshooting, and multi-cloud. The biggest risks are GitHub Actions, depth of Python, AWS Lambda depth, AI/data platforms, SLO/SLI, cost optimization, and demonstrating senior-level proactivity/communication.

The objective is not to convince them that you have used every technology. It is to make Cynthia think:

"He has the DevOps fundamentals and enterprise experience we need. He understands AWS and Terraform, can automate, understands security/governance, communicates well, takes ownership, and can quickly adapt to GitHub Actions and our AI/data environment."

# Intact Interview — Action Plan

## 1. Your Priority Matrix

Don't spend equal time on everything.

| Priority | Area                           | Your Position                      | Preparation                 |
| -------- | ------------------------------ | ---------------------------------- | --------------------------- |
| 🔴 1     | AWS                            | **Very strong**                    | Deep preparation            |
| 🔴 2     | Terraform                      | **Very strong**                    | Deep preparation            |
| 🔴 3     | Python                         | **Good, but not primary language** | Prepare practical questions |
| 🔴 4     | GitHub Actions                 | **Gap vs GitLab**                  | **High priority**           |
| 🔴 5     | Enterprise governance/security | **Very strong**                    | Prepare stories             |
| 🔴 6     | Scenario/behavioral            | **Critical**                       | Prepare 6–8 stories         |
| 🟠 7     | AWS Lambda                     | **Real experience**                | Refresh deeply              |
| 🟠 8     | Kubernetes/EKS                 | **Very strong**                    | Refresh troubleshooting     |
| 🟠 9     | Monitoring/reliability         | **Good**                           | Prepare SLI/SLO/KPI answers |
| 🟠 10    | Cost optimization              | **Some experience**                | Prepare concrete examples   |
| 🟡 11    | GCP                            | **Good exposure**                  | Don't over-invest           |
| 🟡 12    | AI/ML platforms                | **Limited direct experience**      | Learn DevOps perspective    |
| 🟡 13    | Databricks/Snowflake           | **Not core experience**            | Understand concepts only    |
| 🟡 14    | Mentoring                      | Need to position                   | Prepare examples            |



## 2. Your Biggest Interview Advantage

Your resume has something that is very relevant to this role:

Enterprise + Cloud + DevOps + Security

At CACIB you have:

Financial-services environment
Enterprise governance
Security/quality controls
CI/CD standardization
Linux/Windows
Application deployments
GitLab infrastructure
Production-oriented processes

At EXFO you have:

AWS
EKS
Lambda
Terraform
S3
VPC
IAM
Python
Kubernetes
GitOps
CI/CD
Security scanning
Monitoring
Multi-cloud
Production troubleshooting

That combination should become the central theme of your interview.


## 3. Prepare Your 90-Second Introduction

Do not start by reading your resume.

Your introduction should position you specifically for this role.

Something like:

"I'm a DevOps Engineer with over four years of experience working primarily with AWS, Kubernetes, Terraform and CI/CD automation across enterprise and cloud-native environments.

At EXFO, I worked extensively on AWS infrastructure, including EKS, S3, VPC, IAM, Lambda and Terraform, while also managing Kubernetes, GitOps with Argo CD, CI/CD and security automation.

More recently at Crédit Agricole CIB, I've been working in a highly governed financial-services environment, where I've been focused on CI/CD standardization, automation, security controls and supporting both Linux and Windows environments.

What attracted me to this role is the combination of AWS cloud engineering, infrastructure automation, CI/CD and reliability within an enterprise environment. I also see an opportunity to bring my existing GitLab experience and adapt those CI/CD practices to GitHub Actions."

Don't memorize every word.

Memorize the structure.


## 4. AWS — Your #1 Technical Preparation

This is where I'd spend the most technical preparation time.

They specifically told you:

AWS > GCP

Be prepared to discuss:

### Core services
- Lambda
- S3
- VPC
- EC2
- IAM
- CloudWatch
- EKS
- Load Balancing
- AWS Backup
- Route 53
- KMS

### Especially Lambda
Because Lambda appears explicitly in the mandate, prepare:

- What Lambda is
- Event-driven architecture
- Invocation types
- Synchronous vs asynchronous
- Environment variables
- IAM execution roles
- VPC integration
- Cold starts
- Timeout/memory
- Concurrency
- Reserved concurrency
- Provisioned concurrency
- Logging with CloudWatch
- Monitoring
- Error handling
- Retries
- Dead-letter queues
- Deployment/versioning
- Layers
- Security
- Terraform-managed Lambda

### Expect scenario questions

For example:

**"A Lambda suddenly starts timing out in production. How would you troubleshoot it?"**

You should be able to answer systematically:

Impact → CloudWatch → duration → logs → dependencies → networking → IAM → concurrency → downstream service → recent changes → mitigation → root cause → prevention


## 5. Terraform — Prepare Like a Senior

You already have excellent material here.

Be prepared to explain your real experience with:

### Your strongest examples

- Reusable Terraform modules
- AWS/Azure/GCP
- Multi-account
- Multi-environment
- Remote state
- S3 backend
- GitLab-managed state
- State corruption/recovery
- TFLint
- CI validation
- Terraform security scanning
- Infrastructure deployments through CI/CD

### Questions you should expect

- How do you structure Terraform for multiple environments?
- How do you manage Terraform state?
- What happens if Terraform state is corrupted?
- How do you prevent two pipelines from modifying the same infrastructure?
- How do you secure Terraform?
- How do you manage secrets?
- How do you design reusable modules?
- How do you handle changes to production infrastructure?
- How do you detect drift?

You have real stories for several of these. Use them.


## 6. GitHub Actions — Your Most Important Gap

This is probably the area where you need the most focused preparation.

Do not try to hide that GitLab is your stronger experience.

Instead, understand the conceptual mapping.

- GitLab	GitHub Actions
- .gitlab-ci.yml	Workflow YAML
- Pipeline	Workflow
- Job	Job
- Stage	Job dependencies / workflow structure
- Runner	Runner
- Variables	Variables / secrets
- Artifacts	Artifacts
- Environments	Environments
- Rules	if / event conditions
- GitLab Registry	GitHub Container Registry
- Merge Request	Pull Request
- Protected branches	Branch protection
- CI/CD variables	GitHub Secrets / Variables

You need to be able to say:

**"My strongest hands-on experience is GitLab CI/CD, but I've worked deeply with CI/CD architecture, runners, reusable templates, artifacts, secrets, environment promotion, security scanning and deployment automation. Those concepts transfer directly to GitHub Actions, and I would expect the main learning curve to be the GitHub-specific syntax and ecosystem rather than CI/CD itself."**

That is a much stronger answer than pretending you're an expert in GitHub Actions.


## 7. Python — Don't Over-Present Yourself

Your resume gives you legitimate Python experience:

- Behave
- CI/CD
- Automation
- Security tooling
- Remote operations
- E2E testing

So don't say:

"I'm an expert Python developer."

You don't need to.

Instead:

**"I primarily use Python as an automation and DevOps scripting language rather than as my primary application-development language."**

Be prepared for practical questions:

- Lists/dictionaries
- Functions
- Exceptions
- File handling
- JSON/YAML
- API calls
- requests
- subprocess
- Environment variables
- Logging
- Basic classes
- Iteration
- Error handling
- Writing a small automation script
- Practice 3 scripts

You should be able to write quickly:

Read JSON and extract values.
Call an API and process the response.
Execute a system command and handle errors.


## 8. Kubernetes/EKS — Use This as a Strength

Don't let the interview become exclusively AWS Lambda.

You have very strong Kubernetes experience.

### Prepare production scenarios:

#### Pod failure
- Pod is CrashLoopBackOff. What do you do?
- Node failure
- A node becomes NotReady.
#### Networking
- Application can't communicate with another service.
- Ingress
- Service is healthy but inaccessible externally.

#### Deployment
- New deployment causes errors.

#### Resource issue
- Pods are being OOMKilled.

#### Security
- How do you secure Kubernetes?

You can bring in:

- RBAC
- Secrets
- Network policies
- Istio/mTLS
- Ingress
- Monitoring
- Logs
- Resource limits
- Probes
- Kubernetes events
- Prometheus/Grafana


## 9. Enterprise Governance — Prepare 2 Strong Stories

This is extremely important based on your recruiter feedback.

Prepare one story from CACIB and one from EXFO.

### CACIB story

Focus on:

**Manual process → enterprise transformation → standardized CI/CD → security/quality gates → controlled deployment → traceability.**

This directly matches Intact's enterprise environment.

### EXFO story

Focus on:

***Cloud infrastructure → security requirements → Terraform → CI/CD → automated security scanning → controlled deployment.***

Your answer should demonstrate:

Speed does not replace governance.

A good DevOps engineer in an enterprise environment automates within the organization's controls.


## 10. Prepare Your "Pressure" Story

You were specifically told they may ask:

**"How do you handle pressure?"**

Have one strong incident ready.

Use:

**Context → Impact → Prioritization → Communication → Technical action → Resolution → Prevention**

Do not answer:

"I work well under pressure."

Instead tell a real story.

**Your Kubernetes/CI/CD/production troubleshooting experience gives you several possibilities.**


## 11. Prepare 6 STAR Stories

I would prepare these six stories tonight.

### Story 1 — Proactivity

Problem you identified yourself → improvement you initiated → result

### Story 2 — Production incident

Production problem → troubleshooting → mitigation → resolution

###  Story 3 — Security

Your Trivy/Grype/Kubesec work.

###  Story 4 — Terraform

Your multi-account/multi-cloud Terraform work or state recovery.

### Story 5 — Enterprise governance

Your CACIB CI/CD standardization/security/quality work.

### Story 6 — Difficult situation/teamwork

A situation involving:

- Another team
- Conflicting priorities
- Pressure
- Communication
- Finding a solution

***For each story, write 5 bullets only:***

- Situation
- Task
- Action 1
- Action 2
- Result

Don't memorize paragraphs.


## 12. Prepare for the AI/Data Part

This is not where I'd spend most of your preparation time.

The role says AI Engineering, but the responsibilities are largely:

Cloud infrastructure
CI/CD
Terraform
Monitoring
Security
Deployment
Reliability
Platform support

You don't need to become an ML engineer in two days.

Understand the DevOps side:

AI/Data Team
     ↓
Data / Model Pipeline
     ↓
Build / Test
     ↓
CI/CD
     ↓
Infrastructure
     ↓
Deployment
     ↓
AWS/GCP
     ↓
Monitoring
     ↓
Production

Understand at a high level:

What Databricks is
What Snowflake is
What an ML pipeline is
Model deployment
Data pipelines
Infrastructure supporting AI workloads
Monitoring
Security
Cost management

If asked about lack of direct Databricks/Snowflake experience:

**"I haven't been the primary engineer administering Databricks or Snowflake, but I've worked extensively on the infrastructure, CI/CD, security, automation and Kubernetes side of platforms. I'm comfortable working with application and data teams and supporting the underlying platform. I would expect the platform-specific concepts to be something I can learn quickly."**

## 13. SLO / SLI / KPI — Prepare This

This is explicitly in the requirements.

Know these three:

### SLI

What you measure.

Example:    Request success rate = successful requests / total requests.

### SLO

The target.

Example:    99.9% successful requests.

### KPI

Business/operational performance indicator.

Example:    Deployment success rate, MTTR, infrastructure cost.

Also know:

- Availability
- Latency
- Error rate
- MTTR
- MTTD
- Deployment frequency
- Change failure rate

You don't need to pretend you've built an SRE program from scratch.

**Explain how you have monitored and improved reliability, then connect it to SLI/SLO concepts.**


## 14. Cost Optimization

**Prepare 2–3 concrete examples from your experience.**

You already have:

- GitLab OCI cleanup
- Storage cleanup
- Terraform resource management
- Cloud infrastructure management

Potential AWS examples:

- Right-sizing
- Unused resources
- S3 lifecycle policies
- EBS cleanup
- EKS resource optimization
- NAT Gateway considerations
- Auto Scaling
- Reserved/committed capacity
- CloudWatch cost considerations

The key is:

**Cost optimization without compromising reliability or security.**


## 15. Mentoring / Senior-Level Behavior

The role says:

**"Experience mentoring engineers."**

If your formal mentoring experience is limited, don't invent it.

Talk about:

- Creating reusable templates
- Establishing standards
- Helping developers troubleshoot pipelines
- Improving developer experience
- Documenting processes
- Sharing knowledge
- Helping teams adopt security practices

This is still technical leadership through enablement.


## 16. Cynthia's Presentation Is a Major Opportunity

This is one of the most important pieces of advice you received.

When Cynthia speaks:

Don't just listen.

### Create a mental table:

**What Cynthia says	My matching experience**
- AWS challenge	EXFO AWS
- Lambda	EXFO Lambda
- Terraform	EXFO Terraform
- CI/CD	EXFO/CACIB
- GitHub Actions	GitLab → transferable
- Security	CACIB/EXFO
- Governance	CACIB
- Reliability	Prometheus/Grafana/K8s
- AI/Data	Data warehouse/CACIB + platform experience
- Cost	Registry/storage/resource optimization

Then use those connections during your answers.


## 17. Questions You Should Ask Cynthia

Have 3 prepared, but ask based on what she says.

- **"What are the biggest technical challenges the team is currently trying to solve?"**

- **"What would success look like for the person joining the team in the first three to six months?"**

- **"You mentioned [specific thing she discussed]. Could you tell me a little more about how the team currently handles that?"**

**That third one is particularly powerful because it proves you actually listened to her presentation.**


## 18. Your Interview Answer Formula

For technical questions:

**Direct answer → real example → technical details → result**

Example:

"Yes, I've worked extensively with Terraform. At EXFO, I designed reusable Terraform modules for AWS, Azure and GCP..."

Then explain.

For behavioral questions:

Context → Task → Action → Result

### For scenario questions:

Assess → Prioritize → Communicate → Mitigate → Resolve → Prevent (APCMRP)

For technology gaps:

Be honest → show transferable knowledge → demonstrate learning ability


## 19. What NOT to Do

- ❌ Don't say:

"I haven't used GitHub Actions."

Instead:

**"My strongest experience is GitLab CI/CD, but I've worked extensively with the underlying CI/CD architecture and automation patterns, so I can transfer that experience to GitHub Actions."**

- ❌ Don't pretend to be an AI/ML engineer.

You aren't.

Position yourself as:

DevOps/platform engineer supporting AI and data teams.

- ❌ Don't overemphasize GCP.

They explicitly told you AWS is more important.

- ❌ Don't give 10-minute answers.

Answer the question first. Then provide the example.

- ❌ Don't dump technology names.

Don't say:

"I used Terraform, Kubernetes, AWS, Docker, Helm, ArgoCD, Istio...". Instead tell a story.

- ❌ Don't forget soft skills.

They explicitly told you soft skills are very important.


## 20. Your 48-Hour Preparation Plan

If you have roughly two days, I would do this:

Day 1 — Technical
Session 1 — AWS

2 hours

Focus:

Lambda
S3
VPC
IAM
CloudWatch
EKS
Backup
Security
Session 2 — Terraform

1 hour

Review:

Modules
State
Backend
State locking
Variables
Workspaces/environments
CI/CD
Security
Drift
Session 3 — GitHub Actions

1.5 hours

Learn:

Workflow
Jobs
Steps
Actions
Runners
Secrets
Environments
Artifacts
Conditions
Matrix
Reusable workflows
Deployment

Focus particularly on:

GitLab → GitHub Actions mapping

Session 4 — Python

1 hour

Practice small automation scripts.

Day 2 — Interview
Session 1 — Your Stories

1.5 hours

Prepare the six STAR stories.

Session 2 — Behavioral

1 hour

Practice:

Tell me about yourself.
Why Intact?
Why this role?
Tell me about a difficult situation.
How do you handle pressure?
Tell me about a production incident.
Tell me about a time you were proactive.
Tell me about a disagreement.
Tell me about a failure.
Tell me about a security issue.
Tell me about a time you improved something.
Session 3 — Scenario Questions

1.5 hours

Practice:

Lambda failure
Terraform failure
CI/CD failure
Production deployment failure
Kubernetes failure
Security vulnerability
Cloud outage
High AWS cost
Failed rollback
Developer wants to bypass governance
Session 4 — Intact Research

30–45 minutes

Know:

What Intact does
Its insurance business
Its technology/cloud direction
Why the role interests you
Why you want to work with the team
Session 5 — Mock Interview

45–60 minutes

Do it out loud, not silently.


## 21. The Night Before

Don't learn new technologies.

Review only:

Technical
AWS Lambda
Terraform
GitHub Actions
Python
EKS
CI/CD
Security
SLO/SLI
Stories
Proactive
Production incident
Security
Terraform
Enterprise governance
Team conflict/pressure
Interview
90-second introduction
Why Intact
Why this role
3 questions for Cynthia


## 22. During the Actual Interview

Your mindset should be:

Cynthia presents

Listen → Take notes → Identify problems

↓

You present

Context → Relevant experience → Result

↓

Technical questions

Answer directly → Explain → Give real example

↓

Scenario

Assess → Communicate → Act → Resolve → Prevent

↓

Your questions

Ask 2–3 thoughtful questions based on her presentation


## 23. Your Core Positioning

If I were preparing you for this interview, I would want every major answer to reinforce these five things:

1. Strong AWS experience
2. Strong Terraform/IaC experience
3. Enterprise governance + security experience
4. Strong CI/CD + automation + reliability experience
5. Proactive, adaptable team player

And your GitLab → GitHub Actions gap becomes:

"The platform is different, but the engineering concepts are familiar."

Your AI/data gap becomes:

"I'm not an ML engineer, but I have the cloud/platform/DevOps expertise required to support AI and data teams."

Your GCP position becomes:

"I have multi-cloud exposure, but AWS is where I have the strongest hands-on experience."

That is an honest and very defensible positioning based on the resume you provided.

One Final Thing: Don't Underestimate Your CACIB Experience

For this particular Intact role, your April 2026–Present CACIB experience is extremely useful.

Don't present it as merely:

"I build GitLab pipelines."

Present it as:

**"I'm currently working in a regulated financial-services environment where automation has to coexist with security, governance, quality controls, traceability, and structured deployment processes."**

That directly answers one of the things Randstad told you Intact cares about:

large enterprise + governance + security + adaptability.

Then combine that with your EXFO experience:

CACIB = enterprise governance/security/process
EXFO = AWS/cloud/Kubernetes/Terraform/automation

That combination should be the backbone of your interview.