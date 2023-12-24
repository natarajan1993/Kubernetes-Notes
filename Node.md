- Servers that are workers of the K8 cluster
1. Watch the API server for new work assignments
2. Execute work assignments
3. Report back to [[Control Planes]] using the [[API Server]]

![[Pasted image 20231217065051.png|450]]

## Components
### [[Kubelet]]
- Main K8 agent that runs on every cluster [[Node]]
- Kubelet is the same as a [[Node]]
- Watches the [[API Server]] for new work tasks
	- Reports back after executing that task
- Kubelet is **not** responsible for executing failed tasks
	- Reports back to the [[Control Planes]] and lets that decide what to do
### [[Container Runtime]]
- A [[Kubelet]] uses a container runtime to perform container related tasks like pulling images, starting and stopping containers
- Uses an abstract interface called a [[Container Runtime Interface]]
	- Container services like [[Docker]] can plug into these
- `containerd` is the most common container runtime 
	- Stripped down [[Docker]] basicially

### [[Kube Proxy]]
- Runs on every [[Node]] and is responsible for local cluster networking
- Ensures each [[Node]] get it's own IP address
- Implements local IP tables for load balancing and traffic routing
