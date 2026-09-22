# 1. If AWS already provides ECS, why would you use Kubernetes (EKS)?

ECS is a very good choice when the team wants a fully managed container platform with minimal operational overhead.

Kubernetes becomes useful when **organizations need portability across clouds**, **advanced orchestration features**, or a large ecosystem of tools like service meshes and GitOps platforms.

In many cases ECS is simpler, but Kubernetes is chosen when teams need more flexibility or already have Kubernetes-based tooling. 

EKS for Service mesh, GitOps tools, Portability across clouds, Third-Party Integration, Control & customization(Granular scheduling, CRDs, Advanced AutoScaling)


# 2. If AWS already provides CloudFormation, why use Terraform?

Terraform provides a **multi-cloud abstraction** and a **very strong module ecosystem**, which makes it **easier to reuse infrastructure** patterns across environments.

It also has a **simpler workflow** for teams that manage infrastructure across different cloud providers.

CloudFormation integrates very well with AWS services, but many teams **choose Terraform for portability and modular infrastructure design**.


# 3. What problem does GitOps actually solve?

GitOps makes Git the **single source of truth** for infrastructure and application configuration.

Instead of pushing deployments manually, the system continuously reconciles the cluster state with what is defined in Git.

This improves auditability, rollback capability, and deployment consistency.


# 4. If one Kubernetes node dies, what happens?

When a node fails, the Kubernetes control plane marks the node as NotReady.

The scheduler then reschedules affected pods to healthy nodes in the cluster, assuming sufficient capacity exists.

If cluster autoscaling is enabled, additional nodes may be provisioned automatically.


# 5. What are the biggest security risks with Docker containers?

Some common risks include using vulnerable base images, running containers with excessive privileges, and embedding secrets directly in images.

To mitigate this we use image scanning, minimal base images, and secret management tools like AWS Secrets Manager or Vault.


# 6. What should you monitor first in production?

In production systems I focus first on key signals like **error rates**, **latency**, **traffic volume**, and **system saturation**.

These metrics quickly indicate whether the system is healthy or experiencing performance issues.


# 7. What happens if Terraform state is lost?

Terraform state is critical because it **maps real infrastructure to the Terraform configuration**.

That's why we store state remotely, typically in S3 with versioning enabled and DynamoDB locking.

If the state were lost, recovery would involve restoring from versioned backups or importing resources back into Terraform state.


# 8. If you join tomorrow, what is the first thing you would want to understand about our infrastructure?

I would first try to understand the **architecture**, **deployment workflows**, and **operational challenges** the team currently faces.

That helps identify where improvements in automation, reliability, or cost optimization can bring the most value.