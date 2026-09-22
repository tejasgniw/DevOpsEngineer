# Gitops

- ArgoCD was integrated as an initiative to have a GitOps approach in place. We had a control plane (EKS cluster via terraform and Helm charts deployed in operations project) with applicationsets configured by matching the flavour of environment & the helm chart. This was the entrypoint. Each time, terraform was applied to deploy our platform, the module workload cluster created an Argocd cluster secret that when synced in the entrypoint of ArgoCD, deployed changes to the applications of the helm charts present in ECR. The cluster secrets was matched according to the cluster name and flavour of the environment. The cluster secrets contained all the sensitive/URL/ARN info needed by the Values.yaml of the helm chart for the platform to run accordingly.

