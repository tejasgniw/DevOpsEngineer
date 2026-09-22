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


## Tell me about a challenging problem you solved (Cost optimization, Security hardening)

In one of my projects, we had frequent deployment failures in our Kubernetes environment which were affecting release timelines.

My task was to improve the reliability of the deployment process.

I analyzed the pipeline and identified that there were inconsistencies in environment configurations and lack of proper validation.

I implemented standardized Terraform modules, added validation steps in the CI/CD pipeline, and improved logging for better debugging.

As a result, deployment failures were significantly reduced and the team gained more confidence in releasing changes.


## Tell me about a failure or mistake

In one case, I deployed a configuration change that caused a service disruption due to insufficient validation in staging.

I took responsibility and worked on quickly rolling back the change to restore service.

After that, I improved the pipeline by adding additional validation and testing steps to prevent similar issues in the future.

This experience taught me the importance of validating changes thoroughly before production deployment.


## What is the most complex system you worked on?


## How do you handle pressure or incidents? (Full RDS due to Data aggregation)

During high-pressure situations like production incidents, I focus on quickly identifying the impact and stabilizing the system first.

I rely on monitoring tools and logs to guide decisions, and I communicate clearly with the team throughout the process.

Staying calm and following a structured approach helps resolve issues effectively.


## Tell me about a time you improved something (CI/CD Caching, baking binaires into docker images, Building tools for security reporting, Semantic versioning)

In one project, deployment times were quite slow and affecting developer productivity.

I optimized the CI/CD pipeline by parallelizing stages and improving caching mechanisms.

This reduced deployment time significantly and improved overall team efficiency.


## How do you prioritize tasks? (Impact and urgency)

I prioritize tasks based on impact and urgency.

Production issues and anything affecting customers come first, followed by tasks that improve system reliability and automation.

I also align priorities with team and business goals to ensure we focus on what delivers the most value.


## How do you deal with developers who don’t follow DevOps practices?

I try to understand their challenges first and then work with them to make processes easier rather than enforcing strict rules.

For example, I create reusable pipelines and templates so that following best practices becomes the easiest option for them.

The goal is to enable developers, not block them.


## What would you do in your first 30 days?


## What motivates you?

I enjoy solving complex infrastructure problems and improving system reliability.

I’m also motivated by building automation that makes life easier for teams and allows faster, safer deployments.


## Why this role / company?

I’m excited about this role because **it involves working on large-scale cloud platforms and contributing to real digital transformation projects.**

I also value the opportunity to work in an environment **that emphasizes learning and collaboration, especially being part of a global ecosystem like IBM.**



## Questions to ask

- How mature is the DevOps platform currently in various clients? Are the clients more focused on improving reliability or enabling faster developer deployments?

- What are the biggest operational challenges your team is currently facing? 

- In clients projects, Are most of your workloads containerized or are there still many VM-based systems?

- What are the biggest operational challenges the teams is currently facing?


## Interesting questions to ask

- With Ingress-NGINX being retired this month and no longer receiving updates, how is your team planning to evolve its ingress strategy—are you considering another controller or moving toward Gateway API?

- With IBM acquiring Confluent, how do you see DevOps practices evolving to support real-time, AI-driven workloads—especially in terms of streaming infrastructure and reliability?”

- With k8s v1.36, the feature HPA scaling to zero is likely one of the best features available natively. Any of your clients have plans on upgrading to it in dev/staging?