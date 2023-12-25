---
aliases:
  - ReplicaSets
---
- Adds the self healing and scaling parts of [[Deployment|Deployments]]
- [[Deployment|Deployments]] manage `ReplicaSets` and adds rollouts and rollbacks
	- `ReplicaSets` manage [[Pod|Pods]]

![[Pasted image 20231224165558.png|550]]

- K8 creates a new `ReplicaSet` for each rollout and slowly scales up the new one and scales down the old one
	- **The old `ReplicaSet` still exists even though there are no more Pods running using that object**
	- This is useful when you want to rollback to the old container image

## Manifest file keys
`replicas` - Number of Pod replicas to deploy & manage
`kind: Deployment`
`strategy:` This block controls how updates happen
	`type: RollingUpdate` - Slowly scale down old `ReplicaSet` and scale up the new one
`template:` - Defines a [[PodTemplate]]
	- Define the `spec: ` block for the [[Pod]] inside this
`apiVersion:` is `apps/v1`
`selector` - the matching labels from the Pods that determines which Pods this [[Deployment]] needs to manage
	- The `selector` key/value will match the `labels` key/value from the Pod
`spec.revisionHistoryLimit` determines how many older ReplicaSets to keep
`spec.progressDeadlineSeconds` how long to wait during a rollout for each new [[Replica]] to come online
