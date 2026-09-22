# Summary

DNS points to static EIPs → EIPs belong to an NLB (L4) → NLB forwards raw TCP to an ALB (L7) → ALB terminates TLS, applies WAF, and routes HTTP to EKS.

**Interview explanation**:

“We front the ALB with an NLB to get static IPs. DNS resolves to EIPs attached to the NLB. The NLB forwards raw TCP 443 to the ALB, which terminates TLS, applies WAF, and routes HTTP traffic into EKS running in private subnets.”



![alt text](<ChatGPT Image Feb 4, 2026, 02_09_14 PM.png>)

Internet → Route53

EIPs → NLB (public subnets)

ALB (public subnets)

EKS (private app subnets)

NAT for egress


# Complete explanation:

Each layer solves a different AWS limitation.

1) Why this architecture exists (before the flow)
Why not just an ALB?

Because: ALB does NOT have static IPs

Enterprises want static IPs for:

- firewalls
- partner allowlists
- legacy integrations

2) Why not just an NLB?

Because: NLB is Layer 4 only

No:
- TLS termination (advanced)
- HTTP routing
- WAF
- path/host rules

So the solution is:

👉 NLB for static IPs + ALB for smart HTTP routing (This is a very common enterprise AWS pattern.)

## Step-by-step flow (DNS → Pod)

1️⃣ DNS (Route53)

What happens, for e.g. https://app.example.com

Route53 resolves this hostname

- Terraform mapping (module.domain_name)

**Important detail**

- Route53 does NOT point to the ALB

- It points to static EIPs


2️⃣ EIPs (Elastic IPs)

What they are: One static public IP per public subnet / AZ

Why? Guarantees stable IPs even if load balancers restart

- Terraform mapping (aws_eip.this, module.gateway.ingress_ips)

- These EIPs are attached to the NLB, not the ALB.

3️⃣ NLB (Network Load Balancer – L4)

What it does: Listens on TCP/443

- No TLS termination, No HTTP logic, Just forwards bytes

- Terraform

```
    aws_lb.gateway
    type = "network"
```

- Key concept: The NLB is basically a TCP router with static IPs

4️⃣ NLB → ALB (this is the “confusing” part)

The NLB forwards traffic to a target group of type alb.

Yes — an ALB can be a target of an NLB.

- Terraform

```
aws_lb_target_group.alb
target_type = "alb"
```

What flows:

- Raw TCP stream
- TLS handshake is NOT terminated yet

5️⃣ ALB (Application Load Balancer – L7)

Now we enter the smart layer:

ALB responsibilities:

- TLS termination (ACM cert)
- WAF
- HTTP routing (host/path rules)
- Security groups

Terraform

```
aws_lb.cluster
aws_lb_listener.cluster (443)
aws_wafv2_web_acl_association
```

At this point:

- HTTPS → decrypted to HTTP
- Request headers are visible
- Routing decisions happen

6️⃣ ALB → Target Group → EKS

ALB forwards to:

Target group pointing to:

- EKS node ports or
- Pod IPs (IP mode)

Terraform

```
aws_lb_target_group.cluster
module.eks
```

Network reality

- ALB lives in public subnets

- EKS nodes live in private app subnets

SG rule allows:

```
ALB SG → EKS node SG
```

7️⃣ Kubernetes ingress → Service → Pods

Inside the cluster:

- Ingress controller (NGINX / ALB ingress)
- Routes to Service
- Service routes to Pods

This is pure Kubernetes, deployed via GitOps, not Terraform.


8️⃣ Response path

Response goes back the exact reverse path:

```
Pod → Node → ALB → NLB → EIP → Client
```

Outbound traffic (important distinction)

Pods do NOT go back through ALB/NLB for egress.

Instead:

```
Pod → NAT Gateway → Internet
```

Unless:

- VPC endpoints
- PrivateLink (Mongo Atlas, S3, etc.)