Traffic flow (concise, end-to-end)

- DNS resolution
Your browser requests the web hostname.
Route53 record created by module.domain_name resolves that hostname to the gateway's ingress addresses (the load‑balancer endpoints published by module.gateway.ingress_ips).

- Internet → Load balancer
The client connects to the load balancer (ALB/NLB) deployed by module.gateway in the public subnets (module.networking.public_subnet_ids_map).
TLS is terminated at the load balancer using the ACM cert from module.certificate (HTTPS).

- Load balancer → Target group
The load balancer forwards requests to its target group (module.gateway.cluster_target_group_arn).
The target group is configured to reach the EKS cluster: either the EKS node ports (node-based targets) or pod IPs (IP mode), depending on the ingress setup.

- Target group → EKS ingress / Service → Pods
The request arrives at EKS worker nodes running in private app subnets (module.networking.private_app_subnet_ids_map).
The cluster’s ingress controller / Service (installed into the cluster via GitOps) receives traffic and routes it to the appropriate pod(s) (web-app).
Security groups: ALB SG allows inbound from the internet; EKS node/pod SGs allow inbound from the ALB SG only.

- Response path and egress
Pods respond back over the VPC to the load balancer, which returns the response to the client.
Pods do not have public IPs; any outbound requests from pods (e.g., to external APIs) go through NAT Gateways in public subnets (unless VPC endpoints are used).

- Health checks, sticky sessions, and scaling
The load balancer uses health checks to keep only healthy targets in the target group.
Autoscaling in EKS adjusts node/pod capacity; ALB target group membership updates automatically.

- Files/outputs that wire this up

DNS A/records: module.domain_name → uses module.gateway.ingress_ips

Load balancer & target group: module.gateway (public subnets, cluster_target_group_arn, alb_security_group_id)

Cluster placement: module.eks → subnets_ids_map = module.networking.private_app_subnet_ids_map

Networking primitives: module.networking (public/private subnets, NAT/IGW, SGs)