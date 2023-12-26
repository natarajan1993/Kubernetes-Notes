## kubectl
#### Inspecting Pods
`kubectl version --output=yaml`
`kubectl get nodes` - This should get the list of clusters on the device
- `-o wide` gives a couple more columns but is still a single line of output
• `-o yaml` takes things to the next level, returning a full copy of the Pod from the cluster store.
`kubectl get pods`
- `namespaces` - Used to describe [[Namespaces]]
	- Aliased as `ns`
	- `kubectl describe ns default`
- `replicasets` or `rs` - Alias for [[ReplicaSet|ReplicaSets]]
`kubectl describe`
- `kubectl describe pods hello-pod`
- `kubectl describe deploy <deployment_name>` 
`kubectl logs <pod>`
- `--container <container_name>` option will get the logs for a specific container in a [[Pod]]
	- If we don't specify a container name, it will run it on the **first** container in the Pod

`kubeadm init` - Initialize a K8 [[Cluster]]
- A `kubectl` context is a bunch of settings that tells `kubectl` which cluster to send commands to, and which credentials to authenticate with

#### `config`
`kubectl config set-context --current --namespace <ns_name>` - Set the default [[Namespaces|Namespace]] 
`kubectl config view` - View the current config file
`kubectl config current-context` - Shows the current context 
`kubectl config use-context docker-desktop` - Switch the current context

`kubectl explain pods` - List all [[Pod]] attributes
	`--recursive` - Explains every single [[Pod]] attribute in over 1k lines

#### `apply`
`kubectl apply -f pod.yml` - Create the [[Pod]] using the [[Pod Manifest]] file
	- Use the `--record` flag to save the imperative command (manually run command vs the manifest file) so other devs can view it later
	- View the history using the `history` command
#### `exec` - Runs commands inside a Pod
`kubectl exec` 
- `kubectl exec hello-pod -- ps aux`
- `kubectl exec -it <pod_name> -- /bin/bash` - Runs an interactive terminal inside a pod
- K8 uses the `apk` package manager - Alpine Package Keeper
	- User `apk add <package_name>` to install
	- https://docs.alpinelinux.org/user-handbook/0.1a/Working/apk.html

#### `edit` - Edits some attributes
- `kubectl edit pods <pod_name>`
	- Opens a file and edit with Vim
	- Cannot change things like name, container port, container name, etc.

#### `delete`
- `kubectl delete pod <pod_name>`
- `kubectl delete svc <service_name>`
- Can also use the `-f` flag to delete everything in a [[Pod Manifest]]
	- `kubectl delete -f <mainfest_name>.yml`
	- You can also chain multiple files in the same command like `kubectl delete -f deploy.yml -f svc.yml -f other.yml`

#### `scale`
- Use to scale the number of `Replicas` in the [[Deployment]]
- `kubectl scale deploy <deploy_name> --replicas <number>`

#### `rollout`
- This command is used to check the status or pause/resume a new [[Deployment]] after you run `apply` on the manifest file
- `kubectl rollout status deployment <deploy_name>` - Gets the status of the rollout
![[Pasted image 20231225091720.png|450]]

- `kubectl rollout pause deployment <deploy_name>` - pauses the current deploy
- `kubectl rollout resume deployment <deploy_name>` - resumes the current deploy
- `kubectl rollout history deployment <deploy_name>`
#### `expose`
- Takes an object like a [[Deployment]] and exposes it as a [[Services|Service]]
`kubectl expose deployment <deployment_name> --type=NodePort`

## minikube
`minikube start - vm-driver kvm2` - Start the `minikube` cluster