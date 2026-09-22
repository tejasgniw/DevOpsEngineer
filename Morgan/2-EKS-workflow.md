EKS usage

module.eks provisions a managed EKS control plane and EC2 worker node groups (inputs include instance_type_regular_node, regular_min, regular_max), so workloads run on real EC2 nodes inside the VPC private app subnets (subnets_ids_map = module.networking.private_app_subnet_ids_map).

Other modules consume module.eks outputs (e.g., module.eks.cluster, module.eks.node_group_role, module.eks.admin_role_arn) to attach IAM roles, give Batch nodes access, and wire cluster endpoints into other services (logging, monitoring, etc.).

**Networking**: EKS nodes live in the private app subnets provided by module.networking; they use the app subnet security groups. Pod/Node egress to the internet goes via NAT Gateways in public subnets; inbound traffic from the internet goes through the ALB/gateway in public subnets to the cluster.

**Summary**: EKS is used directly as the Kubernetes runtime; it is not merely a wrapper for existing EC2 instances — Terraform creates and manages the cluster and its node groups.
ArgoCD / GitOps deployment target

module.workload_cluster registers the EKS cluster in a cluster-registry (GitLab project) and writes a cluster secret/metadata that ArgoCD (the control-plane) consumes.

The repo’s README and workload_cluster docs explain that the cluster secret is installed into the control-plane ArgoCD (not into this workload cluster). The project uses an IRSA-based trust model (no plaintext kube credentials in the secret).

**Practically**: ArgoCD runs in a centralized control-plane cluster (managed outside this module). ArgoCD’s ApplicationSets reference the cluster-registry and annotations (populated by workload_cluster) to generate Applications that deploy manifests/charts into this EKS workload cluster.

So ArgoCD doesn’t deploy itself into this EKS cluster here — it deploys applications into this EKS cluster after workload_cluster registers it with the control-plane.
Key integration points to mention in interviews

module.workload_cluster supplies cluster endpoint, CA, admin_role_arn, and many annotated variables (image tags, hostnames, DB endpoints) so ArgoCD ApplicationSets can parametrize deployments per environment.

**Access model**: ArgoCD uses the cluster secret + IRSA (IAM roles / service accounts) to operate against the workload cluster securely.

Terraform ordering ensures networking → EKS → workload registration → GitOps-driven app deployment.