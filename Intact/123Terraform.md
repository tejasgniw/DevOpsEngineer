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




