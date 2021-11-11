# Adding a new in running Hyperledger Fabric Network


[![IMAGE ALT TEXT](http://img.youtube.com/vi/cjXJrlP2owc/0.jpg)](http://www.youtube.com/watch?v=cjXJrlP2owc "Video Tutorial")


## Starting the network

### Creating channel

```bash
./network.sh up createChannel -ca -c mychannel -s couchdb
```

### Installing Chaincode

```bash
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

### Setting up environment variable

```bash
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=${PWD}/../config

export ORDERER_CA=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

export CORE_PEER_ADDRESS=localhost:7051

export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt

export CORE_PEER_LOCALMSPID=Org1MSP
```

### Invoke CC

```bash
peer chaincode invoke -n basic -C mychannel -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com  --tls --cafile "$ORDERER_CA" --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt  -c '{"Args":["CreateAsset", "100","red", "20","aditya","100"]}'
```

### Query CC

```bash
peer chaincode query -n basic -C mychannel -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com  -c '{"Args":["ReadAsset", "100"]}'
```

## Setting up the new peer

### Creating MSP Identities for peer1.org1.example.com

```bash
./organizations/fabric-ca/registerPeer1.sh
```

### Starting up the peer container

```bash
docker-compose -f docker/docker-compose-peer1.yaml up -d

```

## Joining existing channel

### Query channel on peer0.org1.example.com

```bash
peer channel list
peer channel fetch -c mychannel newest
CORE_PEER_ADDRESS=localhost:8051 peer channel getinfo -c mychannel
```

### Query channel on peer1.org1.example.com

```bash
CORE_PEER_ADDRESS=localhost:8051 peer channel list
CORE_PEER_ADDRESS=localhost:8051 peer channel fetch -c mychannel newest
CORE_PEER_ADDRESS=localhost:8051 peer channel getinfo -c mychannel
```

### Join channel

```bash
CORE_PEER_ADDRESS=localhost:8051 peer channel join -b ./channel-artifacts/mychannel.block
```

### Query channel on peer1.org1.example.com

```bash
CORE_PEER_ADDRESS=localhost:8051 peer channel list
CORE_PEER_ADDRESS=localhost:8051 peer channel fetch -c mychannel newest
CORE_PEER_ADDRESS=localhost:8051 peer channel getinfo -c mychannel
```

## Chaincode Setup

### Install CC

```bash
export CC_NAME=basic

CORE_PEER_ADDRESS=localhost:8051 peer lifecycle chaincode install ${CC_NAME}.tar.gz
```

### Query installed CC

```bash
CORE_PEER_ADDRESS=localhost:8051 peer lifecycle chaincode queryinstalled
```

### Invoke CC

```bash
CORE_PEER_ADDRESS=localhost:8051 peer chaincode invoke -n basic -C mychannel -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com  --tls --cafile "$ORDERER_CA" --peerAddresses localhost:8051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt  -c '{"Args":["CreateAsset", "200","red", "20","aditya","100"]}'
```

### Query CC

```bash
CORE_PEER_ADDRESS=localhost:8051 peer chaincode query -n basic -C mychannel -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com  -c '{"Args":["ReadAsset", "200"]}'
```
