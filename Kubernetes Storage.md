- All storage on K8 are called `volumes`
	- K8 storage is called `persistent volume subsystem`
![[Pasted image 20231231132019.png]]

- We only need a plugin to enable cloud storage solutions to work with K8
	- Plugins are based on the **Container Storage Interface (CSI)**
## Persistent Volume Subsystem
- Set of API objects that enable apps to consume storage
1. [[PersistentVolume]] (PV)
2. [[PersistentVolumeClaim]] (PVC)
3. [[StorageClass]] (SC)
- PVs are mapped to external storage classes#
	- Represents external storage devices like AWS EBS in the format that K8 likes
- PVCs authorize [[Pod|Pods]] to use PVs
- SCs make it dynamic

![[Pasted image 20231231132914.png]]

- We can use [[StorageClass|StorageClasses]] to automate this linking of PVs with external storage
- Multiple [[Pod|Pods]] cannot access the same volume
- We **CANNOT** map external volumes to multiple PVs. There is always a 1 to 1 correlation with an external volume and a PV

### Container Storage Interface (CSI)
- Defines standards for what a plugin that enables connectivity with external storage needs to implement
- Plugins run as a set of Pods in the `kube-system` [[Namespaces|Namespace]]

### Access Mode
- `ReadWriteOnce (RWO)` - Defines a [[PersistentVolume|PV]] that can only be bound as R/W by a single [[PersistentVolumeClaim|PVC]]
- `ReadWriteMany (RWM)` - Defines a [[PersistentVolume|PV]] that can only be bound as R/W by a single [[PersistentVolumeClaim|PVCs]]
- `ReadOnlyMany (ROM)` - Defines a [[PersistentVolume|PV]] that can only be bound as Read only by multiple [[PersistentVolumeClaim|PVCs]]

### Reclaim policy
- Tells K8 how to deal with a PV when a PVC is released
- `Delete`
	- Default for [[PersistentVolume|PVs]] created via [[StorageClass|StorageClasses]]
	- Deletes the storage on the external drive as well as the [[PersistentVolume|PV]] on the cluster
- `Retain`
	- Keeps the external drive as well as the [[PersistentVolume|PV]] on the cluster

### VOLUMEBINDINGMODE
- Setting to `Immediate` will create the volume on the external drive as soon as the [[PersistentVolumeClaim|PVC]] is created
	- This can cause the volume to be in a different datacenter from the Pod
- Setting to `WaitForFirstConsumer` will delay creating the drive until a Pod using the PVC is created