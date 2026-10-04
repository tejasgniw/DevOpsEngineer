# Questions I expect them to ask

Prepare strong answers for these:

## Architecture

- How would you design this solution?
- How would you create 10–20 environments?
- How would you make environments isolated?
- How would you handle different versions of the application?
- How would you make the environment reproducible?
- How would you implement reset?
- What's the difference between reset, stop/start and destroy?
- How would you prevent users from accidentally destroying environments?

## Terraform

- How do you structure Terraform modules?
- How do you manage Terraform state?
- How do you handle secrets?
- How do you manage multiple environments?
- How do you handle Terraform drift?
- What happens if Terraform apply fails halfway through?

## Packer

- Why Packer?
- What is a golden image?
- How would you version images?
- How would you test an image before promoting it?

## CI/CD
- GitLab CI vs GitHub Actions?
- What is a self-hosted runner?
- How would you secure the runner?
- How would you implement manual approval?
- How would you trigger workflows through the GitHub API?

## Windows
- How comfortable are you with PowerShell?
- How would you automate Windows configuration?
- How would you troubleshoot a Windows VM that isn't responding?

## Database
- How would you automate PostgreSQL/MySQL backup and restore?
- How would you validate that a restore succeeded?
- How would you make database reset repeatable?

## Security
- How would you integrate Polaris into CI/CD?
- How would you handle critical vulnerabilities?
- How would you protect credentials/secrets?
- How would you secure the VMs?