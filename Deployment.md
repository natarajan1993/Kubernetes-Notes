---
aliases:
  - Deployments
---
## Theory
- The deployment `spec` describes the desired state of a stateless app
- The deployment `controller` implements the spec and manages it
	- It runs as a continuous loop in the background and compares the current state with the desired state
- `Deployment` objects are part of the workload API
![[Pasted image 20231224164831.png|600]]
- Containers exist in [[Pod|Pods]]
	- Adds labels, annotations and other metadata
- [[Pod|Pods]] are wrapped in `Deployment` objects
	- Adds scaling, self-healing and updates
- There can be only **ONE** `Deployment` object for each `Pod`
	- But a `Deployment` can have replicas of the **SAME** Pod

### Scaling Deployments
- You can manually scale the Pods using `kubectl scale deploy <deploy_name> --replicas <number>` 
	- It is **ALWAYS** better to scale using the manifest file
- Scaling operations are nearly instantaneous
- K8 also has autoscalers we can configure

### Rolling Updates
- Updating Pods are **replacing** them with new Pods
`kubectl apply -f deploy.yml --record`
`kubectl rollout status deployment hello-deploy`
`kubectl rollout pause deployment hello-deploy`
`kubectl rollout resume deployment hello-deploy`

### Rollbacks
`kubectl rollout undo deployment <deploy_name> --to-revision=1`
	- Get the revision number from `kubectl rollout history deployment <deploy_name>`
- Follows the same strategy for the rollout defined in the manifest file
- **A rollback will mismatch any container version changes**
	- If the container in the initial rollout was `1.0` and you rollout `2.0` with a rolling update
		- If you rollback the `2.0`, the current Pods will all be running container `1.0` 
			- If you need them to be running `2.0`, then update the manifest first and do another rollout
## Manifest file keys
- `revisionHistoryLimit` - How many of the previous releases to keep
- `progressDeadlineSeconds` - How long to wait for a Pod to start before considering it not working
- `spec.minReadySeconds` - How long to wait for each Pod to be live before the next Pod is started
- 