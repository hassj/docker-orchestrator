# CHAPTER 3 Image Creation, Management, and Registry
## Chapter 3.1 Introduction

Container and images use a layered file system. Each layer contain only the differences from the previous layer.

The image consists of one or more read only layers, while container add one addition writable layer.

[storage driver](https://docs.docker.com/storage/storagedriver/)

check history of image by bellows command:

``docker image history image_name`` this will show all commands used to build this image, some layer is 0B it means that that layers just have file system, don't download anything.

## Chaper 3.2 The components of dockerfile

[docker file](https://docs.docker.com/reference/dockerfile/)

some of directive used in docker file

``FROM``: usually use at the first of dockerfile

``ENV``: set environments inside container, it will be referenced within the dockerfile and are visible to the container at runtime.

``RUN``: Creates new layer on top of previous layer by running a command inside the layer and committed that changes.

``CMD``: specify default command used to run a container at execution time.

``EXPOSE``: specify which port are being published when running a Container

``WORKDIR``: sets the current working directory of subsequent directives: CMD, COPY, ADD, ENTRYPOINT.
you can use multiple WORKDIR either absolute directory or relative. the second relative WORKDIR will base on previous WORKDIR url.

``COPY``: copy file from the local machine to images

``ADD``: similar to COPY, but can also pull files using url 

``STOPSIGNAL``: speecify the type of signal that will be used to stop the container

``HEALTHCHECK``: specify the command in order to run certain task to verify that container is running properly.

## Chapter 3.4 Building efficient images

[multi stage builds](https://docs.docker.com/build/building/multi-stage/)

[best practices for writing dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

Keep in mind some general tips:

>Image are as small as possible
>
>Result in ephemeral containers, that means can be stoped, started and destroyed easily
>
>put things that are less likely to change on lower level layers
>
>don't create any unneccessary layers
>
>avoid including any unneccessary files, packages, etc in the images
>
>using multi stage to decrease size smaller

### Multi stage builds

docker support more than one FROM directive on the same dockerfile, one FROM directive starting one stage.

```
FROM ubuntu:bionic AS stage1
ENV NGINX_VERSION 1.14.0-0ubuntu1.11
RUN apt-get update && apt-get install -y curl
RUN apt-get update && apt-get install -y nginx=$NGINX_VERSION
WORKDIR /var/www/html

FROM alpine:3.9.3
WORKDIR /root
COPY --from=stage1 /var/www/html/index.html ./

CMD ["nginx", "-g", "daemon off;"]

STOPSIGNAL signal SIGTERM
HEALTHCHECK CMD curl localhost:80
```

## Chapter 3.5 Managing images

[docker images](https://docs.docker.com/reference/cli/docker/image/)

list all immediate images as follows:
`docker image ls -a`

inspect image for detail, for example as bellows:

`docker image inspect nginx_image -f "{{.Architecture}} {{.Os}}"`

Notice that with flag "-f" it will remove all tag then remove the image:

`docker rmi -f image_id`

## Chapter 3.6 Flattening a docker image to a single layer

- run a container from an Image
- export that container to archive file
`docker export container_name > tarfile.tar`

- import that archive file to new image (it is flattened image)
`cat tarfile.tar | docker import - new_image:latest`

note: check layer history of Image
`docker image history image_name`

## Chapter 3.7 Docker registries

[docker registry](https://docs.docker.com/registry/)
just run the container by using registry image, port 5000 either set the configurations with -e (ex: REGISTRY_ followed by each configuration key)
For example set log level configuration for registry container.
`docker run -d --name registry_ex -p 5000:5000 --restart=always -e REGISTRY_LOG_LEVEL=debug regsitry:2`

note:

show detail the output of ``docker image history imaage_id`` 
`docker image history image_name --no-trunc`

- Securing a registry server by some steps: use TLS with a certificate and requeire user authentication

```
docker container stop registry && docker container rm -v registry  //  stop then remove registry container existing

mkdir -p ~/registry/auth											// create folder contain the cert file

docker run --entrypoint htpasswd registry:2 -Bbn user_test Passsowrd_which_u_want > ~/registry/auth/htpasswd	// overwrite entrypoint for registry container then create user pass word for registry container then output to auth/htpasswd on host server

mkdir -p certs

openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt	//generate key and cert CA for regisitry server

docker run -d -p 443:443 --restart=always --name registry \
-v /home/cloud_user/registry/certs:/certs\
-v /home/cloud_user/registry/auth:/auth\
-e REGISTRY_HTTP_ADDR=0.0.0.0:443\
-e REGISTRY_HTTP_TLS_CERTIFICATE=/certst/domain.crt\
-e REGISTRY_HTTP_KEY=/certs/domain.key\
-e REGISTRY_AUTH=htpasswd\
-e "REGISTRY_AUTH_HTPASSWD_REALM=Registry realm"\
-e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd\
registry:2

```

## Chapter 3.8 Using docker registries

``docker search ubuntu_image`` allow us to search docker image on docker hub, only work with docker hub, not work with private Registry
[docker search](https://docs.docker.com/reference/cli/docker/search/)

[registry](https://docs.docker.com/registry/)

to work with doker private registry, you must login:
[docker login](https://docs.docker.com/reference/cli/docker/login/)

[docker pull](https://docs.docker.com/reference/cli/docker/image/pull/)

[docker push](https://docs.docker.com/reference/cli/docker/image/push/)

In case you login fail with "unknow authority" because docker does not trust the certificae that our registry is using. Just Add/trust CA into docker host or turn off verify certificate.

- turn off verify authentication on docker registry by edit on daemon.json file ``/etc/docker/daemon.json``

```
{
	"insecure-registries": ["hostname.com"]
}
```

any time when you make a change on that file must restart docker service 

- Adding the CA cert of registry private server to docker host

```
mkdir -p /etc/docker/cert.d/domain_name_folder_on_docker_host
cp user_on_registry_host@registry_host_or_Ip:/location/of/CAcert location_on_docker_host
```
that's it for secure way then login to registry without error message occur




# Reading:

review opessl to crate certs

