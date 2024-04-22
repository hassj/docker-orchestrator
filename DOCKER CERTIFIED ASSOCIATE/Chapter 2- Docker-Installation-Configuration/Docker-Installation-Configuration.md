# CHAPTER 2 DOCKER INSTALLATION AND CONFIGURATION
## Chapter 2.1 Docker community edition

Docker community edition consist of: docker engine, docker swarm, orchestration, network, security

[Docker page](https://docs.docker.com/get-docker/)

## Chapter 2.2: Installing docker on Centos

[Installing docker on centos](https://docs.docker.com/engine/install/centos/)

- Provision a server
- Install requirement package (yum-utils, device-mapper-persistent-data, lvm2)
- Add the Docker repo
- Install Docker and containered package
- Start and enable docker service
- cofigure user belong to docker group 
- Run a container test `docker run hello-world`

## Chapter 2.3 Installing docker on Ubuntu

[Install docker on ubuntu](https://docs.docker.com/engine/install/ubuntu/)

- Provision a server
- Install required package
- Add Docker GPG key and repo
- Install docker and containered package
- Add user to group which able to use DOCKER `usermod -a -G docker user`
- logout and login for take effectively or enter follows command: ` newgrp docker`
- Test 

```
sudo apt-get update \
sudo apt-get -y install apt-transport-https ca-certificates curl gnupg-agensoftware-properties-common \
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
``(optional - verify the integrated of gpg key,... but have not find the number of fingerprint gpg key) sudo apt-key fingerprint 0EBFCD88``
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) 	stable"
sudo apt-get update
sudo apt-get install docker-ce -y

`` or specific version: sudo apt-get install -y docker-ce=5:18.09.5~3-0~ubuntu-bionic docker-ce-cli=5:18.09.5~3-0~ubuntu-bionic containerd.io  ``

sudo usermod -a -G docker cloud_user
```

## Chapter 2.4: Select a storage driver

[docker storage device](https://docs.docker.com/storage/storagedriver/select-storage-driver/)

> Storage driver provide a plugable framework for managing temporary, internal storage of container's writable layer.
> Docker support variety of storage driver.

- overlay2: File-base storage, more efficient for doing a lot of reading. Defautl for Ubuntu and Centos8+
- devicemapper: Block storage, more efficient for doing a lot of writes. Default for Centos 7 and earlier.

You can find out what storage is using on system with `docker info`

You can set the storage driver explicitly by adding flag to docker daemon:
> ` sudo vi /usr/lib/systemd/system/docker.service`
> edit the ExecStart line, adding the --storage-driver devicemapper flag: ``ExecStart=/usr/bin/dockerd --storage-driver devicemapper``

Or set --storage-driver flag to daemon configuration file directory (docker recommended), note that dont do this and set up in daemon service at the same time.

>`sudo vi /etc/docker/daemon.json` 

```
{
  "storage-driver: "devicemapper"
}
```

Notice that, must reload daemon then restart docker service to take effectively.

## Chapter 2.5: Running a container

[Running a container[(https://docs.docker.com/engine/reference/run/)

Run container

`docker run [Options] Image[:Tag] [Command] [Arg]

> Image: The container image to run, by default, these are pulled form the docker hub
>
> Tag: specific version of Image
>
> Command: the command wil be run inside container
>
> Arg: Arguments to pass when running containre command.

For Example running nginx container with other Options

`#docker run -d --name nginx --restart always -p 8080:80 --memory 500M --memory-reservation 256M nginx`
 > -d: detach mode, container run under the background
 > --name: specific name for container, by default without invoke this options, name of container will be called randoom
 > --restart: that is restart policy on container.
 > 1. no (defautl): never restart the container.
 > 2. on-failure: only if the container fails (exits with non-zero exit code)
 > 3. always: always restart container whether it succeed or fails. no matter the case gradfully shutdown the container, an also start container automatically on daemon startup.
 > 4. unless-stopped: similar with always, unless the container was manually stopped.
 > 
 > -p <host_port>:<container_port> : expose container port by mapping container port to host port. this option can be used many times.
 > 
 > -rm: remove container when it exits, cannot be used with option --restart.
 >
 > --memory: hard limit on memory usage
 >
 > --memory-reservation: soft limit memory usage. The container will be restricted to this limit if Docker docker memory contention (not enough memory)
 
 
## Chapter 2.6: Upgrading the docker engine

[Upgrading docker engine](https://docs.docker.com/engine/install/ubuntu/#upgrade-docker-ce)

1. Downgrading docker engine

- stop docker service
- remove docker related package
- install new version 
ex: `sudo apt-get install docker-ce=5:18.09.5.3-ubuntu-bionic docker-cli=5:18.09.5.3`

2. upgrading docker engine

note: dont need to stop docker service

## Chapter 2.7: Configuring logging driver (splunk, journald, etc)

[logging driver](https://docs.docker.com/config/containers/logging/configure/)

1. Logging drivers

It is a plugable framework for accessing log data from services and container in docker.
Docker supports a variety of logging drivers: local, json-file, splunk, gcp, aws, syslog, event...

Among of them, json-file driver maybe best driver with many different options supported.

Checking the current logging driver

`docker info | grep Logging`

2. Configuring loging driver

Creating a json file under ``/etc/docker/daemon.json``
with content: 
```
{
  "log-driver": "json-file",
  "log-opts": {
     "max-size": "15m"
  }
}
```

After that, reload daemon then restart docker service.

3. Specific logging driver for individual container

` docker run --log-driver syslog --log-opt max-size=15m nginx`

## Chapter 2.8: Docker swarm

[docker swarm](https://docs.docker.com/engine/swarm/key-concepts/)

1. Docker swarm

it is a feature called swarm mode, which allows you to build a distributed cluster of container machines. 
It provides many useful features, and can help facilitate orchestration, high availability, and scaling.

2. Build swarm

- provision server
> manager server
> work node server

## Chapter 2.9: Configuring swarm  manager

[Configuring swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/create-swarm/)

- install docker-ce on swarm manager
- initialize swarm
` docker swarm init --advertise-addr <swarm_manager_priate_IP>`
- Checking swarm info and node list
`docker info`
`docker node ls`

## Chapter 2.10: Configuring swarm node

[add node to swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/add-nodes/)

- install docker-ce on worker nodes/
- Get join command from the manager node
`docker swarm join-token worker`

- run join command on all worker nodes
- verify workers successfully joined the swarm
` docker swarm join --token <token> <swarm manager private IP>:23771`

- checking with
` docker node ls`

## Chapter 2.11 Docker swarm backup and restore

[swarm backup and restore](https://docs.docker.com/engine/swarm/admin_guide/#back-up-the-swarm)

1. Backup

- Stop docker service on manager node
- backup all data in directory ``/var/lib/docker/swarm``
`tar -cvzf backup.tar.gz /var/lib/docker/swarm .`
- start docker services again

2. Restoring backup

- stop docker service 
- Delete all data exist on directory ``/var/lib/docker/swarm/*``
- Copy backed-up file to swarm directory.
- start docker service

## Chapter 2.12: Namespace and CGroups

[docker namespace](https://docs.docker.com/engine/security/userns-remap/)

[underlying technology](https://docs.docker.com/get-started/overview/#the-underlying-technology)

1. Namespace

every container created, the docker also create a set of namespace. Each aspect of container run on seperate namespace.
Some commonly namespace use for container

- pid namespace: process isolation
- net namespace: network interface
- ipc Namespace: inter-process communication
- mnt Namespace: filesystem mount
- uts Namespace: kernel and version identifiers
- user Namespaces: requires a special configuration. Allows containers process run as root inside the container, while mapping that user to unprivileged user on the host.

2. Control group (Cgroups)




Reading: 
https://learn.acloud.guru/course/6b00566d-6246-4ebe-8257-f98f989321cf/learn/b483895a-8317-45f0-8cb1-62ff4217484e/adb32178-fafe-4eb2-9d3c-24f156dc921c/watch



Research more deep dive:
 Note: need to deep dive above Options
need to rearch more deep dive about control group and Namespace

