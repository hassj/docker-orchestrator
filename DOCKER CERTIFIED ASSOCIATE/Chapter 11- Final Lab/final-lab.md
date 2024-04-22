# CHAPTER 11 FINAL LAB
## Installation and Configuration

1. What procedure should we follow to upgrade the Docker engine on an Ubuntu server?

- Good answer
Install newer versions of the docker-ce and docker-ce-cli packages.

We must install newer versions of the packages in order to upgrade Docker.

- wrong Answer
Stop Docker, remove the packages, and then reinstall the packages with a newer version.

We don't need to stop Docker or remove the packages before upgrading.

2. Where can we find a repository URL for installing the Docker Enterprise Edition (EE)?

- Good answer
Docker Hub

If we have a Docker EE license, then we can pinpoint and retrieve a repository URL on Docker Hub.

- wrong Answer
Docker Enterprise Hub

A separate hub for Docker EE does not exist.

3. Which of the following is not backed up when performing a Docker Trusted Registry (DTR) metadata backup?
- Good answer
Docker images.

A DTR metadata backup does not include the images themselves.

- wrong Answer
Role-based access control (RBAC) settings.

A DTR metadata backup includes a DTR's RBAC settings.

## Image Creation, Management, and Registry
1. Which of the following is true about the creation of private Docker registries?

- Good answer
We can create our own registry by running a container with the registry image.

Running this image will create a private Docker registry.

- wrong Answer
We need Docker Trusted Registry (DTR) present if we want to generate a private registry.

Docker Trusted Registry is a Docker Enterprise Edition (EE) registry with additional features, but we can run a registry without it.

2. What does the HEALTHCHECK directive do?

- Good answer

It sets a command that will be used by the Docker daemon to determine whether the container is healthy.

The HEALTHCHECK directive sets a command that is used to determine container health.

- wrong Answer

It sets a command that will be used to inform the container of the health status of the docker daemon.

The HEALTHCHECK directive is used by the Docker daemon to determine the health status of the container, not the other way around.

3. How would we go about keeping track of changes made to an image in source control (i.e., git)?

- Good answer
We would store the Dockerfile in source control.

We can keep the Dockerfile in source control to track any changes made to the Dockerfile.

- wrong Answer

Maintain tags for each new version within the Docker registry.

Docker registries are not in source control.

### Orchestration
1. Which flag allows us to return specific fields with docker inspect?

- Good answer

--format

The --format flag allows us to supply a Go template so that we can return specific data fields that are in a particular format.

- wrong Answer

--filter

This flag will not return specific fields with docker inspect.

### Storage and Volumes
1. Which of the following commands can we use to locate the actual files that store a container's internal data?

- Good answer

docker container inspect <container>

This command will return the container metadata, including the location of its data on the host.

- wrong Answer

docker image inspect <image>

This command will retrieve the information about an image, not a container's internal data

### Security
1. How can we provide custom certificates to the Universal Control Plane (UCP) and Docker Trusted Registry (DTR)?

- Good answer
We can upload certificates via the UCP and DTR web UIs.

We can upload certificates in the administrative settings section for both UCP and DTR.

- wrong Answer
We must supply the certificates during the UCP and DTR installation process.

We can add our custom certificates after UCP and DTR install.
