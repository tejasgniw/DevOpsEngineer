# 1. GitLab CI/CD — This Is the Core

They don’t want someone who knows CI/CD. They want someone who can design it from scratch.

**Be ready to explain:**

## How you design a multi-stage GitLab pipeline

```
stages:
  - test
  - security
  - deploy
  - publish

default:
  image: registry.gitlab.com/org/base-image:latest

variables:
  AWS_REGION: "us-east-1"
  TF_VAR_environment: $CI_COMMIT_REF_NAME


###### We include GitLab security components to automatically scan Terraform code for IaC vulnerabilities and detect secrets. This enforces security as part of the pipeline. ########

include:
  - component: gitlab.com/components/sast/iac-sast@main
    inputs:
      stage: security
  - component: gitlab.com/components/secret-detection@main
    inputs:
      stage: security

########### test stage: In the test stage, we enforce formatting and validate Terraform configuration before any deployment happens. This acts as a quality gate for merge requests.  ############

terraform:fmt:
  stage: test
  script:
    - terraform fmt -check -recursive

terraform:validate:
  stage: test
  script:
    - terraform init
    - terraform validate


######### deploy stage: Feature branches prefixed with ‘env-’ automatically create ephemeral environments using Terraform. This allows dynamic testing environments per branch. ############

launch_ephemeral_env:
  stage: deploy
  environment:
    name: $CI_COMMIT_REF_NAME
    on_stop: destroy_env
  script:
    - terraform plan
    - terraform apply
  rules:
    - if: $CI_COMMIT_REF_NAME =~ /^env-.*/


##### deploy stage: When the merge request is closed or manually triggered, the environment is destroyed. This prevents unused infrastructure and controls cost.#########

destroy_env:
  stage: deploy
  environment:
    name: $CI_COMMIT_REF_NAME
    action: stop
  script:
    - terraform destroy -auto-approve
  rules:
    - when: manual

##### publish stage:  “On the main branch, Terraform modules are packaged and published to GitLab’s Terraform module registry with versioning.” For handling multiple modules, we use parallel matrix  ####

publish:
  stage: publish
  script:
    - tar -czf module.tgz .
    - curl --upload-file module.tgz <gitlab-terraform-registry>
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

```

## How to structure:

- build
- test
- package
- deploy


```
stages:
  - build
  - test
  - package
  - deploy

variables:
  IMAGE_NAME: registry.gitlab.com/$CI_PROJECT_PATH/my-app
  IMAGE_TAG: $CI_COMMIT_SHA
  DOCKER_TLS_CERTDIR: ""

default:
  image: docker:24.0
  services:
    - docker:24.0-dind

# -----------------------------
# BUILD STAGE: Builds Docker image and stores it as an artifact for reuse in later stages.
# -----------------------------
build:
  stage: build
  script:
    - docker build -t $IMAGE_NAME:$IMAGE_TAG .
    - docker save $IMAGE_NAME:$IMAGE_TAG -o image.tar
  artifacts:
    paths:
      - image.tar
    expire_in: 1 hour

# -----------------------------
# TEST STAGE: Loads built image and runs automated tests inside the container.
# -----------------------------
test:
  stage: test
  needs:
    - build
  script:
    - docker load -i image.tar
    - docker run --rm $IMAGE_NAME:$IMAGE_TAG npm test

# -----------------------------
# PACKAGE STAGE: Logs into GitLab Container Registry and pushes the validated image.
# -----------------------------
package:
  stage: package
  needs:
    - build
  script:
    - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER $CI_REGISTRY --password-stdin
    - docker load -i image.tar
    - docker push $IMAGE_NAME:$IMAGE_TAG
  rules:
    - if: $CI_COMMIT_BRANCH

# -----------------------------
# DEPLOY STAGE (HELM): Uses Helm to upgrade or install the application into Kubernetes, injecting the new image tag dynamically.
# -----------------------------
deploy:
  stage: deploy
  image: alpine/helm:3.13.0
  needs:
    - package
  before_script:
    - mkdir -p ~/.kube
    - echo "$KUBE_CONFIG" > ~/.kube/config
  script:
    - helm upgrade --install my-app ./helm/my-app \
        --namespace production \
        --create-namespace \
        --set image.repository=$IMAGE_NAME \
        --set image.tag=$IMAGE_TAG
  environment:
    name: production
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

```

## Use of: (In 3zgitlabkeywords.md)

- rules
- only/except
- needs
- artifacts
- caching
- environments (dev, qa, prod)

## Deployment/Rollback strategies:

- Manual approvals for Prod
- Rollback strategy

## Manual

### Option A — when: manual in Deploy Job: 

We configure the production deployment job with when: manual, so even after successful build and test stages, deployment requires explicit human approval.

```
deploy_prod:
  stage: deploy
  script:
    - helm upgrade --install my-app ./helm/my-app \
        --set image.tag=$CI_COMMIT_SHA
  environment:
    name: production
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
```


### Option B — Protected Environments (More Secure)

Inside GitLab UI:

- Go to Settings → CI/CD → Protected Environments
- Protect production environment and add approvals requirement
- Go to Settings → Repositories → Protected branches
- Protect specific production branch
- Allow only specific roles (Maintainers) to deploy

Then use:

```
environment:
  name: production
  url: https://example.com/production
```

## Rollback

- Helm maintains release history, so rollback is simply reverting to a previous revision.

```
helm history <RELEASE_NAME>
helm rollback <RELEASE_NAME> <REVISION_NUMBER>
```
- Redeploy Previous Image Tag, Since images are immutable and versioned by commit SHA, rollback is simply redeploying a previous known-good image.

```
deploy_prod:
  stage: deploy
  script:
    - helm upgrade --install my-app ./helm/my-app \
        --set image.tag=$CI_COMMIT_SHA
  environment:
    name: production
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual

rollback_prod:
  stage: deploy
  script:
    - helm rollback my-app 1
  environment:
    name: production
  when: manual
```

- Automatic rollback during upgrade/install (--atomic/--rollback-on-failure)

```
helm upgrade [RELEASE_NAME] [CHART_PATH] --rollback-on-failure --timeout 5m
```

- GitLab Environments: Use the built-in "Rollback" button in GitLab environments, which triggers a previous successful pipeline.



## Expect questions like:

- How would you design CI/CD for two stacks (DB + .NET)?

```
Commit
  ↓
Build & Test .NET
Build & Validate DB (PL/SQL / migrations)
  ↓
Package Artifacts
  ↓
Deploy DB (first)
Deploy Application (second)

```

```
dotnet restore
dotnet build
dotnet test
dotnet publish


Validate SQL syntax
Run migration tool (Flyway / Liquibase / custom script)
Deploy schema changes
Run DB tests

🔹 Deployment Strategy

Production flow:

Apply DB migrations (non-breaking)
Deploy app with new image
Smoke test
Monitor
```
- How do you prevent bad code from reaching Prod?

I use **layered quality gates** including **mandatory merge request approvals**, automated testing, **static security scans**, **container scanning**, and **protected production environments**, **automated staging deployment**. Production deployment requires **manual approval**, and I use safe deployment strategies like **Helm atomic/rollback-on-failure upgrades** to prevent faulty releases.

- How do you secure secrets in GitLab?

Secrets are stored as masked and protected GitLab CI variables. For cloud authentication, I prefer OIDC-based role assumption to eliminate static credentials. For sensitive production secrets, I integrate with cloud secret managers and inject them dynamically during deployment.

- ❓ What about secret rotation?

Secrets are rotated in the cloud secret manager, and pipelines automatically retrieve updated versions. Since we avoid hardcoded credentials, rotation does not require code changes.

- ❓How to secure your CI/CD pipeline

To secure a GitLab CI/CD pipeline, I use **protected and masked CI/CD variables** for secrets, **restrict deployments to protected branches**, **run jobs on isolated Docker or Kubernetes runners**, integrate automated security scans like SAST and secret detection, enforce **manual approvals** for production, and maintain audit logs. I also follow immutable infrastructure practices with Terraform or Helm to prevent manual tampering.

## You must confidently explain:

- GitLab Runners (shell vs docker executor)

- Shell executor runs jobs directly on the host OS without container isolation, which is faster but less secure and harder to scale.

- Docker executor runs each job inside a container, providing isolation, reproducibility, and better scalability. It’s the preferred approach for modern CI/CD.

- Dind: When your job needs to build Docker images, you need Docker inside the container.

```
image: docker:24
services:
  - docker:24-dind
variables:
  DOCKER_TLS_CERTDIR: ""


Job container → connects to → docker:dind service

```

Why it’s needed:

Because:

- Docker executor runs inside container created from the specified Docker image
- You **can’t access host Docker daemon by default**
- So you run a **Docker daemon as a service container**

DinD requires: Privileged mode enabled in runner config, kaniko is a better alternative which runs without a privilege


- Self-hosted runners: A GitLab runner installed and managed by your organization instead of shared GitLab runners.

Instance runners, group runners, project runners (Group runners are visible under the project)

Step 1 — Install Runner in the VM using documentation under Settings -> CI/CD -> Runners

```
sudo apt install gitlab-runner
```

Step 2 — Register Runner

```
gitlab-runner register
```

Step3: You provide:

- GitLab URL
- Registration token
- Executor type (shell, docker)
- Default image (if docker, can be overriden in ci file)

Step 3 — Configuration File (Advanced)

```
/etc/gitlab-runner/config.toml
```

```
[[runners]]
  name = "docker-runner"
  url = "https://gitlab.com"
  token = "TOKEN"
  executor = "docker"

  [runners.docker]
    image = "docker:24"
    privileged = true
    volumes = ["/cache"]
```



- Managing runners at scale

GitLab Runners execute CI/CD jobs using executors like shell or Docker. Docker executor is preferred because it provides isolation and reproducibility. When building Docker images, we use Docker-in-Docker with privileged mode, though tools like Kaniko are safer alternatives. For production workloads, we use self-hosted runners to access private infrastructure. At scale, runners are managed using tags/Gitlab runner on EKS for autoscaling, autoscaling with Kubernetes executor(Enterprise-Level), and environment-based isolation (prod) to ensure security and performance.”