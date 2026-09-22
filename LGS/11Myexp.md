# GCP

In this project, GCP is used exclusively as the identity and authentication control plane. We use Google Cloud Identity Platform as an OIDC-compliant Identity Provider for user authentication, with Cloud Functions (acting as policy enforcement points during user lifecycle events). The application itself runs on AWS EKS, consuming GCIP-issued tokens. CI/CD authentication to GCP is handled securely using Workload Identity with GitLab via OIDC, avoiding static credentials.

# AWS

- We front the ALB with an NLB to get static IPs. DNS resolves to EIPs attached to the NLB. The NLB forwards raw TCP 443 to the ALB, which terminates TLS, applies WAF, and routes HTTP traffic into EKS running in private subnets.

## What the code does: 

modules/gateway/main.tf allocates one aws_eip per public subnet and maps each EIP into the NLB subnet_mapping so the NLB has static IP addresses (those EIPs are what Route53 resolves to).

## Why that matters (reasons)

- Fixed/public IPs for DNS and allowlists: some clients, partner firewalls, or webhook providers require an IP (or IP range) to be whitelisted — EIPs give stable addresses you can share.
- Predictable outbound/ingress presence: ALB DNS records’ underlying IPs can change; putting an NLB with EIPs in front guarantees stable L4/IPv4 endpoints.
- TLS termination + WAF behind: the NLB forwards raw TCP(443) to the ALB; ALB can terminate TLS and apply WAF while the NLB preserves the static IP entry point.
- Legacy or constrained clients: devices or on‑prem appliances that only support IPs (not ALIAS/DNS names) benefit from static EIPs.

## AWS infra

- I built and maintained the AWS infrastructure using Terraform modules: I designed a secure VPC with public, private and data subnets, NAT/IGW, SGs and PrivateLink; deployed EKS for workloads and an ALB+ACM for ingress; and provisioned stateful services (RDS, Neptune, managed RabbitMQ, Mongo Atlas) inside private networks. I also implemented IAM roles/policies, secrets management, VPC endpoints, and monitoring to enforce least-privilege, keep traffic internal where possible, and support GitOps-driven app deployments.

- Step Functions is a serverless orchestration service that automatically chains multiple Lambda functions together without writing orchestration code. When a user uploads a report file to S3, Step Functions (already has **definition.json**) triggers a state machine that executes our Lambdas sequentially—first **throw_on_error** state validates the file, then **update_report_attachments** state stores it, followed by **update_report_status** state marking it as processed, and finally **update_report_warnings** state logs any issues. If any Lambda fails, Step Functions automatically retries it or jumps to an error handler, eliminating the need to write retry logic in code and giving us a complete audit trail of each report's processing journey.

or in short

- Step Functions orchestrates our report processing by trigerring a state machine that automatically executes four Lambdas in sequence—validate, store, update status, and log warnings according to the definition.json —without us writing any orchestration code. If one fails, Step Functions automatically retries or handles the error; we get a complete execution history showing exactly where each report succeeded or failed.