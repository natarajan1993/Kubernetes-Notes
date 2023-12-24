- Provides networking communication between [[Deployment]] objects so Pods can reliably talk to each other

![[Pasted image 20231219211207.png|450]]

- They are also complete Kubernetes objects with a DNS name, IP address and port
- Acts as a load balancer  between the Pods
- It **cannot** provide application layer host and path routing
	- For that you will need [[Ingress]] objects