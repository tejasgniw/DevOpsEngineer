🎯 What They Will 100% Test You On

# 1️⃣ GitLab CI/CD – Deep Practical Questions

Be ready to explain:

- How you structure .gitlab-ci.yml
- How you handle multi-stage pipelines
- How you manage different environments (dev/qa/prod)

How you use:

- Runners
- Artifacts
- Cache
- Variables
- Protected branches
- Manual approvals

Example structure you should confidently explain:

```
stages:
  - build
  - test
  - scan
  - deploy

build:
  script: dotnet build
  artifacts:
    paths:
      - bin/

test:
  script: dotnet test

deploy_prod:
  script: ./deploy.sh
  when: manual
  only:
    - main
```

## How do you design CI/CD for multi-technology stack?

- Separate jobs per tech (PL/SQL + .NET)
- Common artifact repo
- Environment-specific variables
- Reusable templates

# 2️⃣ Data Warehouse + PL/SQL Automation (Very Important)

This is the key differentiator. They want automation for:

- PL/SQL packages
- Functions
- Procedures
- Database deployments

Be ready to explain:

## How would you automate PL/SQL deployment?

I would store PL/SQL scripts in Git, version them properly, and create a GitLab pipeline that:

- Validates syntax
- Runs unit tests (if available)
- Deploys via SQL*Plus or a migration tool
- Executes in order using versioned scripts
- Uses environment-specific DB credentials stored as GitLab protected variables

Bonus points if you mention:

- Liquibase
- Flyway
- Schema versioning
- Rollback strategy

# 3️⃣ .NET CI/CD

They may ask:

How do you build .NET apps in GitLab?

How do you deploy .NET framework apps?

You should mention:

```
dotnet restore
dotnet build
dotnet test
dotnet publish
```

Deployment options:

- IIS
- Windows server
- Containerized deployment
- Artifact publishing

I separate build and deploy stages and use environment-based configuration transforms


# 4️⃣ GitLab Runners

- Difference between shared vs specific runner?
- Docker executor vs shell executor
- How to scale runners?
- How to secure runners

I prefer Docker executor for isolation and reproducibility. For .NET framework targeting Windows, I may configure a Windows runner with shell executor.

# 5️⃣ Troubleshooting Scenarios (They WILL Ask)

## ❓ Pipeline suddenly failing – what do you do?

- Check job logs
- Verify runner health
- Check variable changes
- Compare with last successful commit
- Reproduce locally if needed

## ❓ Deployment succeeded but app not working?

- Check logs
- Verify config files
- Check DB connectivity
- Validate environment variables
- Check firewall / network

# 6️⃣ DevOps Culture & Mentoring

This is a leadership role (3–5 yrs + modernization).

- How you convince legacy teams to adopt CI/CD?
- How you introduce automation gradually?
- How you document pipelines?
- How you reduce manual deployment

I first analyze current manual processes, identify repetitive steps, automate incrementally, and conduct knowledge-sharing sessions to onboard the team.


# ⚠️ What Makes This Role Unique

This is NOT just cloud DevOps. It is:

- Legacy system modernization
- Database CI/CD
- .NET framework
- Multi-technology orchestration
- Cultural transformation

That’s what they’re evaluating.


# 💥 Likely Client Questions

Predicting these:

- How do you version control database changes?
- How would you handle rollback of a failed DB deployment?
- How do you ensure CI/CD doesn’t break production data?
- How do you manage secrets in GitLab?
- How do you design a pipeline for both .NET and PL/SQL?
- How do you modernize a legacy data warehouse?


# 🧠 What Will Impress Them Most

If you speak confidently about:

- Versioned database migrations
- Environment-based deployment strategy
- Git branching strategy
- Automated validation before production
- Proper approval gates


🚀 What You Should Revise Tonight

- GitLab CI/CD deep understanding
- Basic .NET build lifecycle
- PL/SQL deployment basics
- Artifact handling
- Rollback strategies
- Runner configuration
- Shell scripting basics