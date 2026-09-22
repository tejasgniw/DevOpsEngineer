# High-level Architecture

- Root Module: modules/exchange composes many focused modules (networking, gateway, eks, postgresql_db, privatelink, mongo_cluster, aws_mqbroker, etc.) to build the platform.

- Networking responsibility: module.networking is the single source of truth for the VPC, subnets, route tables, and primary security groups; almost every other module consumes its outputs.

## VPC & Subnets

- VPC: A single VPC is created (or referenced) by module.networking.vpc_id and used by EKS, RDS, privatelink, Neptune, Step Functions, etc.

**Subnet types**:

- Public subnets: module.networking.public_subnet_ids_map — host internet-facing resources (ALB / gateway, NAT gateways, possibly bastion).

- Private app subnets: module.networking.private_app_subnet_ids_map — host EKS worker nodes, batch, workers, and other private services.

- Data subnets (optional): module.networking.data_subnet_ids_map — dedicated for stateful data services (RDS) with tighter controls; created only if var.create_data_subnet and certain DB flags are true.

**Routing basics**:

- Public subnets have a route to an Internet Gateway (IGW) for inbound/outbound internet traffic.

- Private subnets route outbound internet via NAT Gateway(s) in public subnets.

- Data subnets typically have restricted routes (no direct internet or controlled via NAT/endpoint).

**Security Groups & Network Controls**

module.networking provides maps of security group IDs:

- App subnet SGs: attached to EKS nodes and private workloads.

- Data subnet SGs: attached to RDS and other data services.

Security controls enforce:

- ALBs/gateway accept public traffic on HTTP(S) only.

- EKS and internal services accept traffic from ALB SG and other internal SGs.

- RDS is accessible only from allowed internal SGs (EKS, batch, admin IPs or bastion).

- There may also be VPC endpoints (PrivateLink) created by module.privatelink to keep traffic to external managed services (e.g., Mongo Atlas privatelink) inside AWS network.
Where key modules live relative to the VPC

- module.gateway (ALB / ingress layer): in public subnets, uses module.networking.public_subnet_ids_map and ACM certificate from module.certificate. Publishes ingress IPs consumed by module.domain_name.

- module.eks: worker nodes and control-plane networking use private app subnets and app_subnet_security_group_ids. Cluster API is reachable from management plane and workload_cluster registration.

- module.postgresql_db: RDS instances in data subnets with data_subnet_security_group_id. Controlled public access via var.postgresql_public_access_enabled.

- module.privatelink + module.mongo_cluster: set up interface endpoints / VPC connections so MongoDB Atlas (or other services) can be accessed privately without leaving VPC.

- module.aws_mqbroker: messaging broker (RabbitMQ) typically deployed in private subnets with internal security groups.

- module.s3, module.cdn: S3 is global but may be fronted by CDN and referenced by services; traffic to S3 can use VPC endpoints (if configured) to avoid internet egress.

**Ingress and Egress Flows (concrete examples)**

Web request from internet:

Client → DNS → NLB(static EIPs) → ALB/gateway in public subnets.
ALB forwards to EKS ingress (pods) in private subnets via NLB/ALB SG → pod/node SG.
Pods handle request and may call internal services (RDS, MQ, external APIs).
Pod accessing external internet (image pull, telemetry):
Pod → Node NAT route → NAT Gateway in public subnet → Internet. Egress IP observed is NAT GW.
Pod accessing RDS:
Pod → internal VPC network → RDS endpoint in data subnet (SG rules allow this).
App connecting to Mongo Atlas created in same infra:
If mongo_cluster + privatelink created, traffic uses PrivateLink endpoints (interface ENIs) — stays inside AWS network.
If not, pod connects to var.mongo_atlas_host over the internet (outbound NAT).

**Data plane isolation rules**

Stateful systems (RDS, Neptune, Mongo privatelink) are placed in private/data subnets and accept connections only from explicit SGs (EKS nodes, batch, specific roles).

Public access to databases is disabled by default; var.postgresql_public_access_enabled can open RDS to the internet (use carefully).

Secrets are stored in module.secrets (Secrets Manager); applications read them via IAM roles attached to service accounts or nodes.

**Deployment / Lifecycle & Ordering**

Terraform modules are wired with depends_on (e.g., module.gateway depends on module.networking) so the VPC and networking come up first.

**Typical order**:

- Provision VPC, subnets, NAT, IGW, SGs (module.networking).

- Create certificate (module.certificate) and DNS records (module.domain_name).

- Provision EKS (module.eks) and IAM roles (module.iam, module.iam_exchange_role, module.iam_exchange).

- Create data services (RDS, Neptune, Mongo privatelink, AWS MQ) once networking and IAM exist.

- Register workload cluster (module.workload_cluster) in GitLab for GitOps pipeline to deploy apps into EKS.

- After infra is up, application deployment is typically done via GitLab/GitOps (workload cluster registration + manifests), not Terraform.

**What stays inside the VPC vs what leaves**

- Stays inside:

Inter-service traffic (pods → RDS, pods → MQ, pods → Neptune) if services are in VPC or via PrivateLink.

Private endpoints created by module.privatelink.

- Leaves the VPC:

Outbound HTTP/HTTPS traffic from pods via NAT (unless VPC endpoints are used).

Connections to external managed services when PrivateLink is not configured (e.g., external Atlas host).

Traffic to S3/Cloud services unless VPC endpoints are configured.

### Interview-ready 2–3 minute summary (what to say)

"This Terraform root composes a modular AWS platform. Networking is centralized in module.networking, which creates a VPC with public, private (app) and optional data subnets; it also exposes security groups and subnet maps consumed by other modules. Internet traffic terminates at an ALB/gateway in public subnets, which forwards to EKS workloads in private subnets. Outbound traffic from private workloads uses NAT Gateways in public subnets. Stateful data services like RDS and Neptune live in dedicated data subnets with strict security groups; Mongo Atlas is integrated either via PrivateLink (interface endpoints) to keep traffic internal, or via a public host if PrivateLink isn't used. Terraform wiring enforces ordering: networking → certificate/DNS → EKS/IAM → data services → workload registration; application deployment is handled by GitLab/GitOps into the EKS cluster. Key security points: least-privilege SGs, private subnets for stateful systems, and PrivateLink to avoid exposing sensitive traffic to the internet."