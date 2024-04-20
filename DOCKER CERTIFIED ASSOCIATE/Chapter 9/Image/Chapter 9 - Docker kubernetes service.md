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
[KUBERNETES netowrking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#the-kubernetes-network-model)

Each pod have unique IP on cluster. They can communicate with other on all cluster whether they running on difference nodes. This creates a virtual network that allows pods to easilly communicates with each 
other.

## Chapter 9.5: Service and DNS
[DNS for services and pod](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

[services](https://kubernetes.io/docs/concepts/services-networking/service/)

### Services types
Each services has a type. The type determines the behavior of the serivce and what it can be used for

- ClusterIP(default): Exposes to pod within cluster using unique IP on cluster's virtual network. It's gonna used for communicate within cluster between pods
- NodePort: Expose the service outside cluster on kubernetes nodes. Using a static IP
- Loadbalancer: exposes the service externally using a cloud provider's loadbalancer.
- externalName: Expose an external resource inside cluster network using DNS
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-internal-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### DNS
Kubernetes includes DNS that helps containers locate services using domain names

Pod within same namespace as a service can simply use the service name as domain name.
residen difference namespace must use domain name as `` service_name.namespace.svc.cluster.domain_name``
ex: ``nginx-internal-service.default.svc.cluster.local`` with ``nginx-internal-service`` is service name; ``default`` is default namespace ; ``svc.cluster.local`` is default name of domain name in kubernetes

## Chapter 9.6: Deployment
[deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

Allow you to manage changes to a set of replicas pods.

You can use deployment to do somethings like rolling updates and scalling.
```
apiVersion: v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx.1.19.1
        ports:
        - containerPort: 80
```
> matchLabels parameter used to manged existing pods

## Chapter 9.8: DaemonSets
[daemonsets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

https://learn.acloud.guru/course/6b00566d-6246-4ebe-8257-f98f989321cf/learn/51184904-5d71-4cce-88d1-b12ffee8dd59/1e25b17a-72c8-4b3d-9f34-4d4898ead9ba/watch


