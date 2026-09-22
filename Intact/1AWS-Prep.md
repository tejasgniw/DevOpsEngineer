# AWS

EKS (Elastic Kubernetes Service) uses EC2 nodes to run Kubernetes pods (autoscaling groups(using launch template) for scaling nodes, HPA for scaling pods)

## EXFO
#### Brief Networking
We front the ALB with an NLB to get static IPs. DNS resolves to EIPs attached to the NLB. The NLB forwards raw TCP 443 to the ALB, which terminates TLS, applies WAF, and routes HTTP traffic into EKS running in private subnets.

#### Detailed explanation
I built and maintained the AWS infrastructure using Terraform modules: I designed a **secure VPC** with public, private and data subnets, NAT/IGW, SGs and PrivateLink; deployed EKS for workloads and an NLB+ALB+ACM for ingress; and provisioned stateful services (RDS, Neptune, managed RabbitMQ, Mongo Atlas) inside private networks. I also implemented IAM roles/policies, secrets management, VPC endpoints, and monitoring to **enforce least-privilege**, **keep traffic internal where possible**, and support GitOps-driven app deployments.

## Core services

### S3
- S3 (Simple Storage Service): S3 is highly durable object storage for storing and retrieving data.
Used for: Storing files, backups, logs, static websites, images, and application assets.

#### What the module is doing:

- Creates an S3 bucket with tags and naming conventions
- Enables versioning
- Adds lifecycle rules (Lifecycle rules in AWS—most commonly used in Amazon S3 Lifecycle Management—automate the management, cost optimization, and deletion of your cloud data as it ages.)
- Blocks public access
- Enables encryption
- Configures CORS if needed (CORS is a security configuration that allows web applications hosted on one domain to access and interact with resources inside an Amazon S3 bucket belonging to a different domain. By default, web browsers enforce a security rule called the Same-Origin Policy (SOP). This policy blocks frontend client-side scripts (like JavaScript's fetch() or Axios) from pulling assets or uploading files to a different domain unless that destination explicitly grants permission. Configuring CORS on your S3 bucket acts as that explicit permission slip.)

#### How I used it:

- It was used as a storage layer for platform data and artifacts
- It enforced best practices like encryption, versioning, and public access restrictions
- Versioning and lifecycle support recovery and cost efficiency
- It fit well with the platform need for controlled, governed object storage

#### Why it was beneficial:
- Secure object storage without overbroad access
- Versioning helps with rollback and recovery
- Lifecycle policy reduces storage cost
- Public access is blocked by default, which is important in enterprise environments

**I used S3 as the default object storage pattern for the platform. The module enforced versioning, encryption, lifecycle rules, and public access blocking, which is exactly the kind of governance and security baseline you want in a cloud-native platform.**


### VPC
- VPC (Virtual Private Cloud): A VPC is a logically isolated virtual network in AWS.
Used for: Controlling networking — subnets, routing, internet access, private/internal services, security boundaries.

#### What the module is doing:
- Creates the VPC and subnet layout
- Creates public, application, and data subnets
- Creates security groups and ingress/egress rules
- Sets up internet gateways and route tables
- Creates a VPC endpoint for S3

#### How I used it:
- This is the core network isolation layer for the AWS environment
- I used it to separate public ingress from application workloads and data tiers
- Security groups controlled the allowed communication paths
- VPC endpoint for S3 reduced direct internet dependency and helped with private networking

#### Why it was beneficial:
- Network segmentation reduces blast radius
- Better security posture
- Clear separation between public and private resources
- Easier to manage service connectivity and controls

**The VPC module was the network foundation for the platform. It gave us segmented subnets, explicit security group policies, private routing, and S3 VPC access. That was important because we were trying to keep workloads isolated while still allowing the required communication paths.**

### EC2
- EC2 (Elastic Compute Cloud): EC2 provides scalable virtual servers in the cloud.
Used for: Running applications, APIs, backend services, custom workloads where you manage the OS.

#### How I used it:
Direct EC2 usage is not the main pattern here, but the repo still shows the EC2-backed side of the platform:
- EKS node groups use EC2 underneath
- Launch template configuration is in main.tf

#### What this means in practice:
- I did not build standalone EC2 fleet architectures as the main pattern
- Instead, I used EC2 as the compute substrate for EKS worker nodes
- The launch template enforced:
    - encrypted EBS (a virtual, high-performance block storage service designed to act as a persistent hard drive for Amazon EC2 instances. Data remains intact even if you stop, restart or terminate the attached EC2 instance. Volumes are tied to a specific Availability Zone (AZ) for high replication, meaning an EBS volume and its target EC2 instance must reside in the same AZ)
    - IMDSv2 (AWS IMDSv2 (Instance Metadata Service Version 2) is a session-oriented security feature for Amazon EC2 that prevents credential theft and blocks Server-Side Request Forgery (SSRF) attacks.)
    - instance tagging
    - proper operating standards

**I didn’t primarily manage standalone EC2 instances in this repo, but I did work with the EC2-based foundation underneath EKS. That included node group configuration, storage encryption, and metadata hardening, which is critical for a production Kubernetes platform.**

### IAM
- IAM (Identity and Access Management): IAM controls authentication and authorization in AWS.
Used for: Managing users, roles, permissions, policies, and service access securely.

#### What the module is doing:
- Creates IAM policies and roles
- Connects EKS service accounts to AWS permissions via OIDC
- Provides permissions for:
    - Route53
    - CloudWatch
    - autoscaling
    - EBS CSI
    - EFS CSI
    - ALB controller
    - secret access
    - Prometheus remote write

#### How I used it:
- IAM was used as the security boundary for workloads and cluster integrations
- Instead of embedding static credentials, I used role-based access and service account association
- This is a key enterprise pattern: least privilege, not broad access

#### Why it was beneficial:
- Stronger security posture
- Easier rotation and governance
- Better separation between workloads and AWS access
- Auditable access model

**IAM was a central part of the design. I used roles and policies to let Kubernetes workloads access AWS services through OIDC and IRSA patterns, rather than injecting secrets into workloads. That aligns with least privilege and reduces security risk.**


### CloudWatch
- CloudWatch: CloudWatch is AWS’s monitoring and observability service.
**Used for: Logs, metrics, alarms, dashboards, and alerting for applications and infrastructure.**

#### What the module is doing:
- Grants monitoring-related permissions for CloudWatch and Prometheus remote write
- WAF rules produce CloudWatch metrics
- Lambda logs go to CloudWatch automatically
- EKS-related roles are set up to send logs and metrics

#### How I used it:
- It is the observability layer for AWS operations
- CloudWatch gives me metrics, logs, and alarms for platform health and service behavior
- The platform was set up to surface issues early rather than relying only on manual checks

#### Why it was beneficial:
- Better troubleshooting
- Faster detection of incidents
- Operational visibility for both infrastructure and application behavior

**CloudWatch was the operational visibility layer for the platform. I used it for logs and metrics, and in practice the design was to make failures observable without requiring long manual investigation cycles. That is critical in production AWS environments.**


### EKS
- EKS (Elastic Kubernetes Service): EKS is AWS’s managed Kubernetes service.
Used for: Running Kubernetes workloads with full K8s control while AWS manages the control plane.

#### How I used it:
- I provisioned the Kubernetes control plane and worker nodes in AWS
- I configured access patterns using IAM, cluster access entries, and node roles
- I used the platform to host workloads while keeping security and networking in control

#### Why it was beneficial:
- Standardized deployment of containerized workloads
- Easier scaling and automation
- Strong integration with AWS-native security and networking

**The EKS work was about more than just creating a cluster. It was about giving the platform a reliable Kubernetes control plane, secure worker nodes, and AWS-native access control so workloads could run in a consistent, governed way.**


### Load Balancing
- Elastic Load Balancing (ELB): serves as a traffic distribution system designed to improve application availability, fault tolerance, and security.
Used for: Preventing Downtime(traffic routing to healthy servers), Handling Traffic Spikes(auto-scaling), Securing Applications(handles SSL/TLS encryption encryption), Routing Smartly(URL based redirection).

#### What the module is doing:
- Creates a public NLB
- Creates an application load balancer
- Configures listeners and target groups
- Attaches target groups to ALB/NLB
- Adds WAF rate limiting and IP protect patterns

#### How I used it:
- I used load balancing to front traffic and route it to the correct backend
- The NLB handled external entry; the ALB handled application-level routing
- WAF patterns were layered in for rate limiting and access control

#### Why it was beneficial:
- Reduces single points of failure
- Enables traffic control and routing
- Improves resilience and security at the edge

**I used AWS load balancing as the ingress and traffic management layer. We had NLBs for network-level entry and ALBs for app traffic routing, plus target groups and listener rules. That gave us both traffic distribution and better control over exposure and health checks.**

### AWS Backup
- AWS Backup is a fully managed, policy-based service that centralizes and automates data protection across Amazon Web Services (AWS) services, hybrid workloads, and on-premises environments.
#### Core Features
- Centralized Policies: Define backup schedules, retention periods, and lifecycle rules in a single place.
- Resource Tagging: Automatically apply backup plans to AWS resources using tags.
- Cross-Service Support: Protects Amazon EBS, EC2, RDS, DynamoDB, S3, EFS, and more.
- Hybrid Integration: Secures on-premises VMware workloads and hybrid environments.
Used for: Automating and centralizing data protection across AWS services and hybrid workloads using policy-based backup plans.

**I haven’t directly implemented AWS Backup as a central module in this repo, but I understand the value of backup strategy in enterprise cloud operations. In the architecture I worked on, resilience was built around versioned storage, lifecycle controls, resource tagging, and infrastructure repeatability, which are the same principles used in Backup planning.**

### Route 53
- Route 53: Route 53 is AWS’s scalable DNS and domain management service.
Used for: Routing traffic to applications (ALB, NLB, CloudFront), health checks, and domain registration.

#### What the module is doing:
- Looks up the hosted zone
- Creates Route53 A records

#### How I used it:
- It was used for DNS publication of service endpoints
- It supported domain mapping and traffic resolution to exposed services
- In the wider platform, it was part of certificate and environment routing patterns

#### Why it was beneficial:
- Stable, managed DNS
- Integration with public hosting and validation processes
- Makes service exposure predictable and environment-friendly

**Route53 was part of the platform’s public entry patterns. I used it for record creation and domain association so services and environments had stable, managed DNS names instead of ad hoc endpoint configuration.**


### KMS
- AWS KMS (Key Management Service) is a managed cloud service that lets you create, control, and manage cryptographic keys used to protect your data.
Used for: AWS KMS is used to securely generate, store, and control cryptographic keys to encrypt data across AWS services and custom applications.

I’ve worked with encryption at rest as a platform requirement, especially in S3 and EBS. In an enterprise environment, KMS is the natural control plane for key management, and my experience is aligned with using managed encryption and key-based governance rather than leaving storage unencrypted.”

### Lambda

#### What the module is doing:
- Packages Lambda code as a ZIP payload and uploads it to S3
- Creates Lambda functions with:
- function name
- runtime
- memory
- timeout
- IAM role
- VPC config
- environment variables
- Adds a CloudWatch Event schedule for warmup
- Grants Lambda permission to be invoked by EventBridge

How I used it:

- I used **Lambda as serverless compute for event-driven processing** behind Step Functions
- The functions were not standalone “toy Lambda apps”; they were integrated into orchestrated workflows and platform automation
- The function had VPC access so it could reach private resources without exposing them publicly
- It was managed as code in Terraform, which made deployments repeatable and auditable.

**At EXFO, I used Lambda as part of a Step Functions-driven workflow. The Lambda functions were Terraform-managed, had VPC access, used an execution role with least privilege, and were configured with defined memory/timeout values and CloudWatch logging. This gave us serverless automation without losing security or observability.**

### Step functions

- Step Functions is a serverless orchestration service that automatically chains multiple Lambda functions together without writing orchestration code. When a user uploads a report file to S3, Step Functions (already has **definition.json**) triggers a state machine that executes our Lambdas sequentially—first **throw_on_error** state validates the file, then **update_report_attachments** state stores it, followed by **update_report_status** state marking it as processed, and finally **update_report_warnings** state logs any issues. If any Lambda fails, Step Functions automatically retries it or jumps to an error handler, eliminating the need to write retry logic in code and giving us a complete audit trail of each report's processing journey.

or in short

- Step Functions orchestrates our report processing by automatically executing four Lambdas in sequence—validate, store, update status, and log warnings—without us writing any orchestration code. If one Lambda fails, it automatically retries or handles the error; we get a complete execution history showing exactly where each report succeeded or failed.

#### Lambdas (How Lambdas Work in Step Functions?)

Lambdas = Individual work units orchestrated by the state machine:

Each Lambda handles a specific task in the reporting pipeline (e.g., data extraction, processing, report generation)
The Step Function definition (**definition.json**) specifies the workflow order and transitions between Lambdas
Lambda gets invoked via the state machine, processes data, returns results, then passes control to the next step

#### CloudWatch
CloudWatch serves three purposes here:

1) Lambda Warmup (keep-alive)

aws_cloudwatch_event_rule: Scheduled rule that runs every 5 minutes
Invokes Lambdas to prevent cold starts (avoids latency spikes when they haven't been called recently)

2) Logs & Monitoring

Collects Lambda execution logs and metrics
Used to debug failed step executions and monitor performance

3) Metrics

Tracks Lambda duration, invocations, errors
Feeds into alerts and dashboards for operational visibility

**Cloud watch vs Cloud trial**

AWS CloudTrail and CloudWatch are distinct monitoring services: CloudTrail logs API activity for security and auditing ("who did what"), while CloudWatch monitors operational performance, metrics, and logs ("how are resources behaving"). CloudTrail tracks user actions, whereas CloudWatch provides real-time infrastructure data and alarms. 

**In summary: Lambdas do the work, Step Functions choreograph them, and CloudWatch keeps them warm + monitored.**

## Especially Lambda

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



### scenario questions

**"A Lambda suddenly starts timing out in production. How would you troubleshoot it?"**

You should be able to answer systematically:

Impact → CloudWatch → duration → logs → dependencies → networking → IAM → concurrency → downstream service → recent changes → mitigation → root cause → prevention

Or simply

**Logs → Metrics → Configuration → IAM → Network → Dependencies → Recent changes**

### What kind of AWS infrastructure I worked on:

I built and maintained the AWS infrastructure using Terraform modules: I designed a secure VPC with public, private and data subnets, NAT/IGW, SGs and PrivateLink; deployed EKS for workloads and an ALB+ACM for ingress; and provisioned stateful services (RDS, Neptune, managed RabbitMQ, Mongo Atlas) inside private networks. I also implemented IAM roles/policies, secrets management, VPC endpoints, and monitoring to enforce least-privilege, keep traffic internal where possible, and support GitOps-driven app deployments.

#### others
In EXFO, I used Terraform modules to build a reusable AWS platform foundation: VPC/networking, EKS cluster, IAM with OIDC/IRSA, ALB/NLB ingress, S3 storage, Route53, CloudWatch, and Lambda-based Step Functions orchestration.

My hands-on AWS experience is mostly in platform engineering and infrastructure automation. At EXFO, I built reusable Terraform modules for EKS, VPC, IAM, S3, load balancing, Route53, and Lambda-based orchestration. I’ve worked on secure multi-environment AWS platform design, with a focus on least-privilege access, network segmentation, CI/CD automation, and production reliability.

Across EXFO, I used AWS as a platform foundation rather than isolated services. I built VPC networking, EKS Kubernetes, IAM and OIDC-based access, S3 storage with governance controls, Lambda-based automation, Route53-based service naming, and load balancing for traffic routing. Everything was managed via Terraform, which gave us repeatability, secure access patterns, and easier CI/CD-driven deployment. My focus was not just spinning up AWS resources, but building a secure, operationally manageable platform with clear ownership, observability, and production readiness.

### GCP services that I have worked on:

#### Interview explanation

In this project, GCP is used exclusively as the identity and authentication control plane. We use Google Cloud Identity Platform as an OIDC-compliant Identity Provider for user authentication, with Cloud Functions acting as policy enforcement points during user lifecycle events. The application itself runs on AWS EKS, consuming GCIP-issued tokens. CI/CD authentication to GCP is handled securely using Workload Identity with GitLab via OIDC, avoiding static credentials.

#### Interview short

We use GCP for identity and authentication control plane — GCIP provides OIDC-based authentication with custom policy hooks via Cloud Functions, while AWS handles the application and data plane.


