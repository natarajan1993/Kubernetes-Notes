- The version should be between 1 minor and 1 major version
	- If K8 is `1.20.x` then `kubectl` should be between `1.19.x` and `1.21.x`
- Converts CLI commands into REST requests for the API server
- The config file `kubeconfig` lives in `$HOME/.kube/config`
- Contains definitions for
**Clusters
- The list of clusters that `kubectl` knows about
- Each cluster has a name, certificate info and [[API Server]] endpoint
**Users
- Defines different users with different permissions
- Each user has a friendly name, username and credentials 
**Contexts**
- Contexts group (links) clusters and users 
### Example `kubeconfig ` file
```yaml
apiVersion: v1
kind: Config
clusters:
	- cluster:
		certificate-authority: C:\Users\nigel\.minikube\ca.crt
		server: https://192.168.1.77:8443
	name: shield
users:
	- name: coulson
	user:
		client-certificate: C:\Users\nigel\.minikube\client.crt
		client-key: C:\Users\nigel\.minikube\client.key
contexts:
	- context:
		cluster: shield
		user: coulson
	name: director
current-context: director
```
## Common Commands
`kubectl version --output=yaml`
`kubectl get nodes` - This should get the list of clusters on the device
`kubeadm init` - Initialize a K8 [[Cluster]]
- A `kubectl` context is a bunch of settings that tells `kubectl` which cluster to send commands to, and which credentials to authenticate with

`kubectl config view` - View the current config file
`kubectl config current-context` - Shows the current context 
`kubectl config use-context docker-desktop` - Switch the current context

`kubectl explain pods` - List all [[Pod]] attributes
	`--recursive` - Explains every single [[Pod]] attribute in over 1k lines
