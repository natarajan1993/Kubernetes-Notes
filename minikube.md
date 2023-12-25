- K8 Cluster for running K8 locally
- To access any apps running run `minikube ip` and use `<ip>:port` to access the app
	- The port is defined in the [[Pod Manifest]]
- If the above doesn't work, run `minikube service <service_name> --url`

## [[Common Commands]]
`minikube service <service_name> --url` - Get the url for the service and use this to open the app in the web browser
`minikube service --all` - Get all [[Services]] running in the `minikube` Cluster
