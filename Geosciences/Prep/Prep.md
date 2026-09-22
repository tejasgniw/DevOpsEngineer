# Terraform

## General

### Providers

Terraform providers are plugins that act as translators between Terraform and various infrastructure APIs (cloud, SaaS, etc.), enabling the management of resources like virtual machines, networks, and databases. Maintained largely through the Terraform Registry, they are declared in code, configured via provider blocks, and installed using terraform init.

### Resources vs data sources

In Terraform, the key difference is that resources define and manage the full lifecycle (create, update, delete) of infrastructure objects, while data sources are read-only references used to fetch information about existing infrastructure or external data not managed by the current configuration.

```hcl
resource "aws_instance" "example" {
  # ... arguments to create the instance
}
```

```hcl
data "aws_vpc" "existing" {
  filter {
    name = "is-default"
    values = ["true"]
  }
}
```

### Variables, locals, outputs

In Terraform, variables, locals, and outputs are all mechanisms for managing and reusing values, but they serve different purposes related to data flow and scope within your configuration

- Variables: Input variables act as parameters for your Terraform module, allowing you to pass external values into the configuration.

- locals: Local values are like temporary, internal variables or helper values, computed within a module to avoid repeating complex expressions.

- outputs: Output values serve as the return values of a Terraform module, exposing specific data about the infrastructure it has created.

### Modules (how & why)

- Terraform modules are containers for multiple resources used together, enabling code reusability, standardization, and encapsulation of complex infrastructure.

- They allow teams to manage infrastructure as code (IaC) by organizing, sharing, and calling pre-built configurations, reducing duplication and improving maintainability across projects.

### State

Terraform state (terraform.tfstate) is a critical JSON file that maps real-world infrastructure resources to your configuration, allowing Terraform to track, update, or destroy managed resources. It acts as a "source of truth," enabling terraform plan and apply to identify changes.

- remote state(s3): Using remote backends (like S3 with DynamoDB) prevents concurrent modifications that can corrupt the state. Only one user at a time will make changes.

- state lock: Terraform will lock your state for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state.

### plan vs apply vs validate

- terraform plan is a read-only command that previews proposed changes to your infrastructure without making any modifications, acting as a "dry run". In contrast, terraform apply is the command that executes those proposed changes, creating, updating, or destroying real infrastructure resources.

- terraform validate is the fastest and safest check, used to verify that your configuration files are syntactically valid and internally consistent (e.g., correct argument names and types, valid module paths).

### Dependency handling (depends_on)

- In Terraform, the depends_on meta-argument is used to handle explicit dependencies, instructing Terraform to complete all actions on the specified dependency object(s) before performing actions on the object declaring the dependency.

- It should only be used as a last resort when dependencies cannot be automatically inferred by Terraform through resource references (implicit dependencies)

## Security aspects

- No hardcoded secrets

### Use of

- Environment variables

- Key Vault / Secrets Manager

- Least privilege IAM: Least privilege in IAM (Identity and Access Management) is a security principle granting users, systems, or processes only the minimum permissions necessary to perform their specific, intended tasks.

- Secure backend configuration: Secure Terraform backend configuration involves storing state files remotely with encryption, versioning, and state locking to prevent concurrent modification and data exposure. Best practices include using encrypted S3 buckets with DynamoDB for locking (AWS), using Azure Blob Storage, or using HCP Terraform, while avoiding hardcoded secrets in the .tf file.

# Kubernetes

## General

### Pods vs Deployments vs Services

Pods are the smallest, ephemeral, container-running units. Deployments manage the desired state, scaling, and lifecycle of multiple identical Pods. Services provide stable networking and load balancing to expose Pods to internal or external traffic.

### ConfigMaps vs Secrets

ConfigMaps store non-sensitive, plaintext configuration (e.g., config files, ports, URLs). Secrets are used for sensitive data (e.g., passwords, keys, tokens), providing Base64 encoding, potential encryption at rest, and stricter RBAC controls.

### Ingress vs LoadBalancer

LoadBalancer provides a single, dedicated external IP at the transport layer (Layer 4) for one specific service, while Ingress acts as a "smart router" at the application layer (Layer 7) to manage HTTP/HTTPS traffic to multiple services through a single, cost-effective entry point.

### Namespaces

Namespaces provide a mechanism for isolating groups of resources within a single cluster.

### Resource limits & requests

Kubernetes resource **requests** define the minimum guaranteed amount of CPU and memory a container needs, used by the scheduler to place the Pod on a suitable node. Resource **limits** define the maximum amount a container can consume, enforced at runtime to prevent a single container from monopolizing resources. 

### Rolling update

A rolling update allows a Deployment update to take place with zero downtime. It does this by incrementally replacing the current Pods with new ones. The new Pods are scheduled on Nodes with available resources, and Kubernetes waits for those new Pods to start before removing the old Pods.

## Security aspect

### RBAC, Service Accounts

- **Role-Based Access Control (RBAC)**: In Kubernetes, RBAC manages who (human users or processes) can interact with the Kubernetes API and what actions they can perform (e.g., get, create, delete pods, secrets, services).
Who can do what.

#### Key Components

- Role/ClusterRole: A set of permissions defined as rules. A Role is namespace-specific, while a ClusterRole is cluster-wide.

- RoleBinding/ClusterRoleBinding: An association that grants the permissions defined in a Role or ClusterRole to a user, group, or ServiceAccount.

- **Principle**: RBAC helps enforce the principle of least privilege (**POLP**) by ensuring users and processes only have the minimum permissions necessary to function, thereby reducing the impact of security breaches.

- **ServiceAccount**: A ServiceAccount is a non-human user account intended for processes running within a pod to authenticate to the Kubernetes API server. Pod identity.

Function: Pods use ServiceAccounts to obtain the necessary credentials (API tokens) to interact with other Kubernetes resources or the API itself.

Integration with RBAC: RBAC is used to define the permissions associated with a ServiceAccount. A RoleBinding is created to link a specific Role (containing permissions) to a ServiceAccount within a namespace.


### NetworkPolicies

A NetworkPolicy acts as a firewall for pods, controlling the network traffic flow between them and other network endpoints within the cluster. Pod-to-pod traffic rules.

### Pod security (non-root containers)

To enhance Kubernetes pod security, you should enforce non-root container execution using the securityContext in your pod specifications. This limits the potential damage if a container is compromised, as the process runs with restricted privileges. 


# CI/CD Pipelines (Security-aware automation)

## General 

```
Build → test → scan → deploy
```

### CI vs CD

CI (Continuous Integration) focuses on frequently merging code changes into a central repository, automatically building and testing to detect defects early. CD (Continuous Delivery/Deployment) extends this by automating the release process, ensuring code is either automatically deployed to production (Deployment) or consistently ready for manual release (Delivery). CI reduces integration issues, while CD speeds up, reliably, and automates software delivery. 

### GitOps basics (ArgoCD is a plus)

## Security in CI/CD (DevSecOps core)

### SAST vs DAST

SAST (Static Application Security Testing) and DAST (Dynamic Application Security Testing) are complementary security methodologies. SAST is a white-box "inside-out" approach analyzing source code, bytecode, or binaries at rest for vulnerabilities early in development. DAST is a black-box "outside-in" approach testing running applications for exploitable vulnerabilities, simulating external attacks. 

- SAST scans code without executing it; DAST tests the application in a running, compiled state.

### Dependency scanning vs container scanning

Dependency scanning analyzes your project and tells you which software dependencies, including upstream dependencies, have been included in your project, and what known risks the dependencies contain. Container scanning analyzes your containers and tells you about known risks in the operating system's (OS) packages.

### Secrets scanning

Secrets scanning is an automated security process that searches code repositories, commits, and configurations for exposed sensitive data like API keys, passwords, and tokens.

# Container & Image Security

## General

Implementing Dockerfile best practices such as **multi-stage** builds, using **minimal base images**, and running as a **non-root user** significantly enhances container security, efficiency, and size.

- Prefer specialized, minimal images like those from Alpine Linux, which are small and security-focused.

- Create a dedicated user and group within your Dockerfile using RUN groupadd and useradd instructions.

- multi-stage technique separates the build-time environment (which requires compilers, testing tools, and development dependencies) from the minimal run-time environment. Only the necessary final artifacts are copied to the final, slim image.

## Security

- Docker image vulnerabilities are security weaknesses within a container image's operating system packages, libraries, or configurations that attackers can exploit.

- Code signing is the process of applying a cryptographic signature to software artifacts, such as Docker images, to verify their integrity and authenticity. By signing an image, you ensure that it has not been altered since it was signed and that it originates from a trusted source.

- Runtime security basics: Runtime security is the technology that provides protection to running processes, wherever they are executed. Runtime security is a vital component in cybersecurity — especially in the cloud — protecting your applications, infrastructure, data, and users from malicious code and exploits.


# Cloud Basics (Azure + AWS)

## Focus on security fundamentals

### IAM (roles, policies)

AWS IAM Roles and Policies manage access to cloud resources. Policies are JSON documents defining permissions (allowed/denied actions and resources), while roles are temporary identities assumed by users or services to gain these permissions without permanent credentials. Roles are ideal for cross-account access and service-to-service tasks.

## Networking basics:

### VPC / VNET

A Virtual Private Cloud (VPC) is a secure, isolated, private network hosted within a public cloud provider's infrastructure (e.g., AWS, Google Cloud). It combines public cloud scalability with private network security, allowing organizations to define IP address ranges, create subnets, and configure route tables and gateways. 

### Subnets

A subnet (subnetwork) is a logical, segmented subdivision of an IP network that improves network performance, security, and efficiency by reducing broadcast traffic and grouping devices.

### Security Groups / NSGs

Security groups act as virtual firewalls to control inbound and outbound network traffic for cloud resources (like AWS EC2 or Azure VMs) or to manage user permissions in IT environments (like Active Directory/M365).

### Storage security

Storage security protects digital data, storage devices, and infrastructure from unauthorized access, theft, or corruption, both in the cloud and on-premises. It involves implementing access controls (RBAC), encryption, and monitoring to defend against ransomware and breaches. Key practices include using immutable storage, ensuring secure data destruction, and following the 3-2-1 backup rule.

### Shared responsibility model

The AWS Shared Responsibility Model divides security duties: AWS manages security of the cloud (physical infrastructure, hardware, virtualization), while customers manage security in the cloud (guest OS, application patching, data, network configuration, and IAM).


# Cybersecurity Basics (High-signal, not deep theory)

## Identity & Access

- Authentication vs authorization

- IAM best practices

- Least privilege (POLP)

- RBAC (cloud + K8s)

## App & Infra Security

- Secrets management

- TLS basics: Transport Layer Security (TLS) is a cryptographic protocol that ensures data privacy, integrity, and authentication between communicating applications over a network, most commonly seen as HTTPS on the web. It succeeds SSL, employing a "handshake" to authenticate servers via digital certificates and encrypt data using symmetric/asymmetric cryptography to prevent eavesdropping and tampering. 

- Encryption at rest vs in transit: Encryption at rest protects stored data (files, databases) from unauthorized access or physical theft using methods like AES-256. Encryption in transit secures data moving across networks (emails, API calls) against interception using TLS/SSL or VPNs.

- Secure defaults: refers to designing products and services that are inherently secure "out of the box" without requiring extensive user configuration.

## DevSecOps mindset

- Shift left security:  is the practice of integrating security testing and practices into the earliest stages of the software development life cycle (SDLC)—planning, design, and coding—rather than waiting until testing or deployment.

- Threat modeling:  is a structured, proactive engineering technique used to identify, analyze, and mitigate potential security threats, vulnerabilities, and risks **during the design phase** of software, systems, or networks. By adopting an attacker's perspective, it allows teams to prioritize countermeasures before code is written, reducing risks and strengthening security. 

- Supply chain security (dependencies, images)


# Observability & Reliability

## Logging vs Metrics vs Tracing

- Logging: Detailed, time-stamped records of discrete events that occurred within an application or system, often in plain text or structured JSON format.

- Metrics: Numeric measurements of system health and performance collected over time and aggregated into time-series data. Examples include CPU usage, memory utilization, request rates, and response times.

- Tracing: A way to follow the path of a single request or transaction as it travels through a distributed system or microservices architecture. A trace is composed of multiple "spans," each representing an operation within a service.


# Linux & Troubleshooting

## Basics to refresh

- File permissions: ls -al

- Networking commands: ifconfig, hostname, ping, netstat

- Process inspection

- Logs
