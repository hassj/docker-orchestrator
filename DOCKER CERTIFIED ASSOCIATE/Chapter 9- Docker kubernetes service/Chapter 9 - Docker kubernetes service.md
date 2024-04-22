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

That is tool allows you to run a replica pod dynamically on each node in cluster.

They will run a replica on each existing node and create new replicas on new added node.
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-daemonset
spec:
  selector:
    matchLabels:
      app: my-ds
  template:
    metadata:
      labels:
        app: my-ds
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["sh", "-c", "while true; do sleep 10; done"]
```

## Chapter 9.9: Kubernetes schedule
[kubernetes schedule](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/)

[taints and toleration](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

[Resource management for pod and containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

### Node taints
Node taints control which pod will be run on which nodes. Pods can have toleration with overwrite taints for the specific pod

Each taints has an effect

The NoExecute effect do two things to pods that do not have appropriate toleration:
- Prevents new pod from being scheduled on the node
- evicts existing pods.

### Resources requests
In the container spec, you can specify resources requests for resources such as memory usage  and CPU

The scheduler will avoid scheduling pod on which node do not have enough available resources to satify the requests
```
apiVersion: apps/v1
kind: Pod
metadata:
  name: my-pod-reasonable
  labels:
    app: nginx
spec:
  containers:
  - name: my-nginx
    image: nginx:1.19.1
    command: ["sh", "-c", "while true; do sleep 10; done"]
    resources:
      requests:
        memory: 54Mi
        cpu: 250m
```
>memory: 54Mi (54 Memibytes)
>
> cpu: 250m (250 mili cpu, around 1/4 of cpu)

## Chapter 9.10: Probes
[liveness, readiness, ans startup probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

probes allows you customize how the kubernetes determines the state of containers

- liveness probes: Check whether the container is healthy
- Readiness probes: check whether the container is ready to services user requests
```
apiVersion: v1
kind: Pod
metadata:
  name: my-probe-pod
spec:
  containers:
  - name: nginx
    image: 1.19.1
    ports:
    - containerPorts: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
    
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
```

## Chapter 9.12: Storage with volumes
[volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

some volume types to know:

- nfs: shared network Storage
- hostpath: file are stored in a directory on a worker node. the same file will not appear if the pod run on another node.
- emtydir: Temporary storage that is deleted if the pod is deleted. Useful for testing simple storage between containers in Pod

## Chapter 9.13 Storage with persistent volumes
[Persistent volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

A persistentvolume has a reclaim policy that used to determines what happen with the volume when associated claims are deleted

### Reclaim policies

- Retain - keeps volume and it's data and allows manually reclaiming. Admin is responsible for cleaning up existing data.
- Delete: Deletes both volume and its underlying storage infrastucture
- Recycle: Deletes the data contained in volume automatically and allows it to be used again.

### Expanding a PVC
you can expand a PersistentVolumeClaim by simply edit the size to larger. For this to work, storage class must support volume expansion, ``allowVolumeExpansion`` field will be set to true

- Storage class definition:
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: localdisk
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true

```

- Persistent volume definition:
```
apiVersion: v1
kind: PersistentVolume
metadata: my-pv
spec:
  storageClassName: localdisk
  persistentVolumeReclaimPolicy: Recycle
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
  - path: /tmp/pvoutput
  
```

- Persistent Volume Claim definition:
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: localdisk
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

> pvc will be bound to PV automatically, with satisfy conditions such as: ``storageclassname``, ``accessmodes``, and resources request is not greater than the available resource in PV.

