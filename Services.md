---
aliases:
  - Service
---
- Provides networking communication between [[Deployment]] objects so Pods can reliably talk to each other

![[Pasted image 20231219211207.png|450]]

Another use case

![[Pasted image 20231225122710.png| 450]]


- They are also complete Kubernetes objects with a DNS name, IP address and port
- Acts as a load balancer  between the Pods
- It **cannot** provide application layer host and path routing
	- For that you will need [[Ingress]] objects
- Every Service object gets it's own stable IP, stable DNS and stable port
- Services use labels and selectors to dynamically send traffic to the correct [[Pod|Pods]]
	- If there are multiple labels, the Service uses an `AND` operation to select them
	- It ignores any extra labels as soon as the Pod is found
### [[Endpoint]] Objects
- The Service dynamically updates the list of healthy Pods using the labels and [[Endpoint]] Objects
- Endpoint Objects are used to store the list of healthy Pods that matches the selectors in the Service Object
- There will be an [[Endpoint]] created every time a Service is created
- When traffic is sent to a Service
	- The cluster's internal DNS resolves the Service name to an IP address
	- Traffic is routed to one of the Pods in the Endpoint list
	- Any app that can query the K8 API can also query the Endpoint API directly without resolving through the Service DNS
#### Commands
- `kubectl get endpointslices`
- `kubectl describe endpointslice <name>`
	- Gives the internal IP for each Pod that is using the Service

### [[LoadBalancer]] Services
- Useful when your cluster is running on a cloud platform
- `spec.type: LoadBalancer` in the manifest file
- `ports.port` is the ingress traffic port for the Service
- `ports.targetPort` is the port that the Pods have open and all the ingress traffic from the port above is forwarded to Pods on this port
- Run `kubectl get svc --watch` and use the `EXTERNAL-IP` value to access the Pods
- 
### Accessing Services from inside the cluster ([[ClusterIP]])
- [[ClusterIP]] is a type of Service object that has an IP accessible only from **inside** the Cluster
- Guaranteed to be a stable IP for the duration of the Service
- All ClusterIP objects are automatically registered with the Cluster's DNS
	- The IP is from the ClusterIP object
	- The name is the name of the Service object
	- Therefore, all Pods in the cluster can automatically access each other via the Service<->ClusterIP combo
- Only works for Pods within the Cluster

### Accessing Services from outside the cluster
#### NodePort
- Gives a dedicated port or every cluster node
```yaml
apiVersion: v1
kind: Service
metadata:
	name: magic-sandbox
spec:
	type: NodePort
	ports:
		- port: 8080 # Port for internal objects to access the Service on
			nodePort: 30050 # Port for internal services to access the Service on
selector:
	app: hello-world
```

#### LoadBalancer
- Integrates with an internet-facing cloud based load balancer service

### Service Discovery
- All K8 clusters run an internal DNS service
- Service names are automatically registered with this DNS Service
	- Every Pod is automatically configured to use the cluster DNS
- Can also use environment variables 
	- Cannot learn about new Services after the Pod that the env variables are in is created
	- Prefer DNS over env variables

### Manifest file keys
```yaml
apiVersion: v1
kind: Service
metadata:
	name: svc-test
spec:
	type: NodePort
	ports:
	- port: 8080
		nodePort: 30001
		targetPort: 8080
		protocol: TCP
selector:
	chapter: services
```
- `metadata` - Name for the Service as well as any labels and annotations
- `spec.port.port:` - Port number for the internal port of the [[Pod]]
- `spec.port.nodePort:` - Port number for the external port that you put in the browser
- `spec.port.targetPort:` - Port number for the what internal port to forward any ingress traffic to
- 