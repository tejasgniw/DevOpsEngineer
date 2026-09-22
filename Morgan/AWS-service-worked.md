# Services Worked On

- VPC / Networking: Designed the VPC topology (public, private/app, optional data subnets), route tables, IGW, NAT gateways, security groups, and VPC endpoints/PrivateLink to keep sensitive traffic internal.

- EKS (Kubernetes): Provisioned cluster and node groups, configured IAM OIDC provider and roles, wired subnets/SGs, and enabled autoscaling and node sizing policies.

- ALB / Gateway (ELB): Deployed internet-facing ALB in public subnets, attached TLS certs, and routed external traffic to EKS ingress.

- ACM (Certificates): Issued/managed TLS certificates used by the ALB/gateway.

- Route53 / DNS: Created DNS records and wildcard subdomains for the gateway using zone outputs.

- RDS (PostgreSQL): Provisioned DB instances in data subnets with restricted SGs, backup/deletion-protection policies, and enhanced monitoring integration.

- MongoDB Atlas / PrivateLink: Integrated Atlas either via PrivateLink interface endpoints (preferred) or through NAT egress; managed host/endpoint configuration for pods.

- AWS MQ (RabbitMQ): Deployed managed brokers inside private subnets, provided users/credentials via secrets.

- S3 & CDN: Configured S3 buckets (storage) and CDN (edge delivery); optionally use VPC endpoints to avoid internet egress.

- Secrets Manager: Stored DB and service credentials; consumed by workloads using IAM roles/service accounts.

- IAM & Security Controls: Built role/policy modules, role-to-service mappings, least-privilege policies, and cross-account/trusted-account configs.

- Privatelink / VPC Endpoints: Created interface endpoints for managed services to avoid public internet and ensure private connectivity.

- Neptune, SageMaker, Batch, Step Functions: Provisioned graph DB, notebooks, batch compute and serverless workflows in private subnets for data-processing use cases.

- CloudWatch / Monitoring & Logs: Enabled metrics, logs, and enhanced RDS monitoring; wired monitoring roles and export settings.

## Interview-ready 2–3 sentence summary

"I built and maintained the AWS infrastructure using Terraform modules: I designed a secure VPC with public, private and data subnets, NAT/IGW, SGs and PrivateLink; deployed EKS for workloads and an ALB+ACM for ingress; and provisioned stateful services (RDS, Neptune, managed RabbitMQ, Mongo Atlas) inside private networks. I also implemented IAM roles/policies, secrets management, VPC endpoints, and monitoring to enforce least-privilege, keep traffic internal where possible, and support GitOps-driven app deployments."