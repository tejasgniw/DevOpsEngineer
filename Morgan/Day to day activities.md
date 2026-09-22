# Cloud

- **AWS**: Deployed and managed AWS infrastructure including EKS, RDS, IAM, S3, VPC, Lambda, Step Functions, Route53 and other services.

e.g. We front the ALB with an NLB to get static IPs. DNS resolves to EIPs attached to the NLB. The NLB forwards raw TCP 443 to the ALB, which terminates TLS, applies WAF, and routes HTTP traffic into EKS running in private subnets.

- **Azure**: Basic knowledge, but can adapt quickly as I have experience in AWS.

- **GCP**:  Used solely for identity and CI integration: it provides Firebase/GCIP for user authentication and lifecycle hooks (via Cloud Functions), and enables GitLab CI to securely access GCP resources using Workload Identity—while all application compute and data remain on AWS.


# Security

**Pipelines**: Implemented IaC-SAST, SAST, Kubesec, container scanning, and dependency scanning across projects based on security requirements.

- IaC-SAST: Scans infrastructure-as-code files (Terraform, Helm, K8s manifests) for misconfigurations and security anti-patterns before deployment.

- SAST: Analyzes application source code to detect vulnerabilities like SQL injection, XSS, and hardcoded secrets without running the code.

- Kubesec: Evaluates Kubernetes manifests for security best practices, ensuring safe container and cluster configurations.

- container scanning: Inspects Docker/OCI images for known OS and library vulnerabilities before deployment. E.g. Trivy, grype scanners on Gitlab

- Dependency Scanning: Checks project dependencies for known security issues in third-party libraries and recommends updates.

# Design, optimize & document

- Designed, optimized, and documented Terraform modules, Kubernetes deployments, and CI/CD pipelines to ensure scalable, secure, and efficient cloud infrastructure.

# IAC

- I used Terraform modules and Kubernetes manifests to define and deploy cloud infrastructure and services across AWS and GCP.

- Networking, compute, databases, and identity/authentication components were fully automated, version-controlled, and securely configured, enabling reliable and repeatable deployments.

# CI/CD automation

- Designed and automated secure end-to-end CI/CD pipelines, including Docker image builds, Helm chart packaging, OCI pushes, semantic versioning, Terraform module deployments, and multi-environment release workflows with integrated security scanning and vulnerability remediation.

# Incident management/Chaos engineering

- Led incident management and reliability efforts by proactively diagnosing and resolving production and non-production issues across Kubernetes clusters, CI/CD pipelines, and cloud infrastructure.

# DevOps tools

- AWS, Terraform, Kubernetes, Helm, Ansible, GitLab CI/CD, OCI, Docker, Semantic-release, MetalLB, Istio service mesh, Argo CD, Kind, Container scanning, Prometheus, Grafana, Elasticsearch, Pulsar, MQTT

# Troubleshooting on premises & cloud


# Observability


# Reduce toil automation, Architecture/Process improvements




