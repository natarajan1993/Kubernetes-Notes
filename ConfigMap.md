- It is not good practice to store your application and it's configuration in the same container image
- We will have to rebuild multiple images for any code changes if we couple the configuration with the image
- ConfigMap stores the configuration data outside the [[Pod]] and allows you to inject it into the Pod environment dynamically
- Exists in the `v1` API group
- Used to store **non-sensitive** configuration data
	- Environment variable
	- Config files
	- Hostnames
	- Service ports
	- Account names
- To store secret config info, use the [[Secret]] object
- They are basically key/value pairs and can store simple strings or complex files
```json
db-port:13306
hostname:msb-prd-db1
```
```json
key: conf
value:
	directive in;
	main block;
	http {
		server {
		listen        80 default_server;
		server_name   *.nigelpoulton.com;
		root          /var/www/nigelpoulton.com;
		index         index.html
		location / {
			root      /usr/share/nginx/html;
			index     index.html;
		}
	}
}
```

- The app will not know that the data injected into the [[Pod]] came from a ConfigMap
## Command line
`kubectl create configmap <configmap_name> --from-literal key_1=value_1 --from-literal key_1="value_1"`
- `--from-literal` is used to define the key/value pairs in the CLI
- `--from-file` is used to defined the key/value pairs in a file
	- `kubectl create cm <configmap_name> --from-file <filename.ext>`
- `cm` is the shortname

## Using a file
- Use the `|` character for multi-line key-value pairs
	- Treats all the next lines as a single string
```yaml
kind: ConfigMap
apiVersion: v1
metadata:
	name: test-conf
data:
	test.conf: |
		env = plex-test
		endpoint = 0.0.0.0:31001
		char = utf8
		vault = PLEX/test
		log-size = 512M
	key_1: value_1
	key_2: value_2
```

## Injecting into Pods
- As environment variables
- Arguments into container startup commands
- As files into a volume

### Environment variables
- Reference the config variables in the Pod specs
```yaml
apiVersion: v1
kind: Pod
...
spec:
	containers:
		- name: ctr1
		env:
			- name: FIRSTNAME
			valueFrom:
				configMapKeyRef:
					name: <ConfigMap> name
					key: <value from ConfigMap> file
			- name: LASTNAME
			valueFrom:
				configMapKeyRef:
					name: <ConfigMap> name
					key: <value from ConfigMap> file
...
```

`kubectl apply -f <filename>.yml`
- Environment variables are **STATIC**
	- Once the Pod is created, then any changes to the ConfigMap will **NOT** be reflected to the Pod
### Container startup commands
- Use the `$(Config_name)` name in the shell command to reference the ConfigMap shell command
```yaml
spec:
	containers:
	- name: args1
		image: busybox
		command: [ "/bin/sh", "-c", "echo First name $(FIRSTNAME) last name $(LASTNAME)" ]
		env:
		- name: FIRSTNAME
			valueFrom:
				configMapKeyRef:
					name: multimap
					key_1: value_1
		- name: LASTNAME
			valueFrom:
				configMapKeyRef:
					name: multimap
					key_2: value_2
```

- `configMapKeyRef.name` is the `metadata.name` from the ConfigMap manifest file

### Use with volumes
1. Create ConfigMap manifest file
2. Create a ConfigMap volume in the [[Pod Manifest]]
3. Mount the ConfigMap volume into the container
4. ConfigMap entries will appear as individual files

![[Pasted image 20240101071137.png|400]]

- There is a `spec.volumes` key in the [[Pod Manifest]]
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cmvol
spec:
	volumes:
	  - name: volmap
	    configMap:
	        name: <ConfigMap_name>
	containers:
	  - name: ctr
	    image: nginx
	    volumeMounts:
	      - name: volmap
	        mountPath: /etc/name
```

- `spec.containers.volumeMounts` reference the `spec.volumes.name`
- The above creates a file for each of the key and the contents of the file contain the value from the ConfigMap