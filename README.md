# Introduction

Puffchain is a custom blockchain network built on Hyperledger Fabric, designed for secure, efficient, and transparent transactions. It supports modular smart contract execution, decentralized identity management, and seamless integration with enterprise applications.

# Architecture
   Puffchain consists of the following components:
  - Peer Nodes – Maintain the blockchain ledger and execute smart contracts.
  - Orderer Nodes – Manage transactions and consensus.  
  - Certificate Authority (CA) – Handles identity and authentication.
  - Chaincode (Smart Contracts) – Custom business logic deployed on the blockchain.
  - Client Applications – Interact with the network using Fabric SDK.


## Installation


1. **Install Hyperledger Fabric:**
    Follow the official [Hyperledger Fabric installation guide](https://hyperledger-fabric.readthedocs.io/en/release-2.2/install.html).

2. **Prerequisites**
    ## Installing required packages for the JS chaincode
    - Navigate to the `chaincode-javascript` 
    ```bash
        cd chaincode-javascript
    ```
    - Install required node packages as listed in `package.json`
    ```bash
        npm install
    ```

    ## Adding binaries to `PATH`
    - Navigate to the `network` dir
    ```bash
      cd network
    ```
    - Perform the followinf command to add binaries to `PATH`
    ```bash
        export PATH=${PWD}/bin:$PATH
    ```

    ## Setting core config files for the network
    - The core config files are under `config` dir in the root folder of the project
    - commands below should be run while in `network` dir
    ```bash
     export FABRIC_CFG_PATH=$PWD/../config/ 
    ```

2. **Start the network:**
    ## start the network
    - To build the `orgs` and the `peers` of sample fabric network 
    ```bash
      ./network.sh up createChannel
    ```


## Operation

1. **Package our Puffchain minimal implementation chaincode**
    ```bash
        peer lifecycle chaincode package puff.tar.gz --path chaincode-typescript/ --lang node --label puff_1.0
    ```

2. **Deploying chaincode**
    - After we package the puffchain smart contract, we can install the chaincode on our peers. The chaincode needs to be installed on every peer that will endorse a transaction. Because we are going to set the endorsement policy to require endorsements from both Org1 and Org2, we need to install the chaincode on the peers operated by both organizations:
        peer0.org1.example.com
        peer0.org2.example.com
    
    ## ORG 1 (SET ENV VARIABLES)
    ```bash
        export CORE_PEER_TLS_ENABLED=true
        export CORE_PEER_LOCALMSPID=Org1MSP
        export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
        export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
        export CORE_PEER_ADDRESS=localhost:7051
    ```
    ## ORG 1 (command to install the chaincode on the peer)
    ```bash
       peer lifecycle chaincode install puff.tar.gz
    ```
    ## ORG 1 (approve a chaincode definition for organization)
    ```bash
       peer lifecycle chaincode queryinstalled
    ```
    - Set env with copied value as shown below
    ```bash
        export CC_PACKAGE_ID=puff_1.0:69de748301770f6ef64b42aa6bb6cb291df20aa39542c3ef94008615704007f3
    ```
    - Execute command as shown below
    ```bash
        peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name puff --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"

    ```

    ## ORG 2 (SET ENV VARIABLES)
    ```bash
        export CORE_PEER_LOCALMSPID=Org2MSP
        export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0. org2.example.com/tls/ca.crt
        export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
        export CORE_PEER_ADDRESS=localhost:9051
    ```
    ## ORG 2 (command to install the chaincode on the peer)
    ```bash
       peer lifecycle chaincode install puff.tar.gz
    ```
    ## ORG 2 (approve a chaincode definition for organization)
    ```bash
       peer lifecycle chaincode queryinstalled
    ```
    - Set env with copied value as shown below
    ```bash
        export CC_PACKAGE_ID=puff_1.0:69de748301770f6ef64b42aa6bb6cb291df20aa39542c3ef94008615704007f3
    ```
    - Execute command as shown below
    ```bash
        peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name puff --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"

    ```
3. **Check commit readiness**
    - After a sufficient number of organizations have approved a chaincode definition, one organization can commit the chaincode definition to the channel. If a majority of channel members have approved the definition, the commit transaction will be successful and the parameters agreed to in the chaincode definition will be implemented on the channel

    ```bash
        peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name puff --version 1.0 --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --output json
    ```
   
    - if both are ready, execute
    ```bash
        peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name puff --version 1.0 --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"
    ```


# Puffchain Chaincode invoking and query

 **Setting up our Accs**
 ```bash
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n puff --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'
```


**Assigning Roles**
```bash
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n puff --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"AssignRoles","Args":[]}'
```


**Transaction**
```bash
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n puff --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"Transfer","Args":["Client2","Client1","25"]}'
```


**Propose Block**
```bash
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n puff --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"ProposeBlock","Args":["Client2"]}'
```


**Adjusting packers**
```bash
    peer chaincode query -C mychannel -n puff -c '{"Args":["AdjustPackers"]}'
```


**validating blocks**
```bash
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n puff --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"ValidateBlock","Args":["Client2", "ControlBlock_1737331701"]}'
```


**Commit block**
```bash
     peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n puff --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"CommitBlock","Args":["ControlBlock_1737331701"]}'
```
![commit](screenshots/commitblock.png)
