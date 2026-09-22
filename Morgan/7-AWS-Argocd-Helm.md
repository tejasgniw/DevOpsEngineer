Architecture Overview: Helm Charts Deployment on ArgoCD in AWS
Based on my analysis of your project structure, here's how your system works:

1. Control Plane Setup (operations)
The Control Plane is the central cluster managed in operations:

Location: AWS EKS cluster in account 905418345845 (us-east-1)
Infrastructure: Deployed via main.tf
EKS cluster provisioned using gitlab.com/exfo/eks/aws module
Networking, IAM, ACM certificates configured
Route53 DNS management
2. ArgoCD Installation on Control Plane
ArgoCD itself is deployed as a Helm chart on the control plane:

Installation Module (main.tf:37-150):

module "argo" {
  source  = "gitlab.com/exfo/argocd/kubernetes"
  version = "~> 7.0"
  
  entrypoint_namespace         = "argocd"
  entrypoint_git_path          = "charts/control-plane"
  entrypoint_git_repo          = "https://gitlab.com/exfo/products/tandm/basecamp/platform/operations.git"
}

ArgoCD Configuration includes:

Authentication: SAML integration with AWS Identity Centre for SSO
Credentials: Deploy tokens for Git repo access (infra, apps, operations, environments, crds)
RBAC: Group-based access control with admin/readonly roles
Image Updater: ArgoCD Image Updater monitors ECR for image tag updates
3. ArgoCD Control Plane Templates
The templates contains the core GitOps definitions:

exchange.yaml - ApplicationSet
Uses ApplicationSet with cluster generator
Filters clusters with product: exchange and flavour: prod labels
Deploys the exchange-gitops Helm chart from ECR
Pulls configuration from cluster annotations (domain, AWS account, etc.)
exchange-envs.yaml - Application
Simple ArgoCD Application pointing to exchange-environments repo
Deploys all manifest files from that repository
Enables automatic sync with self-heal and prune
argo-ingress.yaml - Access
AWS ALB Ingress for ArgoCD UI at argo.<domain>
Routes both HTTP and gRPC traffic (for ArgoCD API)
TLS termination with ACM certificate
4. Helm Charts Deployment Flow

┌─────────────────────────────────────────────────────────────────┐
│                      Operations Repository                      │
│                  (Control Plane & ArgoCD Config)                │
│  - Terraform: EKS, IAM, Networking, ACM, Route53               │
│  - Helm Chart: control-plane with ApplicationSets              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   AWS EKS       │
                    │  Control Plane  │
                    │  (Kubernetes)   │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  ArgoCD Server  │
                    │  & Controller   │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
   ┌─────────┐         ┌──────────┐        ┌──────────────┐
   │exchange │         │  cloud   │        │ exchange-    │
   │-gitops  │         │ runtime  │        │  environments│
   │Helm     │         │ -gitops  │        │  Git Repos   │
   │Charts   │         │Helm      │        │              │
   │(ECR)    │         │Charts    │        │ ApplicationSet
   └────┬────┘         │(ECR)     │        │ generator    │
        │              │          │        │              │
        ▼              ▼          ▼        ▼
   ┌─────────────────────────────────────────────┐
   │  Dynamic Exchange Environments (EKS)        │
   │  - QA, Dev, Prod clusters                   │
   │  - Each with unique config via annotations  │
   └─────────────────────────────────────────────┘

5. Helm Chart Repositories
Core Charts in helm-charts:

exchange-gitops: Main application chart for Exchange deployments

Uses values.yaml
Contains all microservices, APIs, and UI configurations
cloud-runtime-gitops: Platform infrastructure chart

Uses values.yaml
Deploys: Istio, Prometheus, cert-manager, EBS CSI, External Secrets, etc.
All charts are:

Versioned using semantic versioning
Published to EXFO artifact registry (905418345845.dkr.ecr.us-east-1.amazonaws.com)
Released via CI/CD pipeline (.gitlab-ci.yml)
6. Environment Provisioning (exchange-environments)
exchange-environments uses GitOps for infrastructure:

One branch per environment (qa, pnc, dev, etc.)
Each branch has unique configs/ with:
backend.tfbackend - Terraform state location
variables.tfvars - Environment-specific variables
gitlab-tfvars.json - Sensitive secrets
The main.tf uses the exfo/exchange module which references lineup.json for component versions.

7. Platform Modules (platform-modules)
platform-modules provides reusable Terraform modules for:

AWS VPC & networking
EKS clusters
ArgoCD deployments
Databases & managed services
AWS OIDC for GitLab integration
8. AWS Account Structure

EXFO Centralized Solutions Account
├── Terraform state (S3 bucket)
└── Monitoring role: arn:aws:iam::010526245188:role/exchange-global-write-prometheus

Operations Account: 905418345845
├── Control Plane EKS cluster
├── ArgoCD server & controller
├── ECR repositories
│   ├── exchange/exchange-gitops (helm chart)
│   ├── exchange/cloud-runtime-gitops (helm chart)
│   └── exchange/* (app images)
└── Route53 DNS

Dynamic Environment Accounts (per env)
├── EKS clusters (QA, Prod, etc.)
├── RDS, MongoDB Atlas, Neptune databases
├── S3 buckets
└── Application workloads

9. Deployment & Promotion Workflow
Code Change: Commit to helm chart repo or application repo
CI/CD Pipeline: Release new chart version to ECR
Promotion: Update lineup.json in exchange-environments branch
Git Push: Trigger environment branch pipeline
Terraform: Updates cluster resources and configures ArgoCD targets
ArgoCD: Detects changes, syncs helm charts to target clusters
Self-Healing: Continuous reconciliation ensures desired state
This architecture provides GitOps-driven, multi-environment deployments with strong separation of concerns between the control plane and application environments.


# workload to application process

Progress: located the workload_cluster TF module, control-plane ApplicationSet templates, and ArgoCD/Terraform wiring. Next: ask if you want a diagram or a concrete example secret.

How changes register in ArgoCD — summary (step‑by‑step)

Terraform writes a cluster "secret" into the cluster-registry Git repo:
The workload_cluster Terraform module creates/updates a file in the GitLab project using gitlab_repository_file (it commits a Secret YAML).
See: main.tf (main.tf)
Secret shape and purpose:
The file is a Kubernetes Secret with:
label argocd.argoproj.io/secret-type: cluster (so ArgoCD treats it as a cluster entry)
product/flavour labels (e.g. product: exchange, flavour: dev) used by ApplicationSet cluster generators
annotations carrying environment-specific values (ALB ARNs, account/region, image tags, etc.)
stringData.config containing server (API endpoint), awsAuthConfig.roleARN, and TLS CA data — this is how ArgoCD connects/authenticates to the EKS cluster (IRSA/assume-role approach).
Example writer: main.tf (main.tf)
How the secret gets into the control plane argocd namespace:
The module commits to the cluster-registry GitLab repo (project id referenced in modules). That repository’s contents are then applied to the control‑plane (the project’s CI or a control‑plane ArgoCD Application syncs those files into the argocd namespace). The module’s README documents this pattern.
Look for the cluster-registry project in your GitLab (module references gitlab_project_id).
ArgoCD discovers and registers the cluster automatically:
ArgoCD reads cluster Secrets in its namespace (or is given them via the synced repo); secrets labelled argocd.argoproj.io/secret-type=cluster become entries in ArgoCD’s cluster list.
When a new/updated secret appears ArgoCD adds/updates that cluster (connects using the server URL + roleARN/IRSA info).
ApplicationSet generates Applications for that cluster:
Control-plane ApplicationSet templates use a cluster generator with matchLabels (e.g. product: exchange and flavour: dev) to select the cluster Secrets. Example: exchange-dev.yaml uses exactly that selector and passes Secret annotations into Helm values.
See: exchange-dev.yaml (exchange-dev.yaml)
Result: workloads are applied and kept in sync
The ApplicationSet produces per-cluster Applications (templated) that feed the annotation values into Helm/Kustomize values; ArgoCD then deploys and continuously reconciles those Applications onto the workload cluster.
If Terraform updates the secret (annotations or connection info), the changed Secret in the registry repo is re-synced to control plane → ArgoCD re-evaluates cluster list and re-renders ApplicationSets which can change the generated Applications (and thus deployments).
Key files to inspect

Workload-cluster writer (commits the Secret YAML): main.tf
Control-plane ApplicationSet that matches clusters and injects annotations: exchange-dev.yaml and exchange.yaml
ArgoCD Helm chart values / repo credentials and ingress: main.tf (module argo configuration)
If you want I can:

show a concrete example Secret (rendered) and the exact fields ArgoCD uses, or
locate the cluster-registry GitLab project pipeline that syncs files into the control plane (I can search the remote repo or trace the project id). Which would you prefer?