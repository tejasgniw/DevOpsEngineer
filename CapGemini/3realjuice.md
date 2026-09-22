# 1. GitLab CI/CD — This Is the Core

They don’t want someone who knows CI/CD. They want someone who can design it from scratch.

**Be ready to explain:**

## How you design a multi-stage GitLab pipeline


## How to structure:

- build
- test
- package
- deploy

## Use of:

- rules
- only/except
- needs
- artifacts
- caching
- environments (dev, qa, prod)

## Deployment/Rollback strategies:

- Manual approvals for Prod
- Rollback strategy


## Expect questions like:

- How would you design CI/CD for two stacks (DB + .NET)?
- How do you prevent bad code from reaching Prod?
- How do you secure secrets in GitLab?


## You must confidently explain:

- GitLab Runners (shell vs docker executor)
- Self-hosted runners
- Managing runners at scale



# 2. Data Warehouse + PL/SQL Automation (Critical)

**This is NOT just DevOps — this is database CI/CD.**

You must understand:

## How to automate:

- Compilation of PL/SQL packages
- Running DB unit tests
- Version controlling SQL scripts
- Database migrations

## They may ask:

- How do you deploy DB changes safely?
- How do you manage schema versioning?
- How do you roll back a failed DB deployment?

## Know concepts like:

- Idempotent scripts
- Migration tools (Liquibase / Flyway – even conceptually)
- Dependency handling in PL/SQL
- Running SQL via pipeline (sqlplus or similar)



# 3. .NET CI/CD

## You must understand:

- Pipeline for .NET:
- dotnet restore
- dotnet build
- dotnet test
- dotnet publish

And:

- Artifact generation
- NuGet packaging (if applicable)
- Environment-based config transforms
- Deploying to IIS / App Servers / Containers

## They may ask:

- How do you handle environment-specific configs?

- How do you ensure repeatable builds?

- How do you implement blue-green or rolling deployment?



# 4. Infrastructure + DevOps Engineering Depth

This is where your DevOps background will help.

## Be strong in:

- Containers
- Dockerfile best practices
- Multi-stage builds
- Image tagging strategy
- Image scanning

## Terraform basics:

- Modules
- State management
- Remote backends
- Ansible basics for config management

## Linux / Shell
- Bash scripting
- Automating deployment scripts
- Debugging pipeline failures


# 5. Troubleshooting Skills (They WILL Test This)

Be ready to explain:

- How you debug a failed pipeline
- How you trace a deployment issue

## How you isolate:

- App issue vs infra issue vs DB issue

## Real engineers think in:

- Logs
- Exit codes
- Environment parity
- Observability



# 6. DevOps Culture & Collaboration

They will evaluate soft skills.

## Prepare examples where you:

- Introduced CI/CD in a legacy team
- Dealt with resistance
- Standardized deployment practices
- Reduced manual work
- Improved release frequency



# 🎯 What You Should Practically Practice Before Interview

1️⃣ Build a Sample GitLab Pipeline:

- One for a .NET app
- One for SQL deployment
- With Dev → QA → Prod flow

2️⃣ Write:

- A simple Bash automation script
- A Python automation script

3️⃣ Be able to whiteboard:

- End-to-end architecture
- Flow from commit → pipeline → artifact → deployment → monitoring