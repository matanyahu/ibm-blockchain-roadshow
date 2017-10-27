# IBM Blockchain Roadshow Lab 2

## This tutorial will give you a step-by-step overview of setting up your own Hyperledger Fabric network

### **1. Prerequisites**

- Initial setup of Vagrant box and environment complete
- Marbles environment shut down and cleaned up
- Access to the Internet

### **2. Log in to your Vagrant Box:**

- run Vagrant box:

	```
  vagrant up
	```

- The setup should be complete very fast as your VM is already in place. You should be able to ssh into the Vagrant VM just created:

	```
  vagrant ssh
	```

- This should log you into the box to the following folder: **/opt/gopath/src/github.com/hyperledger/fabric**
- Verify if you can use these commands:

```
cryptogen
configtxgen
configtxlator
peer
```

- If they are not recognized, export your binary folder again and try afterwards:

```
export PATH=/opt/gopath/src/github.com/hyperledger/fabric/bin:$PATH
```

- We will be using remaining commands inside the box.

### **3. Let's now clone IBM Blockchain Roadshow repository**

- Clone the current git repository:

```
git clone https://github.com/matanyahu/ibm-blockchain-roadshow/
```

### **4. Let's start from generating network artifacts**

- Go to **ibm-blockchain-roadshow/scripts/lab2/**

```
cd ibm-blockchain-roadshow/scripts/lab2/
```

- First, we will use the **cryptogen** tool to generate the cryptographic material (x509 certs) for our various network entities. These certificates are representative of identities, and they allow for sign/verify authentication to take place as our entities communicate and transact. Cryptogen consumes a file - **crypto-config.yaml** - that contains the network topology and allows us to generate a set of certificates and keys for both the Organizations and the components that belong to those Organizations. Each Organization is provisioned a unique root certificate (ca-cert) that binds specific components (peers and orderers) to that Org. By assigning each Organization a unique CA certificate, we are mimicking a typical network where a participating Member would use its own Certificate Authority. Transactions and communications within Hyperledger Fabric are signed by an entity’s private key (keystore), and then verified by means of a public key (signcerts).

- Before running the tool, let’s take a quick look at a snippet from the crypto-config.yaml. Pay specific attention to the “Name”, “Domain” and “Specs” parameters under the OrdererOrgs header:

```
OrdererOrgs:

  - Name: Orderer
    Domain: example.com

    Specs:
      - Hostname: orderer

PeerOrgs:

  - Name: Org1
    Domain: org1.example.com

    Template:
      Count: 1

    Users:
      Count: 1
```

- The naming convention for a network entity is as follows - “{{.Hostname}}.{{.Domain}}”. So using our ordering node as a reference point, we are left with an ordering node named - orderer.example.com that is tied to an MSP ID of Orderer.

- Let's generate the necessary material now:

```
./1-generate-crypto-material.sh
```

- After we run the cryptogen tool, the generated certificates and keys will be saved to a folder titled crypto-config.

### **5. Let's create genesis block, a channel and an anchor:**

- The **configtxgen** tool is used to create four configuration artifacts:

      -  orderer genesis block,
      -  channel channel configuration transaction,
      -  and an anchor peer transaction - one for each Peer Org.

- The orderer block is the Genesis Block for the ordering service, and the channel transaction file is broadcast to the orderer at Channel creation time. The anchor peer transaction, as the name might suggest, specify Org’s Anchor Peer on this channel.

- **Configtxgen** consumes a file - **configtx.yaml** - that contains the definitions for the sample network. There are three members - one Orderer Org (OrdererOrg) and a Peer Org (Org1) managing and maintaining a peer node. This file also specifies a consortium - SampleConsortium - consisting of our Org and extensible to future new organizations. Pay specific attention to the “Profiles” section at the top of this file. You will notice that we have two unique headers. One for the orderer genesis block - TwoOrgsOrdererGenesis - and one for our channel - OneOrgChannel.

```
Profiles:

    OneOrgOrdererGenesis:
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
    OneOrgChannel:
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1

Organizations:

    # SampleOrg defines an MSP using the sampleconfig.  It should never be used
    # in production but may be used as a template for other definitions
    - &OrdererOrg
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: OrdererOrg

        # ID to load the MSP definition as
        ID: OrdererMSP

        # MSPDir is the filesystem path which contains the MSP configuration
        MSPDir: crypto-config/ordererOrganizations/example.com/msp

    - &Org1
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: Org1MSP

        # ID to load the MSP definition as
        ID: Org1MSP

        MSPDir: crypto-config/peerOrganizations/org1.example.com/msp

        AnchorPeers:
            # AnchorPeers defines the location of peers which can be used
            # for cross org gossip communication.  Note, this value is only
            # encoded in the genesis block in the Application section context
            - Host: peer0.org1.example.com
              Port: 7051

Orderer: &OrdererDefaults

    # Orderer Type: The orderer implementation to start
    # Available types are "solo" and "kafka"
    OrdererType: solo

    Addresses:
        - orderer.example.com:7050

    # Batch Timeout: The amount of time to wait before creating a batch
    BatchTimeout: 2s

    # Batch Size: Controls the number of messages batched into a block
    BatchSize:

        # Max Message Count: The maximum number of messages to permit in a batch
        MaxMessageCount: 10

        # Absolute Max Bytes: The absolute maximum number of bytes allowed for
        # the serialized messages in a batch.
        AbsoluteMaxBytes: 99 MB

        # Preferred Max Bytes: The preferred maximum number of bytes allowed for
        # the serialized messages in a batch. A message larger than the preferred
        # max bytes will result in a batch larger than preferred max bytes.
        PreferredMaxBytes: 512 KB

    Kafka:
        # Brokers: A list of Kafka brokers to which the orderer connects
        # NOTE: Use IP:port notation
        Brokers:
            - 127.0.0.1:9092

    # Organizations is the list of orgs which are defined as participants on
    # the orderer side of the network
    Organizations:

Application: &ApplicationDefaults

    # Organizations is the list of orgs which are defined as participants on
    # the application side of the network
    Organizations:
```

- Let's generate a genesis block first:

```
./2-genesis-block.sh
```

- Then, let's set up channel configuration:

```
./3-channel-config.sh
```

- Finally, let's define anchor peers:

```
./4-anchor-peer.sh
```

### **6. Let's run the network now and join peers to the channel**

- We will leverage a docker-compose script to spin up our network. The docker-compose file references the images that we have previously downloaded, and bootstraps the orderer with our previously generated **genesis.block**.

- Let's inspect **docker-compose** file that we will use to bootstrap the network. As you can see, present are the CA, a Solo Orderer, one Peer (that we will also use for Endorsing the chaincode) and a CLI container that is only used to forward set up commands to the network:

```
version: '2'

networks:
  basic:

services:
  ca.example.com:
    image: hyperledger/fabric-ca
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca.example.com
    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/org1.example.com-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/a22daf356b2aab5792ea53e35f66fccef1d7f1aa2b3a2b92dbfbf96a448ea26a_sk -b admin:adminpw -d'
    volumes:
      - ./crypto-config/peerOrganizations/org1.example.com/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.example.com
    networks:
      - basic

  orderer.example.com:
    container_name: orderer.example.com
    image: hyperledger/fabric-orderer
    environment:
      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/msp/orderer/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 7050:7050
    volumes:
        - ./config/:/etc/hyperledger/configtx
        - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/:/etc/hyperledger/msp/orderer
        - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/:/etc/hyperledger/msp/peerOrg1
    networks:
      - basic

  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    image: hyperledger/fabric-peer
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_PEER_ID=peer0.org1.example.com
      - CORE_LOGGING_PEER=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      # # the following setting starts chaincode containers on the same
      # # bridge network as the peers
      # # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_basic
      #- CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      #- CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      #- CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      #- CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    # command: peer node start --peer-chaincodedev=true
    ports:
      - 7051:7051
      - 7053:7053
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/msp/peer
        - ./crypto-config/peerOrganizations/org1.example.com/users:/etc/hyperledger/msp/users
        - ./config:/etc/hyperledger/configtx
    depends_on:
      - orderer.example.com
      #- couchdb
    networks:
      - basic

  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
      - CORE_CHAINCODE_KEEPALIVE=10
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
        - /var/run/:/host/var/run/
        - ./chaincode:/opt/gopath/src/github.com/
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
    networks:
        - basic
```

- Let's run the network now:

```
./5-bootstrap-network.sh
```

- Next, we are going to pass in the generated channel configuration transaction artifact that we created iearlier (we called it channel.tx) to the orderer as part of the create channel request:

```
./6-channel-create.sh
```

- Now let’s join peer0.org1.example.com to the channel.

```
./7-channel-join.sh
```

### **7. Let's install and instantiate chaincode onto our network**

- Applications interact with the blockchain ledger through chaincode. As such we need to install the chaincode on every peer that will execute and endorse our transactions, and then instantiate the chaincode on the channel.

- First, install the sample Go code onto one of the four peer nodes. This command places the source code onto our peer’s filesystem.

```
./8-chaincode-install.sh
```

- Next, instantiate the chaincode on the channel. This will initialize the chaincode on the channel, set the endorsement policy for the chaincode, and launch a chaincode container for the targeted peer. Take note of the -P argument. This is our policy where we specify the required level of endorsement for a transaction against this chaincode to be validated.

- In the command below you’ll notice that we specify our policy as -P "OR ('Org0MSP.member','Org1MSP.member')". This means that we need “endorsement” from a peer belonging to Org1 OR Org2 (i.e. only one endorsement). If we changed the syntax to AND then we would need two endorsements.

```
./9-chaincode-instantiate.sh
```

- Let's now invoke the chaincode by using an **initLedger** function which will create couple of assets for us:

```
./10-chaincode-invoke.sh
```

- Finally, let's query the chaincode by using an **queryAllCars** function which will query all assets from the blockchain:

```
./11-chaincode-query.sh
```

### **8. Shutting down and cleaning up the environment for the next excercise:**

- Execute the teardown command

```
./teardown.sh
```

## Congratulations! You have now finished Lab 2!
To return back to the main menu, click [here](https://github.com/matanyahu/ibm-blockchain-roadshow)
