- A server running a collection of system services that make up the control plane of the cluster
- Can also be called
	- Masters
	- Heads
	- Header Nodes
- 3 to 5 Control Planes are usually run in Production and 1 is usually run for test or dev environments
## [[API Server]]
- **All** communication between **All** components **MUST** go through the API server
- It exposes a RESTful API that you `POST` YAML configs via HTTPS
	- YAML files are also called **Manifests**
- All requests to the API server are subject to authentication and authorization checks
- After authentication, the YAML file is stored in the persistent store and work is scheduled
## [[Cluster Store]]
- Only stateful part of the control plane and persistently stores the entire YAML config and state of the cluster
- Based on `etcd`
	- Run 3 to 5 replicas of `etcd` for high availability
- This is the **single source of truth** for a cluster
- A default K8 install will install a replica of the cluster store on every control plane node and automatically configures high availability
- `etcd` will halt updates to user configurations if there is no consensus between them
- `etcd` uses the `RAFT` algorithm to resolve data races for database writes
## [[Controller Manager]] and [[Controllers]]
- Implements all the background controllers that monitor cluster components and respond to events
- It is a **controller of controllers**
- Some of the controllers include the [[Deployment]] controller, the [[StatefulSet]] controller, and the [[ReplicaSet]] controller.
#### High level flow of [[Controllers]]
1. Obtain desired state
2. Observe current state
3. Determine differences
4. Reconcile differences
## [[Scheduler]]
- Scheduler watches the API server for new work tasks and assigns it to healthy worker [[Node]]
- If the scheduler cannot find any Nodes, then the task gets marked as pending
- If the cluster is running in the cloud, then the control plane will be running a cloud controller manager

![[Pasted image 20231217064453.png|450]]

