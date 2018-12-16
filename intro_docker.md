
# Quick intro to Docker
When exploring new technologies, we often find tutorials made using docker containers as a way to make installations easier and to avoid cluttering our main operating systems. Docker is quite popular and has additional tools to orchestrate a number of containers (Docker Swarm) and manage a multi-container application (Docker Compose). We are focusing this Docker introduction towards Hyperledger fabric tutorials. Corrections and additional requests are welcomed.

## Contents
* [Installation](#installation)
* [docker-compose](#docker-compose)
* [docker](#docker)


# Installation
Installing Docker is quite straightforward, though we need to install `docker` (~150MB) and `docker-compose` (~3MB) in separate steps.
Use your preferred package manager, here we use DNF:
```
sudo dnf install docker
sudo dnf install docker-compose
```
Check that everything is ok:
```
> docker version
Client:
 Version:         1.13.1
...

Server:
 Version:         1.13.1
...
```
&NewLine;
If we receive an error like `Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`, there are probably some permissions issues.
Check that there exists a `docker` group:
```
> cat /etc/group | grep docker
```
If we  don't find any records in it, we need to create it and add the current user to it.
```
sudo groupadd docker
sudo gpasswd -a ${USER} docker
```
Now we would need to logout/login for the changes to take effect. Or we can apply these changes immediately (but temporarily):
```
newgrp docker $USER
exec sudo su -l $USER
```

Finally, start docker and re-check that everything is ok:
```
> sudo service docker start
Redirecting to /bin/systemctl start docker.service

> docker version
Client:
 Version:         1.13.1
 API version:     1.26
 Package version: docker-1.13.1-61.git9cb56fd.fc28.x86_64
 Go version:      go1.10.3
 Git commit:      1556cce-unsupported
 Built:           Wed Aug  1 17:21:17 2018
 OS/Arch:         linux/amd64

Server:
 Version:         1.13.1
 API version:     1.26 (minimum version 1.12)
 Package version: docker-1.13.1-61.git9cb56fd.fc28.x86_64
 Go version:      go1.10.3
 Git commit:      1556cce-unsupported
 Built:           Wed Aug  1 17:21:17 2018
 OS/Arch:         linux/amd64
 Experimental:    false
```

&NewLine;
> Note that some docker images can be very large and we can easily get out of space with all Hyperledger fabric samples (~10GB+). So be sure that our *system storage size* is sufficient, e.g. more than 10GB on our filesystem mounted on `/`.
> An example of an error that we could get in such a situation: `failed to register layer: Error processing tar file(exit status 1) ... no space left on device`


# docker-compose
To run complex, multi-layered applications, we use Docker Compose. We specify the application with a single `.yaml` configuration file, that has all the containers and their relations defined and can thus be used quite easily.

Yaml *(YAML Ain't Markup Language)* has a straightforward structure.
Example:
```
services:
  orderer:
    container_name: orderer
    image: hyperledger/fabric-orderer
    environment:
      - FABRIC_LOGGING_SPEC=debug
...
  peer:
    container_name: peer
    image: hyperledger/fabric-peer
    environment:
      - CORE_PEER_ID=peer
...
```

We start the application with `docker-composer up`. If we use a non-default Yaml file (`docker-compose.yml`), then we must specify the name of our compose file with `-f <file_name>`.
Example:
```
> docker-compose -f docker-compose-simple.yaml up
Creating network "chaincodedockerdevmode_default" with the default driver
Creating orderer ... done
Creating peer    ... done
Creating cli       ... done
Creating chaincode ... done
```


# docker
To view the status of our docker containers, we use `info`  command:
```
> docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
...
```

Once we have docker running, we execute commands inside it with `docker exec <options> <container name> <command>` for existing containers or `docker run <options> <container_name> <command>` for new containers. 
Options could be for example `-i` for interactions through stdin and `-t` to atlocate a terminal (tty). 
Example: lets start `bash` command in an existing container named `chaincode`:
```
> docker exec -it chaincode bash
root@7552b782cd1b:/opt/gopath/src/chaincode#
```

