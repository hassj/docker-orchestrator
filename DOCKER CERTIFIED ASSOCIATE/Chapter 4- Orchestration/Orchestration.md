# Chapter 4: ORCHESTRATION
## Chapter 4.1: Locking and unlocking a swarm cluster
[swarm manger locking](https://docs.docker.com/engine/swarm/swarm_manager_locking/)

docker swarm encrypt sensitive data in the manager nodes with key is stored in uncrypted on managed disks.
``autolock`` is security feature help docker swarm protect the log, or tls communication inforamtion between nodes from the attacker by encrypted. by default it is turned off.


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

Docker use ``raft consensus algorithm`` to manage manger nodes.

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

## Chapter 2.4 Docker inspect

[docker service create](https://docs.docker.com/reference/cli/docker/service/create/)
[how service work](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/)
[deploy service to swarm](https://docs.docker.com/engine/swarm/services/)

use either both ``docker inspect object_id`` or ``docker <type_of_service> inspect ID`` 

> use ``--pretty`` for readable , it also support for some type of docker object such as: service.

1. use format to inspect docker object

``docker inspect --format='{{.ID_field}}' service_name``

## Chapter 4.5 Docker compose

[Migrate to compose v2](https://docs.docker.com/compose/migrate/)
[Docker compose overview](https://docs.docker.com/compose/)
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose version
```
`` docker-compose up -d``; ``docker-compose ps``; ``docker-compose down`` //remove container defined in docker-compose file without remove network and volumes

## Chapter 4.6: Introduction to Docker Stack
[docker stack](https://docs.docker.com/reference/cli/docker/stack/)
[deploy a stack to swarm](https://docs.docker.com/engine/swarm/stack-deploy/)

docker stack are similar to the multi-container apps created using docker-compose. however they can be scaled and executed across swarm just like normal swarm service.

- Deploy stack to cluster using compose file
`docker stack deploy -c compose_file stack_name`

If you change the compose file then use again same command will update with the channges of that file.

- List current Stack
`docker stack ls`

- list the tasks acssociated with a Stack
`docker stack ps stack_name`

- list the services associated with a Stack
` docker stack services stack_name`

- Scaled service in compose file
```
version: '3'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
    deploy:
      replicas: 3 
  busybox:
    image: radial/busyboxplus:curl
    cmd: /bin/sh -c "while true; do echo $$MESSAGE; curl web:80; sleep 10; done"
    environment:
      - MESSAGE=hello!
```

- docker service log
`docker service logs service_name`

- Remove stack and service underly the compose file (network)
`docker stack rm stack_name`

## Chapter 4.7 Node label
[placement constraint](https://docs.docker.com/engine/swarm/services/#placement-constraints) 
[docker node update](https://docs.docker.com/reference/cli/docker/node/update/#add-label-metadata-to-a-node)

- Add node label to certain node
`docker node update --label-add LABEL=VALUE node_name`

- show node label 
`docker node inspect --pretty node_name`

- run service task on only node with "--constraint" flag 
`docker service create --name service_name --constraint label_name==value --replicas 3 image_name`

You can also use negative value as bellows:
`docker service create --name service_name --constraint label_name!=value --replicas 3 image_name`

You can use "--constraint" flag multiple times to list multiple constraint

- checking task associate with that service 
`docker service ps service_name`

- placement-pref with spread strategy to spread tasks evenly across all value of particular label (value null also is a value of label)
`docker service create --name service_name --placement-pref spread=node.label.label_name --replicas replicas_number image_name `

This will spread all tasks to all node having value for balancing load.





# Reading: 
https://learn.acloud.guru/course/6b00566d-6246-4ebe-8257-f98f989321cf/learn/4849ea83-4ecb-459c-bdb0-72487b63082f/023794fa-1f3a-42f6-be3d-550174b09c96/lab/fc2bac2f-e76c-4a58-8117-451019745114



