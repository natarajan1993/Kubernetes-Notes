---
aliases:
  - Namespace
---
- Divides a single K8 [[Cluster]] into multiple virtual clusters
## Use Cases
- Useful for sharing a single cluster across multiple departments
- Each namespace can have it's own users, permissions and resource quotas
- DO NOT use for isolating different environments

- New objects go into the `default` Namespace unless specified otherwise
- `Kube-system` is where the DNS, metrics server and other [[Control Planes]] run
- `Kube-public` is for objects that need to be readable by anyone

## [[Common Commands]]
- `kubectl get namespaces`
- `kubectl describe ns default
- `kubectl get svc --namespace kube-system`
- You can also use the `--all-namespaces` flag to return objects from all Namespaces
- `kubectl create ns <ns_name>`
- `kubectl config set-context --current --namespace <ns_name>` - Set the default [[Namespaces|Namespace]] 

## Deploying to Namespaces
- Imperatively - using the `-n` or `--namespace` flag
- Declaratively - Using the YAML file
- 
