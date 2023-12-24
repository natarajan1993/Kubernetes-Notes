## kubectl
#### Inspecting Pods
`kubectl version --output=yaml`
`kubectl get nodes` - This should get the list of clusters on the device
- `-o wide` gives a couple more columns but is still a single line of output
• `-o yaml` takes things to the next level, returning a full copy of the Pod from the cluster store.
`kubectl get pods`
`kubectl describe`
- `kubectl describe pods hello-pod`
`kubectl logs <pod>`
- `--container <container_name>` option will get the logs for a specific container in a [[Pod]]
	- If we don't specify a container name, it will run it on the **first** container in the Pod

`kubeadm init` - Initialize a K8 [[Cluster]]
- A `kubectl` context is a bunch of settings that tells `kubectl` which cluster to send commands to, and which credentials to authenticate with

`kubectl config view` - View the current config file
`kubectl config current-context` - Shows the current context 
`kubectl config use-context docker-desktop` - Switch the current context

`kubectl explain pods` - List all [[Pod]] attributes
	`--recursive` - Explains every single [[Pod]] attribute in over 1k lines

#### `apply`
`kubectl apply -f pod.yml` - Create the [[Pod]] using the [[Pod Manifest]] file

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
## minikube
`minikube start - vm-driver kvm2` - Start the minikube cluster