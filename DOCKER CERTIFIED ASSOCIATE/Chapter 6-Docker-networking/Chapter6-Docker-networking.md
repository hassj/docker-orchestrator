# CHAPTER 6: NETWORKING
## Chatper 6.1: Docker networking
[mirantis document](https://docs.mirantis.com/welcome/)

Container netowrking model (CNM) mange networking for containers

- Sandbox: the isolated networking unit contain all the network components associated with a Container, usually like a Linux namespace

- Network: Collections of endpoints connected to one another

- Endpoint: Connects a Sandbox/container to network. Each sandbox/container can have any number of endpoint, but has exactly one endpoint for each network it is connected to.

- network driver: Handles the actual implementations of CNM concepts

- IPAM: ip address manager manage allocating the ip and subnet for endpoint and network

## Chapter 6.2: Built-in network drivers
[network driver](https://www.docker.com/blog/)
[networking](https://docs.docker.com/network/)

Docker includes in some of Built-in network drivers, knowns as Native network driver

with docker run, you can chose network driver by using --net=driver option.

1. Host driver

- Container on host use the same network resource of host network namespace components.

- No both containers use same port on host

`docker run -d --net host --name busy-box radial/busyboxplus:curl sleep 3600`

2. Bridge driver

- The bridge netowrk driver use linux bridge network to provide connectivity between containers on the same host. this is default network driver for containers running on the same host (not support in swarm cluster)

- Create a Linux bridge for each docker network

- Use cases: isolated networking among the containers on a single host.

3. Overlay driver

- Providing connectivity between containers across docker hosts such as docker swarm

- Uses VXLAN data plane, allows the underlying infrastructure network to route the data traffic between the hosts as a way that is transparent to the containers themselves.

- Automatically configure network interface, bridge, etc... on each hosts as needed

- Use cases: networking between container across hosts

```
docker netowrk create -d Overlay my-overlay-net
docker service create --name overlay-nginx --network my-overlay-net nginx
docker service create --name overlay_busybox --network my-overlay-net radial/busyboxplus:curl sh -c 'curl localhost:80 && sleep 3600'
```

4. MacVlan driver

- Offering the lightweight connecting for containers by connecting directly to the host network interface.

- Uses directly Linux interface instead of Linux bridge interface

- Harder to configure and greater dependency between MacVlan and external network

- more lightweight and less latency 

- Use cases: in case need for extremely low latency, need for the case containers's ip address have same subnet with external subnet

`docker netowrk create -d macvlan --subnet x.x.x.x/subfix --gateway y.y.y.y -o parent=eth0 network_name`

5. None

- Does not provide any networking implementation

- container is completely isolated from other containers and host 

- None driver will create seperated namespace for each container without network and endpoint components

- Use cases: when you want to set up manually the networking or there is no need for container networking
```
docker run --net none -d --name none_nginx nginx
docker run --rm radial/busyboxplus:curl curl non_nginx:80

``the result in is: "Curl (6) Couldn't resolve host 'none_ngin' ``
```

## chapter 6.3: Docker bridge network
[Bridge network driver](https://docs.docker.com/network/drivers/bridge/)

### Common commands

- Creating a network, by default docker use Bridge driver
`docker network create --driver driver_name network_name`

- Run a contain and connecting it to specific network
`docker run -d --name container_name --network network_name image`

- Connecting a running container to a specific network

`docker network connect network_name container_name`

- List networks
`docker network ls`

- disconnect a running container from a network
`docker network disconnect network_name container_name`

- inspect network
`docker network inspect network_name`

- remove a network
`docker network rm network_name`

Note: to remove network, it must either disconnect all containers running on or remove them.

### Embeded DNS

Docker networks implement an embeded DNS server, allowing containers and services to locate and communicate with one another.

Container can communicate with serivcies and other containers using service or container name, or network alias

`docker run -d --name test_container --network example_net --network-alias my_example_net image`

`docker network connect --alias alias_network_name example_net container_name`

## Chapter 6.4: Deploying service on docker overlay network
[overlay network driver](https://docs.docker.com/network/drivers/overlay/)

- Docker swarm use overlay network named ingress by default

`docker network create --driver overlay network_name`

`docker network create --driver overlay --attachable my_network_name`

> --attachable enable manual container attachment

- You can create your own networks to provide isolated communication between services and containers.

- Example communicate between container and service on swarm.
```
docker network create --driver overlay --attachable my-overlay-net
docker service create --name overlay-service --replicas 3 --network my-overlay-net nginx
docker run --rm --network my-overlay-net radial/busyboxplus:curl curl overlay-service:80
```

## Chapter 6.5: Exposing container externally
[docker container port](https://docs.docker.com/reference/cli/docker/container/port/)
[docker service create](https://docs.docker.com/reference/cli/docker/service/create/)
[docker container run](https://docs.docker.com/reference/cli/docker/container/run/)

- Publish port on host, mapping it to container port
`docker run -p HOST_PORT:CONTAINER_PORT image`

- publish a port on swarm hosts, mapping it to a port on the service's Container
`docker service create -p HOST_PORT:CONTAINRE_PORT service_name`

### Hosts and ingress mode on docker services
docker swarm supports two modes for publishing ports on services.

1. ingress

- the default mode on docker

- Use a routing mesh 

2. Host

- Publish port running on host where the task are running

- can not have multiple replicas task on same service on same host.

`docker service create --name service_name -p mode=host,published=<host_port>,target=<container_port> service_name`

- Use cases: go fit with global option on docker service, and dealling with low latency traffic.

## Chapter 6.6: network troubleshooting
[troubleshooting container networking](https://success.mirantis.com/article/troubleshooting-container-networking)

1. Network troubleshooting

- Get container logs
`docker logs container_name`

- Get logs from a task of a service 
`docker service logs service_name`

- Get docker daemon logs
`journalctl -u docker`

2. Netshoot image

A greate way for troubleshooting is run a container inside context of network space using ``nicolaka/netshoot``

```
docker run -d --name none-net-nginx --network none nginx
docker run --rm --network container:none-net-nginx nicolaka/netshoot curl localhost:80
```

## Chapter 6.7: Configuring docker to use external dns
[daemon dns options](https://docs.docker.com/reference/cli/dockerd/#daemon-dns-options)
[Networking overview](https://docs.docker.com/network/)

- Change default configuration dns for docker via ``daemon.json`` file

```
sudo vi /etct/docker/daemon.json``

{
   "dns": ["8.8.8.8"]
}

then restart daemon
sudo systemctl restart Docker

Test again

docker run --rm nicolaka/netshoot nslookup google.com
```

- another way to overwrite above configuration
` docker run --dns 8.8.4.4 nicolaka/netshoot nsookup google.com`
