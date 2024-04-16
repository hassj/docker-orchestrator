# Chapter 5: STORAGE AND VOLUMES
## Chapter 5.1: Docker storage in deepth
[Rancher](https://www.suse.com/c/rancher_blog/block-storage-object-storage-and-file-systems-what-they-mean-for-containers/)
[docker STORAGE dirver](https://docs.docker.com/storage/storagedriver/select-storage-driver/)
1. Storage driver
may be called graph drivers such as: overlay2 (ubuntu currently, centos/rhel), aufs (ubuntu 14.04 and older), devicemapper (centos 7 and earlier)

2. Storage model

Persistent data can be managed by storage model: 

2.1. File system storage (overlay2, aufs)

- Data is stored on file system

- Efficient use of memory

- Inefficient with write heavy workloads

2.2. Block storage (devicemapper)

- Store data in Block

- Efficient with write heavy workloads

2.3. Object storage

- Stores data as external objects based store

- Application must be designed to use object model.

- Flexible and scalable easyly.

3. Docker layer file system

both container or image use layer file system consist of image layer and container layer on top of layer.
you can inspect both container and image for deep know location of file system, in generaly overlay2 file system contain data in ``/var/lib/docker/overlay2/<STORAGE_HASH>/``

## Chapter 5.2: Configuring devicemapper
[select-storage-driver](https://docs.docker.com/storage/storagedriver/select-storage-driver/)
[use storage devicemapper storage - deperecated](https://docs.docker.com/storage/storagedriver/device-mapper-driver/)

you can customize devicemapper by configure file daemon.json

devicemapper support two modes:
- lvm-mode by default used in centos 7. It is simulation of external physical disk, so it is inefficient performance than lvm-direct mode

- lvm-direct mode. it is recommended used in production instead of lvm-mode
you can edit directly on daemon.json File
```
sudo systemctl disable docker
sudo systemctl stop docker

then  edit on daemon.json

{
"storage-driver": "devicemapper",
"storage-opts": [
"dm.directlvm_device=/dev/nvme1n1",
"dm.thinp_percent=95",
"dm.thinp_metapercent=1",
"dm.thinp_autoextend_threshold=80",
"dm.thinp_autoextend_percent=20",
"dm.directlvm_device_force=true"
]
}

finaly start docker again, then check create new container and checking file type system on it.
```

## Chapter 5.3: Docker VOLUMES
[volumes](https://docs.docker.com/storage/volumes/)
[bind mount](https://docs.docker.com/storage/bind-mounts/)
[volumes](https://docs.docker.com/storage/)

### Mount types

you can use either bind mount or volume mount to mounting external storage to a container.

1. bind mount 

- Mount a specific path on host machine to container.

- Not portable, dependent on host's file system and directory structure.

2. Volumes mount

- Stores data on host files system, but store path location is managed by docker, so you could write on it.

- More portable

- Can mount the same volume to multiple container.

### Working with VOLUMES

1. Mount syntax

- With bind mount 
`sudo docker run --mount type=bind,source=/home/cloud_user/message,destination=/root,readonly busybox cat /root/message.txt`

- with volume mount
`sudo docker run --mount type=volume,source=share_volume,destination=/root busybox sh -c 'echo vitq >> /root/message.txt && cat /root/message.txt'`

> With volume mount you can write since docker managed the path location on container, so it have write permission.
>
> You can also use --mount with service 

- Use -v flag instead --mount flag for bind mount

`sudo docker run -v /home/cloud_user/message:/root:ro busybox cat /root/message.txt`

- Use -v flag instead --mount flag for volume mount

`sudo docker run -v share_volume:/root busybox sh -c 'echo kaka > /root/message.txt && cat /root/message.txt'`

`sudo docker run -v share_volume:/root busybox cat /root/message.txt`

### docker volume command commonly

- Create volume
` docker volume create volume_name`

- List VOLUMES
`docker volume ls`

- Get information about volumes
`docker volume inspect volume_name`

- docker volume remove
`docker volume rm volume_name`

## Chapter 5.4: Image Cleanup
[docker system df](https://docs.docker.com/reference/cli/docker/system/df/)

- Get information about disk usage on system
`docker system df`  or for more detailed `docker system df -v`

- Remove dangling images (images not reffered by any tag or container)
`docker image prune`

- Remove all unuseed images (not used by any container) 
`docker image prune -a`

## Chapter 5.5: storage in a cluster 
[volumes](https://docs.docker.com/storage/volumes/#share-data-among-machines)

when working with cluster node such as swarm, you may need to share the docker volume between nodes. some options you may reffered to:

- Use application logic to store data in external Object

- Use volume driver to create volume, that is external to 

1. install docker plugin vieux/sshfs on all nodes in the swarm
`docker plugin install --grant-all-permissions vieux/sshfs`

2. On the additional server use for external STORAGE
```
mkdir /home/cloud_user/external
echo External storage! > /home/cloud_user/external/message.txt
```
3. for testing using external share STORAGE, this step maybe optional
```
docker volume create --driver vieux/sshfs \
-o sshcmd=cloud_user@<STORAGE_SERVER_PRIVATE_IP>:/home/cloud_user/external \
-o password=<PASSWORD> \
sshvolume
docker volume ls
```
4. Create service that automatically manages share volume.
```
docker service create --name service_name --replicas number_of_replicas \
--mount volume-driver=vieux/sshfs,source=volume-share,destination=/container/path/mapping,volume-opt=sshcmd=userlogin_external_server@<Private_IP>:/path_sharing_external-server,volume-opt=password=<PASSWORD> image [Args]
```




