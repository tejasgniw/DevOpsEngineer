# 1️⃣ Shared vs Specific Runners

I typically use **Docker executors for isolation and reproducibility**. **For Windows-based .NET Framework builds, I configure dedicated Windows runners with shell executor**. I **scale runners using autoscaling or Kubernetes-based runners** and **secure them through protected branches**, **restricted tags**, and minimal privilege execution.

## Shared Runner (gitlab hosted)

- Available to multiple projects
- Managed centrally
- Good for general workloads

## Specific (Group/Project) Runner (Self hosted)

- Assigned to specific project/group
- More control
- Better for sensitive or custom environments

- I use shared runners for standard builds and dedicated runners for sensitive workloads or special requirements like Windows builds.


# 2️⃣ Docker Executor vs Shell Executor

## 🐳 Docker Executor (Preferred)

- Each job runs in isolated container
- Clean environment every time
- Reproducible builds
- Better security

Best for:

- Linux builds
- Containers
- Modern apps

## 🖥 Shell Executor

- Runs directly on host machine
- No isolation
- Faster but less secure

Best for:

- Windows .NET Framework builds
- Legacy systems
- When Docker isn’t viable

- I prefer Docker executor for isolation and reproducibility. For .NET Framework targeting Windows, I configure a Windows runner using shell executor.


# 3️⃣ How to Scale Runners

- Add more runner instances
- Use auto-scaling runners (Docker Machine / cloud VMs)
- Kubernetes executor for [dynamic scaling](https://oneuptime.com/blog/post/2026-01-27-gitlab-ci-runners-kubernetes/view#:~:text=Running%20GitLab%20CI%20runners%20on,projects%20on%20the%20same%20cluster)
- Configure concurrency, limits (config.toml)

- For high workload environments, I use autoscaling runners or Kubernetes-based runners to dynamically provision build capacity.

# 4️⃣ How to Secure Runners

- Use protected runners for production jobs
- Restrict runner to protected branches
- Avoid privileged mode unless required
- Use Docker executor for isolation
- Store secrets in GitLab variables
- Limit runner access via tags

- I secure runners by restricting them to protected branches, avoiding privileged execution where possible, and using isolated Docker environments to reduce host-level exposure.



# Explanation of protected runners

In GitLab, You can set a runner to protected mode at the instance, group, or project level. 

For a project runner:
- Navigate to your project's Settings > CI/CD.
- Expand the Runners section.
- Find the specific runner you want to restrict and click the Edit (pencil) button.
- Select the Protected checkbox.
- Select Save changes. 

- Once this setting is applied, the runner will only pick up jobs that are triggered on branches or tags that are also defined as "protected" within the repository settings.