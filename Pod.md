---
aliases:
  - Pods
---
- Atomic unit of scheduling in Kubernetes
- Pods **contain** Docker containers and containers run inside pods
- Simplest model is to run a single container in a single pod

![[Pasted image 20231219205834.png|400]]

## Pod anatomy
- A Pod is a shared execution environment that can run multiple containers
	- Containers are application processes
- Resources that are shared
	- Net namespace
	- PID namespace
	- `mnt` namespace: Volumes and filesystems
	- UTS namespace: Hostname
	- IPC namespace: Unix domain sockets and shared memory
- Pods **do not** run applications. 
	- Pods only run containers
	- Containers run applications
- Multiple containers in the same Pod share the network stack, volumes, memory, etc.

![[Pasted image 20231219210112.png|400]]
- Multiple containers in the same Pod communicate with each other using `localhost`
- Best practice is to put each container in it's own Pod and use a [[Service Mesh]] to communicate between them
- **Always** scale the app with multiple Pods and not multiple containers in the same Pod
![[Pasted image 20231219210539.png|450]]

- Do not couple your app to the Pod ID and IP address because when the Pod dies, those change
- **Pods are immutable.** You cannot change them once they start running
	- When updates are needed, replace all old Pods with new ones that have the updates
	- When failures occur, replace failed Pods with new identical ones

### Pods augment containers
- Labels and annotations
	- Group [[Pod|Pods]] and associate them with other objects
- Restart policies
- [[Probes]]
- [[Affinity]] and anti-affinity rules
- [[Termination control ]]
- Security policies 
- Resource requests and limits
	- Min and max values for CPU usage, disk IO, RAM, etc.
- Basic K8 Pod manifest file
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: hello-pod
	labels:
		zone: prod
		version: v1
spec:
	containers
	- name: hello-ctr
	  image: nigelpoulton/k8sbook:1.0
	  ports:
	  - containerPort: 8080
```

### Pods assist in scheduling
- Every container in a Pod is guaranteed to be scheduled in the same [[Cluster]]

### Pods enable resource sharing
- Pods give a shared execution environment for one or more containers
	- Shared filesystem
	- Shared network stack
	- Shared memory
	- Shared volumes

- **Pods are mortal. Once they are gone, they are gone forever**
- **ALWAYS** store data and state outside the Pod

### Pod Networking
- A Pod has its own IP address, single range of TCP and UDP ports and a single routing table
- Containers can be accessed externally using the Pod IP + [[Container]] port directly
- Pods can be accessed externally using the Pod IP + the container port
- Containers in the same Pod can communicate using the Pod's `localhost` adapter
##### Example

![[Pasted image 20231224073734.png|400]]

- The main container can reach the supporting container on `localhost:5000`
- You can reach the containers externally using `10.0.10.15:80` and `10.0.10.15:8000`

##### Pod Network

![[Pasted image 20231224074102.png|400]]

- The Pod network needs to be locked down using [[Network Policies]]

### Pod Lifecycle
1. Define the Pod anatomy in a declarative YAML file
2. POST the YAML to the API server
3. Pod enters the `pending` phase
4. Pod is scheduled to a healthy [[Node]] 
5. Local `kubelet` tells container runtime to pull images and start containers
6. Once all containers are pulled and running, the Pod enters the `running` phase

#### Short lived and long lived Pods
- Controllers for long-lived Pods are [[Deployment|Deployments]] , [[StatefulSets]] and [[DaemonSets]]
- Controllers for short-lived Pods are [[Jobs]] and [[CronJobs]]
- Short-lived Pods will terminate after a job runs and completes
	- Usually used for batch jobs
- Long-lived Pods are used for things like web servers and they can be restarted always, on failure or never

#### Pod Scaling
- Always add or remove Pods as the unit of scaling
- DON'T add or remove containers in a Pod

### Multi Container Pods
