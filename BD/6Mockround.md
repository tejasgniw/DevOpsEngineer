# Mock round

## 1. Can you briefly introduce yourself and tell me what kind of DevOps problems you enjoy solving the most? (Introduction)

**Intro**: I’m a DevOps Engineer with a little over four years of experience **working on cloud-native & on-prem  platforms**, primarily on AWS and Kubernetes, with strong exposure to Terraform, GitLab CI/CD, and GitOps using Argo CD.

**Focus**: My work focuses on **building scalable infrastructure on AWS** using Terraform, **managing Kubernetes clusters** on prem as well as on cloud, and improving developer workflows by **designing CI/CD pipelines** in GitLab.

**Reliability**: I also spend a lot of time improving **operational reliability** — **monitoring**, **cost optimization**, and **security hardening** across environments.

**Love**: The problems I enjoy solving most are around **making infrastructure repeatable, observable, and resilient**, so that teams can **deploy faster while maintaining production stability**.



## 2. Suppose we have a customer-facing web application running on AWS that suddenly starts receiving 10x more traffic because of a marketing campaign. How would you design the infrastructure so that the system scales automatically while remaining reliable? (Architecture at high traffic)

**Multi-AZ**: First I would **design the system to run across multiple availability zones** inside a VPC with public and private subnets.

**Load balancing**: Traffic would enter through an **Application Load Balancer**, which **distributes requests across the application layer** running in EKS or ECS in private subnets.

**Autoscaling groups & HPA**: At **the compute layer**, I would **enable Horizontal Pod Autoscaling** so pods scale based on CPU or custom metrics, and configure **cluster autoscaling** to automatically add or remove worker nodes when needed.

**Load reduction**: To reduce load on the application, I would also place CloudFront and possibly caching like Redis (ElastiCache) in front of the services for frequently accessed data.

**Storage**: For persistence, I would use RDS in Multi-AZ mode to ensure database availability during traffic spikes.

**CloudWatch**: Finally, monitoring with CloudWatch and Prometheus would help track performance metrics and trigger alerts if thresholds are exceeded.



## 3. One day a developer accidentally runs a change that destroys a critical production resource (for example an RDS instance). What controls or practices would you implement to prevent this kind of situation in production? (Disaster Scenario prevention)

1️⃣ Terraform protections
2️⃣ CI/CD approvals
3️⃣ Separate environments
4️⃣ IAM restrictions
5️⃣ Backup / recovery

**Multi-layer protection**: I would implement multiple layers of protection to prevent accidental destruction of critical resources.

**Terraform level**: First, at the Terraform level I would use lifecycle rules like prevent_destroy for critical resources such as production databases.

**Pipeline only**: Second, Terraform changes would only be applied through CI/CD pipelines with manual approval gates for production environments.

**Workspace, Environment & state**: Third, I would separate environments using different state files and AWS accounts so developers cannot accidentally impact production resources.

**IAM**: Fourth, IAM permissions would restrict who can run production deployments(least-privilege).

**Backups**: Finally, I would ensure automated backups and snapshots for RDS, so that even if something goes wrong we can recover quickly.

**Bonus**: In production environments we should always assume human error will happen, so the system should be designed to minimize blast radius.


## 4. Your team deploys a new version of an application through the CI/CD pipeline. Five minutes later, errors in production suddenly increase and customers start complaining. Walk me through exactly what you would do in the first 10 minutes after discovering the issue. (Production incident)

**Validation**: First I would confirm the alert and quickly assess the impact on users and services using monitoring dashboards like CloudWatch or Grafana.

**Logs of recent deplpoyment**: Since the issue happened right after a deployment, I would immediately check the recent deployment changes and logs to verify if the new version introduced errors.

**Rollback**: If the new deployment is the likely cause, I would trigger a rollback through the CI/CD pipeline to restore the last stable version and stabilize the system.

**Communicate**: At the same time I would communicate the incident to the team so everyone is aware and can assist if needed.

**Post-mortem**: Once the system is stable, we would analyze logs and metrics more deeply to identify the root cause and conduct a post-incident review to prevent similar issues in the future.

**Bonus**: During incidents the priority is always to restore service quickly, and then investigate root cause afterward.



## 5. Many engineers focus heavily on automation and CI/CD pipelines. But sometimes automation itself becomes complex and fragile. How do you decide when to automate something and when to keep it simple/manual?

1️⃣ Frequency
2️⃣ Risk
3️⃣ Operational value


**Factors**: I usually decide whether to automate something based on three factors: **frequency, risk, and operational value.**

If a task is repetitive, such as infrastructure deployment or application releases, automation through CI/CD pipelines is very valuable because it reduces human error and ensures consistency across environments.

For one-time or experimental tasks, I prefer running them manually at first, especially during development or troubleshooting, because it allows faster iteration and debugging.

Once the process becomes stable and repeatable, that's when I move it into automation.

My goal with automation is to **reduce operational risk** and **make processes consistent**, rather than automating everything unnecessarily.

**Bonus**: Automation should simplify operations, not introduce unnecessary complexity.

## 6. Imagine you join a project where the AWS bill is $80,000 per month, and leadership asks you to reduce costs by 30%. What would be the first things you analyze and optimize?

1️⃣ Cost Explorer analysis
2️⃣ Right-sizing compute
3️⃣ Reserved / Savings Plans
4️⃣ Autoscaling
5️⃣ Storage lifecycle policies

**Cost Explorer**: The first step would be to analyze the AWS bill using Cost Explorer and identify the top cost drivers such as compute, databases, storage, and data transfer.

**Compute resources**: Typically, the biggest savings come from optimizing compute resources. I would review EC2 or Kubernetes workloads to ensure instances are right-sized and that autoscaling is configured properly.

**Reserved Instances**: Next, I would evaluate opportunities to use Reserved Instances or Savings Plans for stable workloads, which can significantly reduce compute costs. Or use HPA scale to 0 for asynchronous background workers, queue consumers, batch processing jobs or non-prod /dev envs.

**Review storage**: I would also review storage usage, for example applying S3 lifecycle policies and removing unused EBS volumes or snapshots.

**Optimization**: Finally, I would review architectural optimizations such as caching, reducing NAT gateway traffic, or optimizing database usage. (Using VPC endpoints for S3, DynamoDB, ECR, Cloudwatch & Cache content on the edge using CDN, Reduce NAT Gateways for Non-prod etc & Use Amazon ElastiCache to cache frequently accessed query results, Use read replicas for read-intensive workloads to reduce the load on the primary node.)

**Latest**: With v1.36 of k8s, we have HPA scaling to Zero which was not native earlier,big win for cost optimization especially for dev/staging env where the pods often sit idle. 

The goal is to focus on the highest cost components first to achieve meaningful savings.

**Bonus**: In most environments **compute, data transfer and databases** account for the majority of the AWS bill, so that's where I usually start



## 7. In your opinion, what is the biggest mistake companies make when adopting DevOps?

Treating DevOps as a toolset instead of a culture.

**DevOps**: One of the biggest mistakes I see is when companies treat DevOps as just a collection of tools instead of a cultural and operational change.

**Adopt too fast**: Sometimes organizations rush into adopting tools like Kubernetes, Terraform, or CI/CD pipelines without clearly defining their architecture, workflows, and operational practices.

This can lead to overly complex systems that are difficult to maintain.

**Focus**: In my experience, DevOps works best when teams first focus on collaboration, clear deployment processes, and operational reliability, and then introduce automation gradually where it provides the most value.

**Bonus**: DevOps is successful when it reduces friction between development and operations while improving reliability.



## 8. Let’s say your company has multiple AWS accounts for different environments and teams. How would you design a secure and manageable multi-account AWS strategy? What controls or services would you use to manage those accounts effectively?

**AWS Organizations**: For a multi-account AWS environment, I would start by organizing accounts using AWS Organizations so that governance policies can be applied centrally.

**Separate accounts**: I would separate accounts by environment and purpose — for example development, staging, production, and shared services accounts.

**Security and auditing**: I would enable CloudTrail and centralized logging, sending logs from all accounts to a dedicated logging account.

Amazon CloudWatch monitors the operational health and performance of your resources and applications, whereas AWS CloudTrail tracks and audits user activity, API calls, and account changes for security and compliance.

**SCP**: I would also use Service Control Policies (SCPs) to enforce guardrails, such as restricting certain services or regions.

**IAM**: Access management would be handled through AWS IAM roles or AWS SSO, ensuring users assume roles instead of using long-term credentials.

This approach provides strong security boundaries while keeping the environment manageable.

**Bonus**: Using separate AWS accounts also helps reduce the blast radius of incidents.

## 9. Have you worked with SLI, SLO, SLA and error budgets at EXFO?

I want to be transparent here: I did not formally own or implement an SRE program based on SLOs and error budgets at EXFO as I was in the Development team.

- My role was more focused on DevOps, cloud infrastructure, CI/CD, Kubernetes, automation and operational reliability. **However, I worked with many of the underlying practices that feed into SLI and SLO management.**

For example, we used monitoring and operational data around Kubernetes, AWS infrastructure and CI/CD to identify issues, track system health and troubleshoot production problems. We also looked at things like deployment reliability, failures, recovery and the impact of infrastructure or pipeline issues.

- So while I wasn't responsible for defining formal SLOs or managing an error budget, I understand how they fit into an SRE model.

**For example, an SLI could be availability, latency or error rate. An SLO would define the target for that SLI, such as 99.9% availability over a given period. The error budget is then the amount of unreliability we can tolerate while still meeting that objective.**

If I joined a team where these practices were already established, I would be comfortable working with those metrics and using them to guide operational priorities, deployment decisions and reliability improvements.



| Concept                         | Your EXFO experience                                                       |
| ------------------------------- | -------------------------------------------------------------------------- |
| **SLI**                         | You worked with operational/monitoring measurements                        |
| **SLO**                         | You understand the concept, but didn't formally define/manage them         |
| **SLA**                         | Likely business/customer commitment rather than your direct responsibility |
| **Error budget**                | You understand the concept, but didn't formally manage one                 |
| **Monitoring**                  | **Strong hands-on experience**                                             |
| **Reliability troubleshooting** | **Strong hands-on experience**                                             |
| **Kubernetes/AWS health**       | **Strong hands-on experience**                                             |
| **CI/CD reliability**           | **Strong hands-on experience**                                             |
| **Automation/prevention**       | **Strong hands-on experience**                                             |


## 10. Tell me about your experience on Github actions

My strongest experience is GitLab CI/CD, but I've worked extensively with the underlying CI/CD architecture and automation patterns, so I can transfer that experience to GitHub Actions.


# Important sentences:

1️⃣
“My goal with DevOps is to make infrastructure repeatable, observable, and resilient.”

2️⃣
“In production systems the priority during incidents is always restoring service quickly.”

3️⃣
“Automation should reduce operational risk, not introduce unnecessary complexity.”

4️⃣
“In most cloud environments compute, data transfers and databases are the biggest cost drivers.”

5️⃣
“Good architecture always tries to minimize blast radius.”