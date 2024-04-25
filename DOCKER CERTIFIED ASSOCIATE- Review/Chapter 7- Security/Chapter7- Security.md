# Chapter 7 Security
## Chapter 7.1: Signing image and enable docker content trust
[content trust in docker](https://docs.docker.com/engine/security/trust/#push-trusted-content)

DCT- Docker content trust provides a secure way to verify the integrity of images before you pull or run them

- Creating sign each images

`docker trust key generate <signer name> `

- Adding signer to trust key

`docker trust signer add --key signer_name.pub signer_username name_of_repository`

- Sign an image and push it to repository

`docker trust sign <repo>:<tag>`

- Enable DOCKER_CONTENT_TRUST

` export DOCKER_CONTENT_TRUST=1`

> in Docker enterprise edition, you can also enable it in daemon.json
>
> when DCT is enabled, DOCKER will pull and run the signed image. Attemping to run or pull unsigned image will occure error 
>
> when DCT is enabled, ``docker push`` will signe image automatically before pushing it.

## Chapter 7.2: Default docker engine Security
[docker security](https://docs.docker.com/engine/security/)

## Chapter 7.3: Docker MTLS
[Manag swarm security with public key infrastructure - PKI](https://docs.docker.com/engine/security/)

[overlay network driver](https://docs.docker.com/network/drivers/overlay/)

### Overlay newtork encrypted
Overlay network ecnrypted with `` -opt encrypted`` flag when create network on docker swarm.

### MTLS
Mutually authenticated transport layer security - is enabled by default, you won't need to do anything for set it up.

When a swarm is initialized, a root certificate authority (CA) is created, it is used to generate certs for all nodes as they join in cluster.

Note: Needed review

## Chapter 7.4: Securing the docker daemon http socket
[Protect the docker daemon socket](https://docs.docker.com/engine/security/protect-access/)

Docker by default listen on socket, so not expose to the network.

Note: Needed review



