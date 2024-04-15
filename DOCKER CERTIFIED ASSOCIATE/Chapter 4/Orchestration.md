# Chapter 4: ORCHESTRATION
## Chapter 4.1: Locking and unlocking a swarm cluster
[swarm manger locking](https://docs.docker.com/engine/swarm/swarm_manager_locking/)

docker swarm encrypt sensitive data in the manager nodes with key is stored in uncrypted on managed disks.
autolock is security feature help docker swarm protect the log, or tls communication inforamtion between nodes from the attacker by encrypted. by default it is turned off.


- Turning autolock on existing swarm on with belows command:
`docker swarm update --autolock=true`

note: you need to unlock autolock everytime you want to restart docker on manager nodes

- Turning off autolock on manager swarm:

`docker swarm unlock` 

- Disable autolock off existin swarm off:
`docker swarm udpate --autolock=false`

- Enable autolock from the begining on swarm cluster

`docker swarm init --autolock`

- Rotate autolock key on swarm manger 

`docker swarm unlock-key --rotate`

## Chapter 4.2: High availability in swarm cluster

[raft consensus in swarm mode](https://docs.docker.com/engine/swarm/raft/)

[Admin swarm](https://docs.docker.com/engine/swarm/admin_guide/)

1. Multiple manger nodes
More manger nodes the more fault tolerance of high availability for swarm. but many manager nodes will affect to the performance of swarm cluster when maintain consistence state of swarm cluster.

Docker use raft consensus algorithm to manage manger nodes.

2. Quorum
It is a feature of multiple manager nodes. more than one half of number manager nodes create an quorum. support for maintain tasks, nodes...
Generally, Quorum maybe odd number instead of even number.

3. High availability zone
docker recommend at least 3 AZ for manager nodes.

## Chapter 4.3: Introduction docker services

[docker service create](https://docs.docker.com/reference/cli/docker/service/create/)

[how service work](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/)

[Deploy services to swarm](https://docs.docker.com/engine/swarm/services/)

- Service is used to run applications on docker swarm. A services specifies a set of one or more replicas tasks. these tasks will be distributed automatically across docker nodes.

- Syntax
`docker service create [OPTION] image ARGS`

Example:
`docker service create --name nginx --replicas 3 -p 8008:80 nginx `

> -p: public_port:service_port

- Template can be used to give somewhat dynamic values to some flags 

`docker service create --name node-hostname --env NODE_HOSTNAME="{{.Node.Hostname}}" nginx `

- Manage service with commonly commands

`docker service rm service_name`	// remove service will automatically remove container and something related to that service

`docker service ls `				// list current services.

`docker service ps service_name `	// list service's tasks

`docker service inspect service_name`	// get more infor about services 

`docker service rm service_name`		//delete existing service

`docker service update [OPTIONS] service_name` `docker service update --replicas=3 nginx`	//making change to service

`docker service create --mode global service_name`	//run one task on each of nodes in swarm cluster 

`docker service scale service_name=replicas_number` `docker service scale nginx=4`	// scale service is change the number of replicas


# Reading: 
https://learn.acloud.guru/course/6b00566d-6246-4ebe-8257-f98f989321cf/learn/4849ea83-4ecb-459c-bdb0-72487b63082f/ca4c399d-1c92-448a-8251-90c0f5f45aee/watch

