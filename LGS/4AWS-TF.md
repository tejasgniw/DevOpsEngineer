# AWS

## Existing experience

### Brief Networking
We front the ALB with an NLB to get static IPs. DNS resolves to EIPs attached to the NLB. The NLB forwards raw TCP 443 to the ALB, which terminates TLS, applies WAF, and routes HTTP traffic into EKS running in private subnets.

### Detailed explanation
I built and maintained the AWS infrastructure using Terraform modules: I designed a secure VPC with public, private and data subnets, NAT/IGW, SGs and PrivateLink; deployed EKS for workloads and an NLB+ALB+ACM for ingress; and provisioned stateful services (RDS, Neptune, managed RabbitMQ, Mongo Atlas) inside private networks. I also implemented IAM roles/policies, secrets management, VPC endpoints, and monitoring to enforce least-privilege, keep traffic internal where possible, and support GitOps-driven app deployments.

### Step functions
- Step Functions is a serverless orchestration service that automatically chains multiple Lambda functions together without writing orchestration code. When a user uploads a report file to S3, Step Functions (already has **definition.json**) triggers a state machine that executes our Lambdas sequentially—first **throw_on_error** state validates the file, then **update_report_attachments** state stores it, followed by **update_report_status** state marking it as processed, and finally **update_report_warnings** state logs any issues. If any Lambda fails, Step Functions automatically retries it or jumps to an error handler, eliminating the need to write retry logic in code and giving us a complete audit trail of each report's processing journey.

or in short

- Step Functions orchestrates our report processing by automatically executing four Lambdas in sequence—validate, store, update status, and log warnings—without us writing any orchestration code. If one Lambda fails, it automatically retries or handles the error; we get a complete execution history showing exactly where each report succeeded or failed.


### Lambdas (How Lambdas Work in Step Functions?)

Lambdas = Individual work units orchestrated by the state machine:

Each Lambda handles a specific task in the reporting pipeline (e.g., data extraction, processing, report generation)
The Step Function definition (**definition.json**) specifies the workflow order and transitions between Lambdas
Lambda gets invoked via the state machine, processes data, returns results, then passes control to the next step


### CloudWatch
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

### Securityhub

**AWS Security Hub** and **GuardDuty** are complementary AWS security services: GuardDuty acts as the **threat detection engine** (using ML to find malicious activity), while Security Hub serves as the **centralized management platform** that aggregates, organizes, and prioritizes findings from GuardDuty, Inspector, and Macie. Together, they provide continuous monitoring, compliance checks, and automated incident response across AWS accounts. 

### EC2 (via EKS)
Where: main.tf:86

- Why: EKS (Elastic Kubernetes Service) uses EC2 nodes to run Kubernetes pods (autoscaling groups for scaling nodes, HPA for scaling pods)
- Line 92: instance_type_regular_node = local.compute_size[var.size] (t3.nano/micro/small for DEV, m6i.xlarge for PROD)
- EC2 instances form the Kubernetes cluster that runs all the microservices (adminservices, reporting, topology, etc.)

### RDS (PostgreSQL)
Where: main.tf:224

- Why: Persistent relational database for services
- Line 213-217: **Creates databases conditionally** for: adminservices, pki, subscription, equipment-identity
- Line 203: rds = (length(local.psql_databases) > 0) ? [module.postgresql_db[0].rds_databases_arn] : [] — RDS ARN is passed to IAM for access control
- RDS Enhanced Monitoring role (line 112) — allows RDS to write metrics to CloudWatch

### IAM
Where: Three modules:

- main.tf:103 — Base infra (OIDC provider, RDS monitoring role)
- main.tf:129 — Creates service roles for EKS pods
- main.tf:148 — Attaches policies to roles
- Why: Each Kubernetes pod assumes an IAM role to access AWS resources (S3, RDS, Lambda, Neptune, etc.)

### S3

S3 is heavily used. Here's where:

- main.tf:570
What it does: Creates multiple S3 buckets data-driven (via for_each loop over local.all_buckets_list)

- Why & Where used:

#### Attachments/Exports — Store file uploads

- attachments_bucket (line in workload_cluster)
- exports_bucket
- custom_export_bucket

#### Reporting — Store report binaries and data

- reports_binaries_main_bucket — Lambda payload for step functions (line 577)
- reports_customerdata_main_bucket — Report data

#### Topology — Store topology data

- topology_service_bucket

#### Activity/Audit Logs — Compliance

- activity_logs_bucket
- audit_logs_bucket
- message_backups_bucket

#### Lambda Code — Step functions payload

Line 577: lambda_payload_bucket = local.reports_binaries_main_bucket
Line 586: depends_on = [module.s3] — Step functions waits for buckets

#### Configuration: Each bucket has:

Public access blocked (block_any_public_access = true)
CORS enabled for cross-origin requests
RPO (Recovery Point Objective) = 1 hour
Comprehensive tagging for cost/compliance tracking

#### IAM Access: Services get S3 ARNs passed via resources_arn_map["service"].buckets to the IAM policies module for read/write permissions.


## AWS definitions prep (EC2, S3, RDS, Lambda, VPC, IAM, CloudFormation/Terraform, CloudWatch)

- EC2 (Elastic Compute Cloud): EC2 provides scalable virtual servers in the cloud.

Used for: Running applications, APIs, backend services, custom workloads where you manage the OS.

- S3 (Simple Storage Service): S3 is highly durable object storage for storing and retrieving data.

Used for: Storing files, backups, logs, static websites, images, and application assets.

- RDS (Relational Database Service): RDS is a managed relational database service (A relational data service, such as Amazon Relational Database Service (RDS), is a managed cloud service that simplifies setting up, operating, and scaling structured, SQL-based databases.)

Used for: Running MySQL, PostgreSQL, SQL Server, etc., without managing database infrastructure (patching, backups, HA).

- Lambda: Lambda is a serverless compute service that runs code in response to events/triggers.

Used for:
Event-driven processing, APIs, automation, file processing, background jobs — without managing servers.

- VPC (Virtual Private Cloud): A VPC is a logically isolated virtual network in AWS.

Used for:
Controlling networking — subnets, routing, internet access, private/internal services, security boundaries.

- IAM (Identity and Access Management): IAM controls authentication and authorization in AWS.

Used for:
Managing users, roles, permissions, policies, and service access securely.

- CloudFormation / Terraform (Infrastructure as Code): Tools used to provision and manage infrastructure using code.

Used for:
Automating AWS resource creation (EC2, VPC, RDS, etc.), enabling version control, repeatability, and CI/CD automation.

- CloudWatch: CloudWatch is AWS’s monitoring and observability service.

Used for:
Logs, metrics, alarms, dashboards, and alerting for applications and infrastructure.

### Ultra-Short Interview Summary

EC2 is compute, S3 is storage, RDS is managed database, Lambda is serverless compute, VPC is networking, IAM is access control, CloudFormation/Terraform are Infrastructure as Code tools, and CloudWatch handles monitoring and logging.

## AWS: ECS/EKS, API Gateway, Route 53, ALB/NLB, Auto Scaling, AWS Secrets Manager

- ECS (Elastic Container Service): ECS is AWS’s managed container orchestration service.

Used for:
Running and scaling Docker containers without managing Kubernetes. Good for simpler container workloads.

- EKS (Elastic Kubernetes Service): EKS is AWS’s managed Kubernetes service.

Used for:
Running Kubernetes workloads with full K8s control while AWS manages the control plane.

- API Gateway: API Gateway is a fully managed service for creating and managing APIs.

Used for:
Exposing REST/HTTP/WebSocket APIs, integrating with Lambda, ECS, or backend services, handling throttling, auth, and monitoring.

- Route 53: Route 53 is AWS’s scalable DNS and domain management service.

Used for:
Routing traffic to applications (ALB, NLB, CloudFront), health checks, and domain registration.

- ALB (Application Load Balancer): ALB is a Layer 7 load balancer for HTTP/HTTPS traffic.

Used for:
Routing web traffic based on path or host (e.g., /api → backend, /app → frontend).

- NLB (Network Load Balancer): NLB is a high-performance Layer 4 load balancer.

Used for:
Handling TCP/UDP traffic, extreme performance workloads, or when low latency is required.

- Auto Scaling: Auto Scaling automatically adjusts compute capacity based on demand.

Used for:
Scaling EC2 instances, ECS tasks, or EKS nodes to maintain performance and optimize cost.

- AWS Secrets Manager: Secrets Manager securely stores and manages sensitive credentials.

Used for:
Storing database passwords, API keys, tokens, and rotating them automatically.


## AWS Shared responsibility

The AWS Shared Responsibility Model defines which security responsibilities are handled by AWS and which are handled by the customer.

### Core Concept

- AWS secures the cloud.
- Customers secure what they put in the cloud.

### AWS is responsible for:

- Physical data centers
- Hardware
- Networking infrastructure
- Hypervisor
- Managed service infrastructure

#### Global availability:

- Physical server security
- Power, cooling
- DDoS protection at infrastructure layer
- Patching underlying host OS for managed services

### Customer Responsibility (Security in the Cloud)

Customers are responsible for:

- IAM configuration
- Data encryption
- Application security
- OS patching (for EC2)
- Network configuration (Security Groups, NACLs)
- Secrets management
- Monitoring and logging

One liner: AWS follows a shared responsibility model where AWS secures the underlying infrastructure — including hardware, networking, and managed service platforms — while the customer is responsible for securing workloads, data, IAM policies, encryption, and configuration. The exact boundary depends on the service type: for EC2, the customer manages the OS and applications; for managed services like RDS or Lambda, AWS handles more of the operational security.

The more managed the service, the more security AWS handles — but data, identity, and access are always the customer’s responsibility.

## security standards: (PIPEDA, SOC2, ISO 27001)

PIPEDA is a Canadian privacy law governing personal data protection. SOC 2 is an audit framework focusing on data security and trust controls, commonly required for SaaS companies. ISO 27001 is an international information security management standard focused on risk-based security governance. In our architecture, services like IAM, encryption, Secrets Manager, and logging support compliance with these frameworks.

| Framework | Type                   | Focus                          | Mandatory?                       |
| --------- | ---------------------- | ------------------------------ | -------------------------------- |
| PIPEDA    | Law                    | Personal data privacy (Canada) | Yes (if applicable)              |
| SOC 2     | Audit standard/framewrk| Data security & trust controls | Contractual/business req/ SAAS   |
| ISO 27001 | International standard | InfoSec management system      | Certification-based              |

How This Connects to Shared Responsibility

Even though:
- AWS is SOC 2 compliant
- GCP is ISO 27001 certified

👉 You are still responsible for:

- Configuring IAM correctly
- Not exposing S3 publicly
- Encrypting data
- Managing access properly

Cloud providers give you compliant infrastructure. You must configure it securely.

### Relevancy in my project

#### PIPEDA

Stands for Personal Information Protection and Electronic Documents Act. PIPEDA is Canada’s federal privacy law governing how organizations collect, use, and protect personal data.

- Uses GCIP for user authentication
- Stores user data in AWS (EKS, RDS, S3)

You must ensure:

- User data is encrypted (at rest & in transit)
- Access is controlled via IAM
- Logging and monitoring (CloudWatch)
- Secure secret management (Secrets Manager)
- Proper data handling policies

👉 If you store Canadian user data → PIPEDA applies.

#### SOC 2

SOC 2 is a security compliance framework that evaluates how organizations protect customer data.

My architecture controls

| SOC 2 Requirement | Your Implementation  |
| ----------------- | -------------------- |
| Access control    | IAM roles & policies |
| Monitoring        | CloudWatch           |
| Secure secrets    | Secrets Manager      |
| Change management | Terraform            |
| Availability      | Auto Scaling + ALB   |
| Incident logging  | Cloud logs           |

👉 If your company sells SaaS to enterprises, SOC 2 is almost mandatory.

#### ISO 27001

ISO 27001 is an international standard for establishing and maintaining an Information Security Management System (ISMS).

My cloud setup supports ISO 27001 controls like:

- Identity management (IAM, GCIP)
- Network segmentation (VPC)
- Encryption (TLS, KMS)
- Logging & monitoring
- Infrastructure as Code (change traceability)
- Workload Identity (secure CI/CD)

👉 ISO 27001 is broader than SOC 2 — it includes governance and risk management, not just technical controls.


# Terraform

## Terraform: HCL, Modules, State Management, Workspaces

HCL (HashiCorp Configuration Language)

- Built a production-grade infrastructure using HCL with conditional logic, locals, loops, and dynamic blocks
- Used for_each loops to provision multiple S3 buckets data-driven:

```hcl
for_each = {
  for bucket in local.all_buckets_list : bucket.key => bucket
  if lookup(bucket, "create", false) == true
}
```

- Implemented complex variable validation and interpolation (e.g., local.compute_size[var.size] mapping DEV/PROD sizes)
- Leveraged try(), contains(), concat(), length() for flexible resource provisioning

**One liner**: I've written complex HCL with conditional provisioning based on environment variables. For example, in the exchange module, I use contains(var.services, "reporting") to conditionally create reporting infrastructure only when that service is enabled. This reduces overhead and keeps deployments lean.

## Modules (Reusable & Maintainable)

### Architecture built:

- 11+ reusable modules managed in a central platform-modules repository (exchange, iam_exchange, iam_exchange_role, postgresql_db, step_functions, neptune, etc.)
- Module composition pattern: Higher-level modules call lower-level modules. For e.g. exchange module (main) → calls iam, iam_exchange_role, iam_exchange, postgresql_db, s3, step_functions, etc.
- Each module published to GitLab package registry with versioning (e.g., ~> 1.0.2)

### Module design practices:

#### Single Responsibility: Each module handles one concern

- iam_exchange_role creates roles only
- iam_exchange attaches policies only
- postgresql_db manages RDS only

#### Inputs/Outputs pattern:

- iam_exchange takes role_name_map as input and outputs policy ARNs
- step_functions returns lambda_arns, core_arns, activity_arn

#### Dependency management: depends_on clauses ensure correct order

- step_functions depends on s3 (needs bucket for Lambda payload(A Lambda payload is the data that is sent to an AWS Lambda function as input when it is invoked, or the data the function returns as output. This data is typically a JSON object, but it can also be a stream, buffer, or string, depending on the invocation type and the configuration. ))
- postgresql_db depends on networking (needs subnets)


**One liner**: I've designed and maintained 15+ reusable Terraform modules organized by infrastructure concern. For example, the iam_exchange and iam_exchange_role modules separate role creation from policy attachment—this follows the principle of single responsibility and makes policies reusable across different services. Each module is published with semantic versioning to a private registry, allowing other teams to consume them without managing raw code.

## State Management

### Remote State Backend: backend S3 & HTTP backend with GitLab Terraform State

If configured to remote state

```hcl
terraform {
  backend "s3" {
    bucket         = "exfo-terraform-states" # fixed value
    region         = "us-east-1"             # fixed value
    dynamodb_table = "exfo-terraform-states" # fixed value
    assume_role = {
      role_arn = "arn:aws:iam::905418345845:role/exchange-terraform-state"
    }
  }
}
```

If Configured in CI/CD

```hcl
terraform {
  backend "http" {}
}
```

```hcl
terraform init \
  -backend-config=address=${GITLAB_ADDRESS} \
  -backend-config=lock_address=${GITLAB_ADDRESS}/lock \
  -backend-config=unlock_address=${GITLAB_ADDRESS}/lock \
  -backend-config=lock_method=POST
```

### State Locking: Prevents concurrent modifications

- Lock/unlock via POST/DELETE methods
- GitLab manages the lock file automatically
or
- Remote state in s3 with a statelock management done by the DynamoDB

### Environment-specific state: Each branch (dev, qa, prod) has its own state

```
GITLAB_ADDRESS: "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/terraform/state/${CI_COMMIT_REF_NAME}"
```

or Workspace and the environments(dev,qa,prod) as below

```hcl

  WORKSPACE: ci-$CI_PROJECT_NAME-$CI_COMMIT_REF_SLUG
  ENVIRONMENT: $CI_PROJECT_NAME-$CI_COMMIT_REF_SLUG
```

### Sensitive outputs: Password hashes, tokens marked as sensitive = true to avoid logs


**One liner**: "I've set up remote state management using GitLab's Terraform backend & also with the s3 remote backend+dynamoDB with automatic locking to prevent race conditions during concurrent deployments. Each environment (dev/qa/prod) maintains a separate state file keyed by git branch name in gitlab/Environment name in S3 cloud, ensuring isolation. State is stored securely in GitLab/S3, and I configure terraform init with backend flags/or S3 backend in the CI/CD pipeline."


## Workspaces (Implicit via Environment Variables)

Uses environment names as state keys instead of workspaces

```
TF_VAR_environment: $CI_COMMIT_REF_NAME  # Branch name = environment
GITLAB_ADDRESS: ...terraform/state/${CI_COMMIT_REF_NAME}  # State per branch
```

### Dev/QA/Prod separation through:

- Different *.tfvars files (dev.tfvars, qa.tfvars, prod.tfvars)
- Flavor variables controlling instance types: SMALL (DEV) vs PROD (m6i.xlarge)
- Node scaling: DEV = 1-20 nodes, PROD = 3-20 nodes minimum

One liner: I implement environment isolation through branch-based state separation and environment-specific variable files. A developer pushing to the dev-* branch triggers Terraform to create infrastructure in the dev state. Also workspace based approach in cloud using the below environment variables, thus having states for both dev in gitlab and qa and prod in remote cloud.

```hcl

  WORKSPACE: ci-$CI_PROJECT_NAME-$CI_COMMIT_REF_SLUG
  ENVIRONMENT: $CI_PROJECT_NAME-$CI_COMMIT_REF_SLUG
```

## Creating Reusable and Maintainable Modules


### Example 1: The IAM Pattern (2-module separation)

```
iam_exchange_role (creates roles)
    ↓
iam_exchange (creates policies+attaches the policy to roles)
```

**Why it's maintainable**:

- Decoupled: Can change policies without recreating roles
- Reusable: 15+ services (adminservices, reporting, topology, batch, etc.) reuse the same module structure
- Extensible: Adding a new service = add module call + one IAM attachment

**One liner**: I've architected the IAM layer with separation of concerns: one module creates bare roles with trust policies, another module generates service-specific policies, and attaches them to the roles. This design allows us to independently update policies without role churn, and new services can be added by simply defining their ARN requirements and policy attachments.

### Example 2: Data-Driven S3 Bucket Module

```hcl
module "s3" {
  for_each = {
    for bucket in local.all_buckets_list : bucket.key => bucket
    if lookup(bucket, "create", false) == true
  }
  bucket_name = each.value.name
  # ... common config applied to all
}
```

**Why it's maintainable**:

- Single module definition, multiple instances via for_each
- Bucket list defined in locals (one source of truth)
- Common tags, CORS, lifecycle policies applied consistently
- Easy to add/remove buckets without duplicating resource blocks

**One liner**: I've implemented data-driven infrastructure using for_each loops. Instead of copy-pasting S3 bucket definitions, I define a list of bucket configurations and let Terraform generate resources dynamically. This ensures all buckets follow the same security standards (public access blocked, consistent tagging, RPO settings) and makes it trivial to onboard new storage requirements.

### Example 3: Conditional Service Provisioning

```hcl
module "step_functions" {
  count = contains(var.create_services, "reporting") ? 1 : 0
  # ...
}

module "neptune" {
  count = var.create_neptune_resources && contains(var.create_services, "topology") ? 1 : 0
  # ...
}
```

**Why it's maintainable**:

- Services can be enabled/disabled via variables
- No need to modify code, just tfvars
- Prevents unnecessary cost and resource churn

**One liner**: I've built conditional logic into modules so infrastructure adapts to input variables. For instance, if a client doesn't need the reporting service, we simply set reporting = false in their tfvars and the step functions, lambdas, and related IAM policies are never created. This supports multi-tenant deployments and cost optimization.

## Understanding of Immutable Infrastructure Principles

- Example: If EKS cluster config changes, the entire cluster is replaced (not modified)

- Blue-green patterns: Old infrastructure exists until new is validated, then traffic switches

**One liner**: I design infrastructure assuming resources are immutable. When a configuration changes, Terraform destroys the old resource and creates a new one rather than modifying it in place. This prevents configuration drift and ensures consistency. For critical components like the EKS cluster, I implement careful planning—running terraform plan before apply to verify that destructive changes happen only when intended.

### Handling Immutability Challenges

Data persistence (breaks immutability):

- RDS databases and S3 buckets aren't destroyed even if removed from code
- You use deletion_protection_enabled = var.flavour == "PROD" ? true : false
- MongoDB with PrivateLink prevents accidental destruction

**One liner**: While compute is ephemeral and immutable, stateful resources like databases need special handling. I implement deletion protection for production databases and use separate backup strategies. S3 buckets have versioning and RPO settings to survive infrastructure recreation. Data and infrastructure are decoupled—infrastructure can be immutable while data persists.

## Explain your Terraform experience

I've architected and maintained infrastructure-as-code for a 30-microservice platform using Terraform. Here's my experience:

**HCL & Modules**: I've built 15+ reusable modules with semantic versioning, published to a private registry. Each module has a single responsibility—e.g., 
iam_exchange_role creates roles, iam_exchange creates policies. This separation lets us modify policies without role churn and support 15+ services from the same module structure.

**State Management**: I use GitLab's HTTP backend for dev with automatic locking/ S3 backend for the qa and prod. Each environment (dev/qa/prod) gets its own state file keyed by branch name/environment name, ensuring isolation. Sensitive outputs are marked as such to prevent accidental exposure.

**Immutable Infrastructure**: I design for immutability. when config changes, resources are recreated rather than modified. This prevents drift. For persistent data (RDS, S3), I implement deletion protection and separate backup strategies. All changes flow through git and CI/CD, creating an immutable audit trail."




