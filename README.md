# Welcome to IBM Blockchain Roadshow!

## This tutorial will help you to create a local development environment which will be used during the workshop.

### **0. Agenda***

| Schedule | Description |
| --- | --- |
| 0:15 | Welcome and Introduction to the workshop |
| 0:20 | Chapter 1 : Blockchain Explained |
| 0:40 | Lab 1 : United Marbles - an Asset Management scenario |
| 0:20 | Chapter 2 : Blockchain Explored |
| 0:40 | Lab 2 : Setting up own blockchain network |
| 0:20 | Chapter 3 : Blockchain Unchained |
| 0:40 | Lab 3 : Writing and deploying own chaincode |

### **1. Prerequisites**

- Access to the Internet
- PC with a 64-bit processor and virtualization bit enabled (**vt-x** or similar), 8Gb of RAM and 30GB of available free space.
- Oracle VirtualBox **5.0 or 5.1** installed (do not install version 5.2 yet): https://www.virtualbox.org/wiki/Downloads
- Vagrant installed: https://www.vagrantup.com/downloads.html
- Git installed: https://git-scm.com/downloads

### **2. Before we launch a local environment, let's download and modify Vagrant file in order to host a web app called Marble to our host:**

- Clone the current Hyperledger Fabric git repository:

	```
  git clone https://github.com/hyperledger/fabric
	```

- Go to the folder created by the repo:

	```
  cd fabric/devenv
	```

- Open **Vagrantfile** with your favourite editor. Add the following line of code in port forwarding area (you will see similar lines already written in for other communition ports such as 7050, 7051...) and save the file:

  ```
  config.vm.network :forwarded_port, guest: 3001, host: 3001, id: "marbles", host_ip: "localhost", auto_correct: true
    ```

### **3. Now let's create a Vagrant box (a virtual machine) which will be used as a sandbox nested on our PC for all Hyperledger Fabric tests**

- run Vagrant box:

	```
  vagrant up
	```

- You may read and compare logs from vagrant VM creation process on your machine with [this file](./logs/vagrant-up.log)
- Go get coffee... this will take a few minutes.
- Once complete, you should be able to ssh into the Vagrant VM just created:

	```
  vagrant ssh
	```

- This should log you into the box to the following folder: **/opt/gopath/src/github.com/hyperledger/fabric**
- We will be using remaining commands inside the box.

### **3. Let's now clone some useful scripts which will automate the process of downloading necessary binaries and Docker images which will be used during the workshop**

- We will now install the Hyperledger Fabric platform-specific binaries. Inside Vagrant box, run the following script:

	```
  curl -sSL https://raw.githubusercontent.com/hyperledger/fabric/release/scripts/bootstrap-1.0.4.sh | bash
	```

- You may read and compare logs on your machine with [this file](./logs/fabric-binaries.log)
- This process may require yet another cup of coffee...after the end, the script should create a new folder called **/bin** with necessary binaries and download Docker images for Hyperledger Fabric.
- Now, let's create an environment variable so that binaries in **/bin** folder can be picked up without fully qualifying the path to each binary:

	```
  export PATH=/opt/gopath/src/github.com/hyperledger/fabric/bin:$PATH
	```

- Check if you can invoke these binaries from the folder : **/opt/gopath/src/github.com/hyperledger/fabric**, e.g. by running all of these:

	```
  cryptogen
  configtxgen
  configtxlator
  peer
	```

- Also check if Docker images are ready:

	```
  docker images
	```

```bash
REPOSITORY                    TAG                  IMAGE ID             CREATED              SIZE
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
busybox                        latest              54511612f1c4        4 weeks ago         1.13MB
```

### **4. If everything went smoothly, we should now be able to run a test Hyperledger Fabric network on our Vagrant box. It is not necessary that you understand what is happening behind the scenes as everything will be explained again during the workshop.**

- We will be executing all scripts $HOME folder, therefore, you may switch back to this folder before you continue further:

```
cd $HOME
```

- Clone Hyperledger Fabric Samples inside your Vagrant box:

	```
  git clone https://github.com/hyperledger/fabric-samples
	```

- Go to **fabric-samples/first-network/**
- Now, let's run an automated script that will firstly generate some necessary crypto for our network:

	```
  ./byfn.sh -m generate
	```

- Confirm default settings by typing "y" and pushing enter to continue:

```bash
  Generating certs and genesis block for with channel 'mychannel' and CLI timeout of '10000'
  Continue (y/n)? y
  proceeding ...
```

- You may read and compare logs on your machine with [this file](./logs/byfn-crypto-generate.log)
- Once the crypto material is generated, let's set up a sample network:

	```
	./byfn.sh -m up
	```

```bash
Starting with channel 'mychannel' and CLI timeout of '10000'
Continue (y/n)? y
proceeding ...
Creating network "net_byfn" with the default driver
Creating orderer.example.com
Creating peer1.org2.example.com
Creating peer0.org2.example.com
Creating peer1.org1.example.com
Creating peer0.org1.example.com
Creating cli...
```

- You may read and compare logs on your machine with [this file](./logs/byfn-up.log)
- If the following screen showed up it means that the setup process has been successful.

```bash
========= All GOOD, BYFN execution completed ===========


 _____   _   _   ____   
| ____| | \ | | |  _ \  
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```

- You may now push ctrl+c to exit the status screen.
- Before turning off a Vagrant box, make sure that you shut down your Hyperledger Fabric network:

```bash
./byfn.sh -m down
```

- If the following screen showed up it means that the setup process has been successful.

```bash
ubuntu@hyperledger-devenv:5226188:/opt/gopath/src/github.com/hyperledger/fabric/fabric-samples/first-network$ ./byfn.sh -m down
Stopping with channel 'mychannel' and CLI timeout of '10000'
Continue (y/n)? y
proceeding ...
WARNING: The CHANNEL_NAME variable is not set. Defaulting to a blank string.
WARNING: The DELAY variable is not set. Defaulting to a blank string.
WARNING: The TIMEOUT variable is not set. Defaulting to a blank string.
Stopping cli ... done
Stopping peer1.org1.example.com ... done
Stopping peer1.org2.example.com ... done
Stopping peer0.org1.example.com ... done
Stopping peer0.org2.example.com ... done
Stopping orderer.example.com ... done
Removing cli ... done
Removing peer1.org1.example.com ... done
Removing peer1.org2.example.com ... done
Removing peer0.org1.example.com ... done
Removing peer0.org2.example.com ... done
Removing orderer.example.com ... done
Removing network net_byfn
WARNING: The CHANNEL_NAME variable is not set. Defaulting to a blank string.
WARNING: The DELAY variable is not set. Defaulting to a blank string.
WARNING: The TIMEOUT variable is not set. Defaulting to a blank string.
Removing network net_byfn
WARNING: Network net_byfn not found.
fd59092ac2d3
0b486cea30ae
0b818e65860a
Untagged: dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab:latest
Deleted: sha256:6d232a7d6cee6d6f7f89d66fb93193ab199c7f6c81137c5a3ae6aec83e19bf0f
Deleted: sha256:8e6d9a0c3e53f3f165e152f631f4e446634ef9df20644ca1834083562de2a5b1
Deleted: sha256:8ed478c7231988da96d24b47dccdc4211ab134f26ccc6bdc1643bf7bdaad8185
Deleted: sha256:e8a2885608815bd5a5496f86cd846155eaa36fb6806036459acd177fc0548f36
Deleted: sha256:7935600004d0911a62fa57f8cff1e16c46663ebb936e452cf58cfe2cc9a9f04c
Deleted: sha256:9375d01dfbba9292ce8eee8e1ea909c9739166a1a7697720ad70b213140dc75c
Deleted: sha256:09254ae16755898acc30ea5354ab0c0d8a877fc230b7c75e02dd0a0d93a2ab7a
Untagged: dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9:latest
Deleted: sha256:11dfb8e3dd76ff430c2852a1497b6371c1f8e5994419ff543d0bcc7c56f4f96e
Deleted: sha256:013bbc759e59790bf4aec8a8392c285672d109304d98dac3fba1592575523e07
Deleted: sha256:775491974b0c616c6110b72917fa38a8bbe034d727ec2cecb3d602a355ec58ce
Deleted: sha256:e0b48d7f9901d28a38f880d37db42f048e87f5d931e91f554e5eb66ffd2facdf
Deleted: sha256:864cf7b1edfab6c2addcfb42caef91e7d6d9edd25fb84e97a3afbf367a2a1561
Deleted: sha256:3431fbda47aaaa18bbb9bcfe543d50ef9d435e10e40bea5efbe9e3cd687df314
Deleted: sha256:bbc4f02b5d58c5c64e684cfc3123201ea9f1c73054dbf4a21aa95e0f07d75e31
Untagged: dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b:latest
Deleted: sha256:c59717dec76c7160cf715915c30996d0a73c6c831464fefc481336dfcb4b1c7e
Deleted: sha256:01ac06f54748f4c1600169c5a2f3d6018a663641b4cb993555fe87b2981bd7f5
Deleted: sha256:81ab3c04d6f1d5989badf3957e4789fa3a10ea82527867b7437ff3a0e2a6fdb9
Deleted: sha256:615a1f5c119c07908015e5680d4c609d17f011cf7b1ee4dcb9336b74d8926bb5
Deleted: sha256:eb7ddbd9c53f0106a97186a7bccf857b119d74eaa423cd06d89c195e677fda6c
Deleted: sha256:73e1ad05e0e43a5805abff5dc13b5080132786427893fb715c3cb76433842630
Deleted: sha256:710ed0193bd10591b4b468f17b228805b39e28b9fefa37fa7885fc181e24ce9a
ubuntu@hyperledger-devenv:5226188:/opt/gopath/src/github.com/hyperledger/fabric/fabric-samples/first-network$
```

- If you want to exit the vagrant box, type "exit" in the command line.
- You may either suspend the VM or turn if off by typing one of the following commands :

```
vagrant suspend
vagrant halt
```


## Congratulations! You are now ready for IBM Blockchain Roadshow!

You may now want to go to [Lab 1](https://github.com/matanyahu/ibm-blockchain-roadshow/tree/master/scripts/lab1), [Lab 2](https://github.com/matanyahu/ibm-blockchain-roadshow/tree/master/scripts/lab2), or [Lab 3](https://github.com/matanyahu/ibm-blockchain-roadshow/tree/master/scripts/lab3)
