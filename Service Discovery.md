## Registration
- Process of an app posting it's connection details to a [[Service Registry]] so other apps can find it

![[Pasted image 20231230192602.png]]

- K8 uses it's internal DNS as a [[Service Registry]]
	- Internal DNS is called `cluster DNS`
	- Exists in the `kube system` [[Namespaces|Namespace]]
	- The name of the [[Pod|Pods]] is called `coredns`
	- The name of the [[Services|Service]] is called `kube-dns`
	- Runs as a K8 native app
		- Knows it's running on K8 and implements a Controller that watches the [[API Server]] for new [[Services|Service]] objects
		- Any time a new Service object is created, the cluster DNS automatically registers it
		- The registered name is the `metadata.name` property for the Service object
- All K8 [[Services]] automatically register their details with the DNS
- Run `kubectl get pods -n kube-system -l k8s-app=kube-dns` to view the [[Pod|Pods]] that implement the DNS Service
- Run `kubectl get deploy -n kube-system -l k8s-app=kube-dns` to view the [[Deployment]]
- Run `kubectl get svc -n kube-system -l k8s-app=kube-dns` to view the [[Services|Service]]
### Service back-end
- K8 builds an [[Endpoint]] for each Service
- The `kubelet` agent on every Node is watching for the [[API Server]] for new [[Endpoint]] objects
- It creates new networking rules to redirect traffic to new [[Pod|Pods]] once it sees a new [[Endpoint]]

![[Pasted image 20231231071841.png]]

## Discovery
- Apps need these 2 things for service discovery to work
	- Name of the other apps (name of the Service). We set this up using the metadata tags
	- How to convert the name to an IP address

### Cluster DNS
- K8 adds the [[ClusterIP]] address to each container's `/etc/resolv.conf` file
For 2 apps to talk to each other
1. Know the name of each other's Service
2. Name resolution
3. Network routing
- Name resolution is achieved with adding the `ClusterIP` to the container's `/etc/resolv.conf` file
- All objects relating to the cluster DNS are in the `kube-system` Namespace and tagged with the `k8s-app=kube-dns` label
- [[Pod|Pods]] for cluster DNS are managed by the `coredns` [[Deployment]]
- [[Services|Service]] for cluster DNS is a [[ClusterIP]] called `kube-dns`
- [[Endpoint]] object is also called `kube-dns`
#### Network routing
- Container sends all [[ClusterIP]] traffic to their default gateway
	-  Default gateways are where traffic goes to when there is no known route
- Container default gateway sends that traffic to the Node
- Node sends it to it's own default gateway
- Node default gateway processes the traffic using the [[kube-proxy]] app
	- `kube-proxy` implements a controller for watching the API server for new [[Services]] and [[Endpoint]] objects
	- Captures traffic intended for [[ClusterIP]] and forwards it to the [[Pod]] based on the Service's label selector
	- Uses IPVS (IP Virtual Server) to complete this flow

![[Pasted image 20231231082718.png]]

### Namespaces
- Every cluster has an `address space` and Namespaces partition it
- The Fully Qualified Domain Name (FQDN) for a [[Service]] is of the format
	- `<object_name>.<namespace>.svc.cluster.local`
	- The cluster domain is usually `cluster.local`
- Namespaces are useful to partition the Service by environments in the real world
	- One NS can be for `prod` and one can be `dev`
	- Service names **DO NOT** need to be unique between namespaces

![[Pasted image 20231231083412.png|400]]

- Objects can connect to each other in the same Namespace using just their name
- Objects need the FQDN to connect across Namespaces

### Troubleshooting
- Make sure the `coredns` [[Deployment]] and [[Pod|Pods]] are running
`kubectl get deploy -n kube-system -l k8s-app=kube-dns`
`kubectl get pods -n kube-system -l k8s-app=kube-dns`
- Check logs for each of the `coredns` pods
`kubectl get pods <pod_name> -n kube-system`
- Check that the [[Services|Service]] and [[Endpoint]] are running
`kubectl get svc kube-dns -n kube-system`
	- [[ClusterIP]] on the Service should match the IP in the `/etc/resolv.conf` file
`kubectl get endpoints -n kube-system -l k8s-app=kube-dns`
	- The [[Endpoint]] IP should match the [[Pod]] IP
- Start a troubleshooting Pod with networking tools like `ping`, `traceroute`, `nslookup`, `curl`, `dig`, `dnsutils` installed
` kubectl run -it dnsutils --image gcr.io/kubernetes-e2e-test-images/dnsutils:1.3`
`nslookup kubernetes` <- Run inside that Pod
- Output should look like 
```text
Server:         10.96.0.10  
Address:        10.96.0.10#53  
  
Name:   kubernetes.default.svc.cluster.local  
Address: 10.96.0.1
```
`10.96.0.1` is the [[ClusterIP]] of the `kubernetes` Service
- If there is any errors, restart the `coredns` Pods