# Welcome to IBM Blockchain Roadshow! 

## This tutorial will help you to create a local development environment which will be used during the workshop.

**1. Prerequisites**

- PC with a 64-bit processor and virtualization bit enabled (vt-x or similar), 8Gb of RAM and 30GB of free space available
- Oracle VirtualBox 5.x installed: https://www.virtualbox.org/wiki/Downloads
- Vagrant installed: https://www.vagrantup.com/downloads.html

**2. Now let's create a Vagrant box (a virtual machine) which will be used as a sandbox nested on our PC for all Hyperledger Fabric tests**

- Clone the current Hyperledger Fabric git repository: 

> git clone https://github.com/hyperledger/fabric

- Go to the folder created by the repo: 

> cd fabric/devenv

- run Vagrant box: 

> vagrant up

- Go get coffee... this will take a few minutes. 
- Once complete, you should be able to ssh into the Vagrant VM just created: 

> vagrant ssh

- This should log you into the box to the following folder: **/opt/gopath/src/github.com/hyperledger/fabric**
- We will be using remaining commands inside the box.

**3. Let's now clone some useful scripts which will automate the process of downloading necessary binaries and Docker images which will be used during the workshop**

- Inside Vagrant box, run the following script: 

> curl -sSL https://goo.gl/Q3YRTi | bash

- This should create a new folder called /bin with necessary binaries and download Docker images for Hyperledger Fabric.
- Now, let's create an environment variable so that binaries in /bin folder can be picked up without fully qualifying the path to each binary: 

> export PATH=/opt/gopath/src/github.com/hyperledger/fabric/bin:$PATH

- Check if you can invoke these binaries from the folder : **/opt/gopath/src/github.com/hyperledger/fabric**, e.g. by running all of these:

> cryptogen
> configtxgen
> configtxlator
> peer

- Also check if Docker images are ready:

> docker images

_REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
hyperledger/fabric-ca          latest              17f38f1c8e80        13 days ago         238MB
hyperledger/fabric-ca          x86_64-1.0.3        17f38f1c8e80        13 days ago         238MB
hyperledger/fabric-tools       latest              ac1f4a1e58a6        2 weeks ago         1.33GB
hyperledger/fabric-tools       x86_64-1.0.3        ac1f4a1e58a6        2 weeks ago         1.33GB
hyperledger/fabric-couchdb     latest              b2188fa55138        2 weeks ago         1.47GB
hyperledger/fabric-couchdb     x86_64-1.0.3        b2188fa55138        2 weeks ago         1.47GB
hyperledger/fabric-kafka       latest              9e2a425c9dd6        2 weeks ago         1.29GB
hyperledger/fabric-kafka       x86_64-1.0.3        9e2a425c9dd6        2 weeks ago         1.29GB
hyperledger/fabric-zookeeper   latest              3b50cfad9af3        2 weeks ago         1.3GB
hyperledger/fabric-zookeeper   x86_64-1.0.3        3b50cfad9af3        2 weeks ago         1.3GB
hyperledger/fabric-orderer     latest              fd1055ee597a        2 weeks ago         151MB
hyperledger/fabric-orderer     x86_64-1.0.3        fd1055ee597a        2 weeks ago         151MB
hyperledger/fabric-peer        latest              b7f253e87c0c        2 weeks ago         154MB
hyperledger/fabric-peer        x86_64-1.0.3        b7f253e87c0c        2 weeks ago         154MB
hyperledger/fabric-javaenv     latest              1d778fcc14c0        2 weeks ago         1.41GB
hyperledger/fabric-javaenv     x86_64-1.0.3        1d778fcc14c0        2 weeks ago         1.41GB
hyperledger/fabric-ccenv       latest              2e5898d8b21b        2 weeks ago         1.28GB
hyperledger/fabric-ccenv       x86_64-1.0.3        2e5898d8b21b        2 weeks ago         1.28GB
busybox                        latest              54511612f1c4        4 weeks ago         1.13MB_

**4. If everything went smoothly, we should now be able to run a test Hyperledger Fabric network on our Vagrant box. It is not necessary that you understand what is happening behind the scenes as everything will be explained again during the workshop.**

- Clone Hyperledger Fabric Samples to your Vagrant box: 

> git clone https://github.com/hyperledger/fabric-samples

- Go to **fabric-samples/first-network/**
- Now, let's run an automated script that will firstly generate some necessary crypto for our network: 

> ./byfn.sh -m generate

- Confirm default settings by typing "y" and pushing enter to continue:

_Generating certs and genesis block for with channel 'mychannel' and CLI timeout of '10000'
Continue (y/n)? y
proceeding ...
/opt/gopath/src/github.com/hyperledger/fabric/bin/cryptogen

##########################################################
##### Generate certificates using cryptogen tool #########
##########################################################
org1.example.com
org2.example.com

/opt/gopath/src/github.com/hyperledger/fabric/bin/configtxgen
##########################################################
#########  Generating Orderer Genesis block ##############
##########################################################
2017-10-18 05:12:44.190 UTC [common/configtx/tool] main -> INFO 001 Loading configuration
2017-10-18 05:12:44.613 UTC [common/configtx/tool] doOutputBlock -> INFO 002 Generating genesis block
2017-10-18 05:12:44.628 UTC [common/configtx/tool] doOutputBlock -> INFO 003 Writing genesis block

#################################################################
### Generating channel configuration transaction 'channel.tx' ###
#################################################################
2017-10-18 05:12:44.737 UTC [common/configtx/tool] main -> INFO 001 Loading configuration
2017-10-18 05:12:44.749 UTC [common/configtx/tool] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2017-10-18 05:12:44.751 UTC [common/configtx/tool] doOutputChannelCreateTx -> INFO 003 Writing new channel tx

#################################################################
#######    Generating anchor peer update for Org1MSP   ##########
#################################################################
2017-10-18 05:12:44.874 UTC [common/configtx/tool] main -> INFO 001 Loading configuration
2017-10-18 05:12:44.899 UTC [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2017-10-18 05:12:44.909 UTC [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

#################################################################
#######    Generating anchor peer update for Org2MSP   ##########
#################################################################
2017-10-18 05:12:45.012 UTC [common/configtx/tool] main -> INFO 001 Loading configuration
2017-10-18 05:12:45.025 UTC [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2017-10-18 05:12:45.030 UTC [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update_

- Once the crypto material is generated, let's set up a sample network: 

> ./byfn.sh -m up

_Starting with channel 'mychannel' and CLI timeout of '10000'
Continue (y/n)? y
proceeding ...
Creating network "net_byfn" with the default driver
Creating orderer.example.com
Creating peer1.org2.example.com
Creating peer0.org2.example.com
Creating peer1.org1.example.com
Creating peer0.org1.example.com
Creating cli..._

- If the following screen showed up it means that the setup process has been successful.

_===================== Query on PEER3 on channel 'mychannel' is successful ===================== 

========= All GOOD, BYFN execution completed =========== _

- You may now push ctrl+c to exit the status screen.


## Congratulations! You are now ready for IBM Blockchain Roadshow!




