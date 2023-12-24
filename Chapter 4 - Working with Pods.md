## Process for running an app on K8
1. Write your app in any language
2. Package it as a container
3. Wrap the container image in a [[Pod]]
4. Run it on Kubernetes

**Containers cannot run directly in Kubernetes**
**Containers will ALWAYS have to be run in a [[Pod]]

## Static [[Pod|Pods]] vs [[Controllers|Controller]]
- Pods can be deployed by
	- Directly using a Pod manifest
	- Indirectly via a Controller 
- Pods deployed via a manifest file are **static** and cannot scale, self-heal or perform updates
	- There is no control plane and it only runs on the local `kubelet` Node
- Pods deployed via [[Controllers]] can scale, restart and perform updates
- 