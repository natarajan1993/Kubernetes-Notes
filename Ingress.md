- Exposes multiple [[ClusterIP]] [[Services]] through a single cloud load balancer

For [[minikube]] you need to run
- `minikube addons enable ingress`
- `minikube addons enable ingress-dns`
- Add `127.0.0.1 mcu.com` to `/etc/hosts` as `sudo`
- Run `minikube tunnel`
- https://stackoverflow.com/questions/58561682/minikube-with-ingress-example-not-working

- Creates a [[LoadBalancer]] Service on port 80 or port 443 and uses host-based or path-based routing to send traffic to the correct backend [[Service]]
- Essentially a load balancer for Services

![[Pasted image 20231226145726.png]]

- Requires a `controller` and a `spec`
- Forwards traffic based on HTTP headers and hostnames or paths
	- If you hit a URL or URI, then the Ingress object can forward that traffic to the correct K8 [[Services|Service]]
	  ![[Pasted image 20231226150350.png]]

## [[IngressClass]]
- Links Ingress objects with Ingress Controllers
## Manifest file
```yaml
apiVersion: networking.k8s.io/v1  
kind: IngressClass  
metadata:  
 name: igc-nginx  
spec:  
 controller: nginx.org/ingress-controller
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mcu-all
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
	  ingressClassName: igc-nginx
  rules:
  - host: shield.mcu.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-shield
            port:
              number: 8080
  - host: hydra.mcu.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-hydra
            port:
              number: 8080
  - host: mcu.com
    http:
      paths:
      - path: /shield
        pathType: Prefix
        backend:
          service:
            name: svc-shield
            port:
              number: 8080
      - path: /hydra
        pathType: Prefix
        backend:
          service:
            name: svc-hydra
            port:
              number: 8080
```

- "If the host name is `mcu.com` and the path is `/shield` then forward that traffic the `svc-shield` [[Services|Service]] on port `8080`"
- - "If the host name is `mcu.com` and the path is `/hydra` then forward that traffic the `svc-hydra` [[Services|Service]] on port `8080`"