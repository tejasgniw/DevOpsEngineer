# Key Interview Talking Points

“When a pipeline fails, I start by reviewing **job logs** and **environment variables**, then **reproduce locally** if necessary. I systematically **verify dependencies, configuration, and CI/CD scripts**. For deployment issues, I check container/pod logs, config files, secrets, and network connectivity. I also use **rollback strategies** and **document root causes to prevent recurrence**.”

# Optional Bonus: Proactive Troubleshooting

- Add health checks in Docker/K8s to catch failures early
- Use CI/CD notifications (Slack, email) for failures
- Maintain audit logs of deployments for traceability

# How you debug a failed pipeline (troubleshooting)

## Step 1 — Check the pipeline logs

Go to the failed job in GitLab pipelines

- Look at stdout/stderr to find the exact error
- Example: missing dependency, permission error, Docker build failure

## Step 2 — Reproduce locally if needed

- Ensures you can debug without repeatedly triggering CI

## Step 3 — Inspect environment & variables

- Missing env vars or secret tokens are common causes
- Check $CI_JOB_TOKEN, AWS credentials, etc.

## Step 4 — Check dependencies & versions

- Verify Node/Python/DotNet versions match pipeline image
- Check Docker image layers, Terraform versions, Helm charts

## Step 5 — Add debug info in scripts

## Step 6 — Check previous stages if needs: is used

- Sometimes a failure in build propagates to deploy

## Step 7 — Review GitLab CI configuration

- rules, only/except, parallel, cache misconfigurations can break pipelines



#  How you trace a deployment issue (troubleshooting)


## Step 1 — Identify the failing component

- App not starting?
- Service unreachable?
- Deployment succeeds but pods fail?

## Step 2 — Check logs and status

## Step 3 — Verify config / secrets

- Environment variables
- ConfigMaps / Helm values
- AWS/GCP secrets

## Step 4 — Network checks

- Can the service reach DB or other microservices?
- Use curl, ping, telnet, nslookup

## Step 5 — Rollback if needed