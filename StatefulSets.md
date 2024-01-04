---
aliases:
  - StatefulSet
---
## Theory
- `kubectl` alias: `sts` 
**Similarities to [[Deployment|Deployments]]**
- Very similar to Deployments but allows preservation of state
- Keep watch over desired and observed states and keep them reconciled
- Supports self-healing, scaling, updates, etc.
**Differences from [[Deployment|Deployments]]**
- Predictable and persistent [[Pod]] names
- Predictable and persistent DNS hostnames
- Predictable and persistent volume bindings
- Starts Pods one at a time and if and only if the previous Pod is up and running
### Naming conventions
- The format of StatefulSet Pod names is `<StatefulSetName>-<Integer>`
	- Indexing starts at 0
- Have to be valid DNS names
### Pod creation
- Creates one [[Pod]] at a time
- Always waits for the previous Pods to be running and ready
- Same rules apply for scaling too
	- The controller terminates the Pod with the highest index first
	- Only terminates the next highest index Pod if this one has successfully terminated
- This is really useful to know which Pods will be scaled down so important data is not lost
### Deleting the StatefulSet object
- Does not terminate Pods in order
	- Scale down the StatefulSet to 0 before deleting the object
- Set `terminationGracePeriodSeconds` to at least 10 seconds to allow for Pods to gracefully shut down
### Volumes
- Volumes are matched 1-1 with the Pods using the name
	- `<vol-<pod_name>>` <- For the volume name
- Volumes are decoupled from [[Pod|Pods]] using the [[PersistentVolumeClaim]] system
	- They can survive independently from the Pod lifecycle
- After scale up and scale down, the new Pods are attached to the appropriate volumes by their name
	- This is possible because the Pods and volumes exist independent of each other
	- This also works even if the last Pod in a StatefulSet is deleted
### Handling failures
-  If you have a StatefulSet called `tkb-sts` with 5 replicas, and `tkb-sts-3` fails, the controller will start a replacement Pod with the same name  `tkb-sts-3` and attach it to the same volumes
- If the original `tkb-sts-3` recovers, then there will be 2 Pods with the same name attempting to write to the `vol-tkb-sts-3` volume
	- This will lead to data corruption
	- Manual intervention is needed
### Headless [[Services|Service]]
- StatefulSets use a headless [[Services|Service]] to create predictable DNS hostnames for every Pod replica
	- The **Head** of a Service is it's IP address and its **Tail** is the list of Pods it sends traffic to
	- Headless Service is a K8 [[Services|Service]] object without an IP address
	- It becomes a StatefulSet's **governing Service** when you set that Service's name under `spec.serviceName`
- The Service creates DNS `SRV` records for each Pod replica that matches the label selector of the headless Service
	- This is the only purpose of a Headless Service
	- Other Pods and apps can find members of the StatefulSet via DNS lookups of the headless Service name
- We can set the `spec.clusterIP: None` in the manifest file for the Service and that should define the Service as headless

## Examples
### [[StorageClass]]
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
	name: flash
provisioner: pd.csi.storage.gke.io
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
parameters:
	type: pd-ssd
```

### Headless Service
```yaml
apiVersion: v1
kind: Service
metadata:
	name: dullahan
	labels:
		app: web
spec:
	ports:
	- port: 80
		name: web
	clusterIP: None # Indicates that it's a headless Service
	selector:
		app: web
```

### StatefulSet manifest
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
	name: tkb-sts
spec:
	replicas: 3
	selector:
		matchLabels:
			app: web
	serviceName: "dullahan" # References the Service we created above. This is the "governing Service"
	template:
		metadata:
			labels:
				app: web
		spec:
			terminationGracePeriodSeconds: 10
			containers:
			- name: ctr-web
				image: nginx:latest
				ports:
				- containerPort: 80
					name: web
				volumeMounts:
				- name: webroot
					mountPath: /usr/share/nginx/html
	volumeClaimTemplates:
	- metadata:
		name: webroot
		spec:
			accessModes: [ "ReadWriteOnce" ]
			storageClassName: "flash"
			resources:
				requests:
					storage: 1Gi
```

#### `volumeClaimTemplates`
- Dynamically creates a [[PersistentVolumeClaim|PVC]] each time a StatefulSet controller spawns a new Pod replica
- Names the created [[PersistentVolumeClaim|PVC]] so it connects to the right [[Pod]]
- `spec.template` defines how to create new Pods
	- `volumeClaimTemplates` defines how to create new [[PersistentVolumeClaim|PersistentVolumeClaims]]
#### Peer discovery
- By default, K8 places all objects in the `cluster.local` DNS subdomain
`<object-name>.<service-name>.<namespace>.svc.cluster.local` - Fully Qualified Domain Name for StatefulSet objects

- You can run `dig SRV <object-name>.<service-name>.<namespace>.svc.cluster.local` from a jump pod

#### Scaling
- When a StatefulSet is scaled up, a [[Pod]] and a [[PersistentVolumeClaim]] are created
- When a StatefulSet is scaled down, the [[Pod]] is deleted but the [[PersistentVolumeClaim]] remains
- When a new Pod is created, it's automatically attached to the correct PVC
- This is possible because StatefulSet keeps track of the Pods and PVCs using ordinals
`kubectl scale sts <StatefulSet_name> --replicas=<number>`
