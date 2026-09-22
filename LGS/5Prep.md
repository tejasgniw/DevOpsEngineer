# K8s cum deployment strategies

Automate build, test, deployment, and rollback processes


## deployment strategies e.g. blue-green, canary, rolling updates

### Rolling updates: 

Gradually replaces old pods with new ones.

**How it works**:

- New pods start
- Old pods terminate gradually
- No downtime

```
Old:  [v1][v1][v1][v1]
New:  [v2][v2][v1][v1]
Then: [v2][v2][v2][v2]
```

**Best for**: Regular production updates with zero downtime.

### Blue-Green Deployment

Run two environments (Blue = old, Green = new) and switch traffic instantly. Ops/aws-seed has it.

**How it works**:

- Deploy new version alongside old
- Switch service/load balancer to new version

```
Blue (v1)  ← live
Green (v2) ← idle

Switch traffic → Green becomes live
```

**Best for**:

- Instant rollback
- Major version changes
- Risky updates

Rollback: Just switch traffic back.

### Canary Deployment

Release new version to a small percentage of users first.

**How it works**:

- 10% traffic → v2
- 90% traffic → v1

Monitor metrics: Gradually increase traffic

```
Users
 ├─ 90% → v1
 └─ 10% → v2
```

**Best for**:

- Testing in production safely
- Feature validation
- High-risk deployments

**Often implemented with**:

- Service mesh (Istio)
- Ingress controller
- Weighted routing (ALB / NGINX)


| Strategy   | Downtime | Risk     | Rollback Speed | Use Case         |
| ---------- | -------- | -------- | -------------- | ---------------- |
| Rolling    | No       | Low      | Moderate       | Standard updates |
| Blue-Green | No       | Very Low | Instant        | Major releases   |
| Canary     | No       | Lowest   | Gradual        | Feature testing  |


**One liner**: We automate build and test using CI pipelines that build Docker images and push them to a registry. For deployments, we use GitOps with tools like ArgoCD to update Kubernetes manifests. Kubernetes supports rolling updates by default, and for more controlled releases we use blue-green or canary strategies. Rollbacks are handled using deployment revision history or by switching traffic back in blue-green setups.

## scalability

**One liner**: Horizontal Pod Autoscaler automatically adjusts the number of pods in a deployment based on CPU or custom metrics. It operates at runtime and works alongside deployment strategies like rolling or canary deployments. In EKS, HPA scales pods while the Cluster Autoscaler ensures sufficient EC2 nodes are available.

# Jenkins vs Gitlab

| Jenkins              | GitLab CI/CD                         |
| -------------------- | ------------------------------------ |
| Jenkinsfile          | `.gitlab-ci.yml`                     |
| Declarative pipeline | YAML-based pipeline                  |
| Scripted pipeline    | Job scripts using bash               |
| Stages               | `stages:`                            |
| Steps                | `script:`                            |
| Plugins              | Built-in integrations + CI templates |
| Agents               | GitLab Runners                       |
| Multibranch pipeline | Branch rules (`rules:`)              |


One liner: In Jenkins, pipelines are defined using Jenkinsfiles with declarative or scripted syntax and extended using plugins. In GitLab, pipelines are defined declaratively in a .gitlab-ci.yml file. Complex multi-stage pipelines are built using stages, job dependencies, artifacts, and rules. GitLab integrates many features natively, reducing the need for external plugins. Runners execute jobs similarly to Jenkins agents.

# Git strategies

Git: branching strategies e.g. GitFlow, trunk-based development

## GitFlow

GitFlow is a structured branching model using dedicated branches for features, releases, and hotfixes.

Main Branches:

- main (production-ready code)
- develop (integration branch)

Supporting branches:

- feature/*
- release/*
- hotfix/*

```
main
  ↑
release/*
  ↑
develop
  ↑
feature/*
```

**How It Works**: 

- Create feature branch from develop
- Merge feature → develop
- Create release branch
- Merge release → main
- Tag production version
- Hotfix branch from main if urgent fix needed

**Best For**:

- Enterprise systems
- Scheduled releases
- Multiple environments
-Strict release control

**Downsides**:

- Complex
- Slower deployments
- Merge conflicts increase
- Not ideal for continuous delivery


## Trunk-Based Development

Trunk-based development means developers commit small, frequent changes directly to the main branch (trunk).

```
main (trunk)
  ↑
short-lived feature branches (optional)
```

**How It Works**:

- Developers branch briefly (optional)
- Merge to main quickly (same day ideally)
- CI runs automatically
- Deploy continuously

**Often combined with**:

- Feature flags
- Strong automated testing

**Best For**:

- CI/CD environments
- Microservices
- Fast-moving teams
- DevOps culture

**Downsides**

- Requires strong automation
- Needs high test coverage
- Discipline required


| Feature           | GitFlow         | Trunk-Based  |
| ----------------- | --------------- | ------------ |
| Complexity        | High            | Low          |
| Release frequency | Scheduled       | Continuous   |
| Branch lifetime   | Long-lived      | Short-lived  |
| Best for          | Enterprise apps | Modern CI/CD |
| Merge conflicts   | More common     | Reduced      |
| DevOps alignment  | Moderate        | Very High    |



# Grafana, Prometheus, Elastic search

## Prometheus, Grafana

- Configured kube prometheus stack for our internal dev/qa team to determine the resource management of Pods and containers to reduce internal costs. It has pre-build dashboards in grafana for the k8s cluster right of the bat.

- Built in house open-source grafana in order to remediate the vulnerabilities and use a specific licensed open source version.

# Elastic search

- Elastic search was configured in our cluster to find the traces/logs using the trace id, thus was very helpful while debugging.

## Summary 

Elasticsearch is managed via Terraform, using Elastic Agent and Fleet integrations, with configuration driven by variables, JSON templates, and dynamic policy generation. The setup is cloud-native and integrates with AWS and Kubernetes environments.

## Configuration Flow:

- Terraform provisions the Elastic Agent and Fleet policies.
- The agent is enrolled with Fleet using an enrollment token.
- The Fleet server pushes the integration configuration (what to collect, how to parse, etc.) to the agent.
- The agent collects logs/metrics and sends them to Elasticsearch.

## OpenTelemetry

 OpenTelemetry collects and exports observability data; Elasticsearch stores and indexes it for search, alerting, and visualization (often via Kibana). This integration provides end-to-end visibility into your applications and infrastructure.