
# Quick intro to Docker
When exploring new technologies, we often find tutorials made using docker containers as a way to make installations easier and to avoid cluttering our main operating systems. Docker is quite popular and has additional tools to orchestrate a number of containers (Docker Swarm) and manage a multi-container application (Docker Compose). We are focusing this Docker introduction towards Hyperledger fabric tutorials. Corrections and additional requests are welcomed.

## Contents
* [Installation](#installation)
* [Basic commands about the environment](#basic-commands-about-the-environment)
* [docker](#docker)
* [docker-compose](#docker-compose)


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


# Basic commands about the environment
Terminology ... An image gets build via a Dockerfile, where each line adds to the layered stack that represents our image. A container then is a running instance of that image.

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

To list all available images, use `docker image ls` or `docker images`:
```
> docker images
REPOSITORY                               TAG                 IMAGE ID            CREATED             SIZE
docker.io/hyperledger/fabric-javaenv     1.4.0-rc1           e1759d6bb479        4 days ago          1.76 GB
hyperledger/fabric-javaenv               latest              e1759d6bb479        4 days ago          1.76 GB
hyperledger/fabric-tools                 latest              b18da9e44a53        5 days ago          1.56 GB
...
```

To check the size of images, we can use `docker system df`. Note thar *RECLAIMABLE* size represents the size of inactive images without any containers, that can be reclaimed by pruning.
```
> docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              9                   4                   2.348 GB            2.081 GB (88%)
Containers          4                   0                   66.8 kB             66.8 kB (100%)
Local Volumes       1                   1                   69.04 kB            0 B (0%)
```

We can get even more details using the `-v` verbose switch. Here we have the size specified in more detail. *SIZE* is the total virtual size of the image, *SHARED SIZE* represents how much an image shares with another one, *UNIQUE SIZE* is the size that is unique for that particular image, which is thus very most important when we try to locate our "large images".
```
> docker system df -v
Images space usage:

REPOSITORY                               TAG                 IMAGE ID            CREATED             SIZE                SHARED SIZE         UNIQUE SIZE         CONTAINERS
docker.io/hyperledger/fabric-javaenv     1.4.0-rc1           e1759d6bb479        4 days ago          1.756 GB            1.389 GB            367.7 MB            0
docker.io/hyperledger/fabric-tools       1.4.0-rc1           b18da9e44a53        5 days ago          1.56 GB             1.389 GB            171.3 MB            1
docker.io/hyperledger/fabric-ca          1.4.0-rc1           ebef9b071211        5 days ago          243.7 MB            123.7 MB            120 MB              0
docker.io/hyperledger/fabric-ccenv       1.4.0-rc1           a6bcefd5b845        5 days ago          1.426 GB            1.389 GB            37.13 MB            1
docker.io/hyperledger/fabric-orderer     1.4.0-rc1           18ebff4f7365        5 days ago          149.9 MB            123.7 MB            26.22 MB            1
docker.io/hyperledger/fabric-peer        1.4.0-rc1           f612bfabedee        5 days ago          156.5 MB            123.7 MB            32.86 MB            1
docker.io/hyperledger/fabric-zookeeper   0.4.14              d36da0db87a4        2 months ago        1.432 GB            1.389 GB            43.78 MB            0
docker.io/hyperledger/fabric-kafka       0.4.14              a3b095201c66        2 months ago        1.442 GB            1.389 GB            53.8 MB             0
docker.io/hyperledger/fabric-couchdb     0.4.14              f14f97292b4c        2 months ago        1.495 GB            1.389 GB            106.9 MB            0

Containers space usage:

CONTAINER ID        IMAGE                        COMMAND                  LOCAL VOLUMES       SIZE                CREATED             STATUS                        NAMES
984abcea4469        hyperledger/fabric-tools     "/bin/bash -c ./sc..."   1                   0 B                 23 minutes ago      Exited (137) 21 minutes ago   cli
48990a940343        hyperledger/fabric-ccenv     "/bin/bash -c 'sle..."   0                   0 B                 23 minutes ago      Exited (137) 21 minutes ago   chaincode
417fe4719672        hyperledger/fabric-peer      "peer node start -..."   0                   33.4 kB             23 minutes ago      Exited (0) 21 minutes ago     peer
93a0bad9cc2e        hyperledger/fabric-orderer   "orderer"                0                   33.4 kB             23 minutes ago      Exited (0) 21 minutes ago     orderer

Local Volumes space usage:

VOLUME NAME                                                        LINKS               SIZE
53b398350fec184b935830311a3a162bc344324b0683ad0813ba9f8cf2946323   1                   69.04 kB
```

The data is located at `/var/lib/docker/<something>`, mine is in the subdirectory `overlay2` (Linux Fedora). It can be changed with `dockerd -s`, see https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-storage-driver .


# docker
Once we have docker running, we execute commands inside it with `docker exec <options> <container name> <command>` for existing containers or `docker run <options> <container_name> <command>` for new containers. 
Options could be for example `-i` for interactions through stdin and `-t` to atlocate a terminal (tty). 
Example: lets start `bash` command in an existing container named `chaincode`:
```
> docker exec -it chaincode bash
root@7552b782cd1b:/opt/gopath/src/chaincode#
```


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

