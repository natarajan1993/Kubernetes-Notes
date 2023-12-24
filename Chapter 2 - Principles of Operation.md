- Kubernetes is 2 things
	- A cluster to run applications on
	- An orchestrator of cloud-native microservice apps
### Cluster
- A bunch of machines to host applications
- Clusters are comprised of **Nodes** that can be physical servers, VMs, cloud instances, etc
#### [[Control Planes]]
- Exposes the API
- Has a scheduler for assigning work
- Records the state of the cluster and app in a persistent store
#### [[Node]]
- Where user applications run
- Every node runs a service called the `kubelet` that registers it with the cluster and communicates with the API server. 
- Can be physical servers, VMs or cloud instances
#### How it works
- We have our app
- We package it as a container
- We give it to the cluster
- The cluster is made up of one or more [[Control Planes]] and a bunch of [[Worker]] nodes
##### Process
1. Design and write the application as small microservices in any language
2. Package each microservice in it's own container
3. Wrap each container in a [[Pod]]
4. Deploy Pods to the [[Cluster]] via [[Deployments]], [[DaemonSets]], [[StatefulSets]], [[CronJobs]], etc.
5. Once the YAML file is defined, then use the Kubernetes CLI to post to the API server with the desired state of the application
![[Pasted image 20231219204338.png|450]]
- `dply` stands for [[Deployment]]
#### Declarative Model
1. Declare the desired state of the application microservice in a manifest file
2. Post it to API server
3. Kubernetes stores it in the cluster store as the desired state
4. Kubernetes implements the desired state on the cluster
5. A controller makes sure that the observed state matches the desired state
**Manifest files**
- Which images to use
- How many replicas
- Which network ports to open
**Post to API server**
- Use `kubectl` to POST to the API server the manifest file
