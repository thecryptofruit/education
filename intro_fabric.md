# Introduction to Hyperledger fabric
This is a brief tutorial to get started with Hyperledger fabric, one of the most popular private blockchain (sorry, *hashchain*) platforms. Corrections and additional requests are welcomed.
The main and official tutorial is available at https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html and here we provide some additional comments.

## Contents
* [Prepare the environment](#prepare-the-environment)
* [Comments about "Chaincode for Developers" tutorial](#comments-about-chaincode-for-developers-tutorial)
* [](#)


# Prepare the environment
## Prerequisites
1. We need to have curl available, which comes in most all Linux distributions. Check with:
```
> curl --version
curl 7.59.0 ...
```
2. If we don't yet have it, [install Go language](intro_go.md#install-go).

4. If we don't yet have it, [install Docker and Docker Compose](intro_docker.md#installation).


## Hyperledger fabric
Official guidelines at https://hyperledger-fabric.readthedocs.io/en/latest/install.html will ask us to run some script from the Internet, something like `curl -sSL http://bit.ly/2ysbOFE | bash -s 1.4.0-rc1` which points to something like `https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh`. 
> Running scripts from the Internet can be harmful, so we must be careful. As a general rule, we should not mix our development environment with the one we use for personal things, banking etc.

As we will come to see, lots of preparation work is running various scripts as there are currently no prepared packages that would abstract this away. Anyhow, this script will automatically:
1. clone `fabric-samples` repository to your current directory;
2. download several binaries to a `/bin` subdirectory (configtxgen, configtxlator, cryptogen, discover, fabric-ca-client, get-docker-images\.sh, demixgen, orderer, peer), which are used for various things in fabric, for example `cryptogen` is used to create test certificates and signing keys ;
3. download several docker images and yaml files, all in all up to *several GB for fabric v1.4*, so be prepared:
```
REPOSITORY                               SIZE                SHARED SIZE         UNIQUE SIZE
docker.io/hyperledger/fabric-javaenv     1.756 GB            1.389 GB            367.7 MB   
docker.io/hyperledger/fabric-tools       1.56 GB             1.389 GB            171.3 MB   
docker.io/hyperledger/fabric-ca          243.7 MB            123.7 MB            120 MB     
docker.io/hyperledger/fabric-ccenv       1.426 GB            1.389 GB            37.13 MB   
docker.io/hyperledger/fabric-orderer     149.9 MB            123.7 MB            26.22 MB   
docker.io/hyperledger/fabric-peer        156.5 MB            123.7 MB            32.86 MB   
docker.io/hyperledger/fabric-zookeeper   1.432 GB            1.389 GB            43.78 MB   
docker.io/hyperledger/fabric-kafka       1.442 GB            1.389 GB            53.8 MB    
docker.io/hyperledger/fabric-couchdb     1.495 GB            1.389 GB            106.9 MB   
```

In the end, we end up with ~100 files in these directories (~ 40 of them):
```
.
├── balance-transfer
│   ├── app
│   ├── artifacts
│   └── typescript
├── basic-network
│   ├── config
│   └── crypto-config
├── bin
├── chaincode
│   ├── abac
│   ├── chaincode_example02
│   ├── fabcar
│   ├── marbles02
│   ├── marbles02_private
│   └── sacc
├── chaincode-docker-devmode
│   ├── chaincode
│   └── msp
├── commercial-paper
│   └── organization
├── config
├── fabcar
│   ├── javascript
│   ├── javascript-low-level
│   └── typescript
├── fabric-ca
│   └── scripts
├── first-network
│   ├── base
│   ├── channel-artifacts
│   ├── org3-artifacts
│   └── scripts
├── high-throughput
│   ├── chaincode
│   └── scripts
└── scripts
    └── Jenkins_Scripts
```
> As we can see, there's lots of material to get us up to speed, but each example use-case could take quite sime time to study.


# Comments about "Chaincode for Developers" tutorial
Let's continue with the Hyperledger fabric intro tutorial "Chaincode for Developers", officially described at https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html and provide some more context along the way. 

What will we do:
1. Prepare a simple chaincode (a.k.a. a smart contract) written in Go in `sacc.go`, that will store a value on the blockchain and later read that value back.

2. Create the network *chaincodedockerdevmode_default* with 4 services (orderer, peer, cli, chaincode). It will keep our terminal window busy with various logs, so we will only observe this window later.
`docker-compose -f docker-compose-simple.yaml up`

3. Start our chaincode in the second terminal.
`docker exec -it chaincode bash`

4. Interact with our chaincode through the third terminal.
`docker exec -it cli bash`


