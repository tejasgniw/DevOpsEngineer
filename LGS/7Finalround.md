# Timing strategy

| Time      | What happens                     |
| --------- | -------------------------------- |
| 0-5 min   | introduction                     |
| 5-15 min  | architecture discussion          |
| 15-25 min | operational / scenario questions |
| 25-30 min | your questions                   |


# Summary
For a 30-minute final round with an Operations IT Director (15 years cloud architect), the goal is to confirm:

- You can design systems, not just run tools
- You understand production operations
- You communicate clearly with clients
- You think about reliability, cost, and security
- You fit the team culture

This round is typically architecture + operational mindset + scenario discussion.

- He will look for signals of:

✔ Architecture thinking
✔ Incident handling
✔ Cost optimization
✔ Security awareness
✔ Communication with dev teams
✔ Decision making


# Strategy (Directors love impact statements.)

Structure every answer like this:

1️⃣ Context
2️⃣ Decision
3️⃣ Tools used
4️⃣ Result

e.g. In one of our environments we had an EKS cluster running microservices. Deployment time was slow. I redesigned the pipeline using GitLab CI with Terraform modules and Helm charts. This reduced deployment time from 20 minutes to 5 minutes and improved reliability.

e.g. As a part of security handling initiative, handling the vulnerability reports of all the microservices was clumsy and slow. I quickly designed a tool on python, and implemented it on a scheduled pipeline on GitLab CI which consolidates, organizes the vulnerabilties of all the microservices and extracts it as a csv. This reduced the reporting time to 2 mins from a day.

# Questions

## Tell me about yourself (Cloud, infra, automation, reliability)

I’m a DevOps Engineer with a little over four years of experience **working on cloud-native & on-prem  platforms**, primarily on AWS and Kubernetes, with strong exposure to Terraform, GitLab CI/CD, and GitOps using Argo CD.

In my recent projects I have been **building scalable infrastructure on AWS** using Terraform, **managing Kubernetes clusters** on prem as well as on cloud, and improving developer workflows by **designing CI/CD pipelines** in GitLab.

I also spend a lot of time improving **operational reliability** — **monitoring**, **cost optimization**, and **security hardening** etc.

What excites me about this role is the **opportunity to work on large-scale cloud platforms** and **contribute to designing reliable infrastructure** for multiple clients.



## How would you design a scalable AWS architecture for a production application? (AWS)

**VPC**: I would **start by designing a VPC with public and private subnets across multiple AZs**.
**Public subnets would host the ALB** while **application workloads run in private subnets** inside EKS or ECS.

**Autoscaling**: The application layer would use **auto scaling groups or Kubernetes HPA** to handle traffic spikes. (The HPA scales pods, and the Cluster Autoscaler scales the underlying EC2 nodes (via the ASG) to meet the aggregate demand.)

**Storage**: For persistence I would use RDS in Multi-AZ mode and store static assets in S3.

**Security**: For security I would **enforce least privilege IAM roles** and **store secrets in AWS Secrets Manager**.

**Observability**: would include CloudWatch metrics, centralized logging, and Prometheus/Grafana dashboards.


## How do you manage Terraform state? (Terraform)

I store Terraform state remotely in S3 with DynamoDB for state locking. This prevents concurrent updates and ensures safe team collaboration.

**Bonus**: I also structure infrastructure using reusable Terraform modules and separate environments using workspaces or separate state files.


## How do you reduce AWS costs? (Cost Optimization)

I regularly review CloudWatch and Cost Explorer metrics to identify underutilized resources.
I also implement auto-scaling and use Spot instances for non-critical workloads.
Additionally, I maintain the infra sheet, tag the resources with the env name, as it becomes easy to clean up the orphaned resources.

## Production is down. What do you do? (Incident Handling)

1️⃣ Check monitoring alerts
2️⃣ Identify impact
3️⃣ Check logs/metrics
4️⃣ Rollback if necessary
5️⃣ Communicate with team

First I check monitoring dashboards to identify what component is failing (1️⃣ 2️⃣).
Then I analyze logs and metrics to isolate the root cause (3️⃣).
If the issue is related to a recent deployment, I immediately rollback using the CI/CD pipeline (4️⃣).
During the incident I keep stakeholders informed and afterwards I conduct a post-mortem to prevent recurrence (5️⃣).


## How do you secure AWS infrastructure? (Security)

Key points:

1️⃣ IAM least privilege
2️⃣ private subnets
3️⃣ secrets manager
4️⃣ encryption
5️⃣ security groups
- patching

I follow the AWS shared responsibility model and enforce least privilege IAM policies (1️⃣).
Infrastructure is deployed inside private subnets with restricted security groups (2️⃣ 5️⃣).
Secrets are stored in AWS Secrets Manager or Vault instead of code repositories (3️⃣).
I also enable encryption for S3, EBS, and RDS and use logging services like CloudTrail for auditing (4️⃣).


## How do you run production Kubernetes? (Kubernetes)

1️⃣ EKS
2️⃣ autoscaling
3️⃣ health probes
4️⃣ monitoring
5️⃣ ingress

In production I use managed Kubernetes like EKS (1️⃣).
I configure horizontal pod autoscaling and readiness/liveness probes for reliability (2️⃣ 3️⃣).
For networking I typically use an AWS Load Balancer controller (the recommended way to manage load balancers for Amazon EKS clusters.) (5️⃣).
Observability is handled through Prometheus and Grafana while logs are centralized using CloudWatch or ElasticSearch (4️⃣).


## How do you work with developers? (Leadership question)

I try to work closely with development teams early in the design phase.
My goal is to make infrastructure self-service through Terraform modules and CI/CD pipelines so developers can deploy safely without needing deep infrastructure knowledge.


## Questions to ask

- How mature is the DevOps platform currently in various clients? Are the clients more focused on improving reliability or enabling faster developer deployments?

- In clients projects, Are most of your workloads containerized or are there still many VM-based systems?

- What are the biggest operational challenges the teams is currently facing?


# Impactful words

✔ improved reliability
✔ reduced deployment time
✔ improved scalability
✔ automated infrastructure
✔ reduced operational overhead


- My goal with DevOps is always to make infrastructure repeatable, observable, and resilient.



