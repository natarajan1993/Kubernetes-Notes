---
aliases:
  - StorageClasses
  - SCs
  - SC
---
- Allows you to define different classes/tiers of storage
- Resides in the `storage.k8s.io/v1` API group
- `kind: StorageClass`
- `kubectl` short name - `sc`
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
	name: fast-local
provisioner: ebs.csi.aws.com
parameters:
	type: io1
	iopsPerGB: "10"
	encrypted: true
allowedTopologies:
- matchLabelExpressions:
	- key: topology.ebs.csi.aws.com/zone
	values:
	- eu-west-1a
```

- `provisioner` is the storage plugin that we want to use
- `allowedTopologies` lists where the replicas should go
- `parameters` are specific to the storage provider and you should view documentation

- `StorageClasses` are immutable
## Deployment and usage process
1. Setup storage on your cloud provider on on-prem
2. Create K8 cluster
3. Install and configure the CSI storage plugin
4. Create on or more StorageClass objects
5. Deploy [[Pod|Pods]] and [[PersistentVolumeClaim]] objects that reference that StorageClass
	- The `metadata.name` in the StorageClass is what's referenced in other places

- In the [[PersistentVolumeClaim]] you have a key called `spec.resources.storageClassName` that matches the `metadata.name` in the StorageClass
- In the [[Pod]] you have a key called `spec.volumes.persistentVolumeClaim.claimName` that matches the `metadata.name` in the [[PersistentVolumeClaim]]

External storage <-- StorageClass <-- [[PersistentVolumeClaim]] <-- [[Pod]]
 