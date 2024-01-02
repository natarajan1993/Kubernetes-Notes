---
aliases:
  - Secrets
---
- K8 does not encrypt secrets
- Stored as base64 encrypted objects

1. Secret is created and persisted to cluster store as **unencrypted** object
2. Pod that uses it is scheduled to a cluster Node
3. Secret is transferred **unencrypted** to the Pod
4. Pod and containers start
5. Secret is mounted via an in-memory `tmpfs` filesystem and decoded to plain text
6. App uses the secret
7. If the [[Pod]] is deleted, then the Secret is also deleted

-  It’s possible to configure encryption-at-rest with [[EncryptionConfiguration]] objects
- Limited to **1 MB** in side

![[Pasted image 20240101073737.png|400]]

- Every Pod gets a Secret mounted into it as a volume, which it uses to authenticate itself if it talks to the API server
	- If you run `kubectl describe pods <pod_name>` you should see a `/var/run/secrets/kubernetes.io/serviceaccount from <secrets_volume_name>` line in the output
- `kubectl describe secret <secrets_volume_name>` to view the secrets volume 
- `kubectl create secret generic <name> --from-literal user=nigelpoulton --from-literal pwd=Password123`
- `kubectl get secret <name> -o yaml`
#### Sample manifest file
```yaml
apiVersion: v1
kind: Secret
metadata:
	name: tkb-secret
labels:
	chapter: configmaps
type: Opaque
data:
	username: bmlnZWxwb3VsdG9u
	password: UGFzc3dvcmQxMjM=
```

- Change `data` to `stringData` for plain text values
- Extra `type` values [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types)

### Using Secrets in [[Pods]]
- We inject secrets into Pods using a `secrets volume`
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: secret-pod
	labels:
		topic: secrets
spec:
	volumes:
	- name: secret-vol
	secret:
		secretName: tkb-secret
	containers:
	- name: secret-ctr
	image: nginx
	volumeMounts:
	- name: secret-vol
		mountPath: "/etc/tkb"
```

- The `secret.secretName` matches the `metadata.name` for the Secret object
- Secret volumes are mounted automatically as `read only`