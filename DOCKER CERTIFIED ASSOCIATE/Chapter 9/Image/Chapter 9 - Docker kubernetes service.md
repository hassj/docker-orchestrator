# CHAPTER 9: DOCKER KUBERNETES SERVICE
## Chapter 9.1 Overview
## Chapter 9.2: KUBERNETES orchestration in DOCKER
## Chapter 9.3: App configuration in KUBERNETES
[config maps](https://kubernetes.io/docs/concepts/configuration/configmap/)

[secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

1. Application configuration is all abou the managing configuration data and passing that data to Applications.

2. Configmaps Allows you to store the configuration data in key/value pairs format. This data can include simple value or large block of data such as multiple line of configuration files.
```
apiVersion: v1
kind: ConfiMap
metadata:
  name: my-configmap
data:
  key1: import data
  key2: |
    another data
	multiple lines
	more lines
```

3. secret
It is similar with ConfigMap, but it store the data at encrypted. Use secrets for stroring sensitive data such as: username, passwrod, or api tokens.
```
apiVersion: v2
kind: Secret
metadata:
  name: my-secret
type: opaque
data:
  username: <base64 encode pass>
  pass: <base64 encode value>
```

3. Environment variables

configuration values can be passed to the Environment variables that will be visible to the container process at runtime.

volumemount configuration data can ben mounted as the fomr of docker file system

```
apiVersion: v1
kind: Pod
metadata:
  name: my-configuration-Pod
spec:
  restartPolicy: Never
  containers:
  - name: busybox
    image: busybox
	command: ["sh", "-c", "echo \"ConfigMap environment variable: $CONFIGMAPVAR\" && echo \"Secret Environment variable: $SECRETVAR\" && echo   "]
	env: 
	-  name: CONFIGMAPVAR
	   valueFrom:
	     configMapKeyRef: 
		   name: my-configmap
		   key: key1
    -  name: SECRETVAR
	   valueFrom:
	     secretKeyRef:
		   name: my-secret
		   key: username
	
	volumeMounts:
	- name: configmap-vol
	  mountPath: /etc/configmap
	- name: secret-vol
	  mountPath: /etc/secret
  
  volumes:
  - name: configmap-vol
    configMap:
	  name: my-configmap
  - name: secret-vol
    secret:
	  name: my-secret

```

## Chapter 9.4: Kubernetes network model

https://learn.acloud.guru/course/6b00566d-6246-4ebe-8257-f98f989321cf/learn/51184904-5d71-4cce-88d1-b12ffee8dd59/e7fa45c4-8057-4112-8ca6-6e32e0975e56/watch
