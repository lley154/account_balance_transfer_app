# Simple Account Transfer Application in Hyperledger Fabric
Note: You must complete Lab #1 first which will include the required binary files. 
```
cd fabric-samples
git clone https://github.com/lley154/account_balance_transfer_app.git
```

## Part 1: Using peer command to interact with Chaincode

```
cd fabric-samples/test-network
./network.sh down 
./network.sh up createChannel -ca -s couchdb
./network.sh deployCC -ccn balance_transfer -ccv 1.0 -ccp ../account_balance_transfer_app/balance_transfer -ccl javascript
```

Set environment variables:
```
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
```
Test to see if you see the mychannel and installed chaincode
```
peer channel list
peer lifecycle chaincode queryinstalled
```

Invoke Command:
```
peer chaincode invoke \
    -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
    -C mychannel \
    -n balance_transfer \
    --peerAddresses localhost:7051 \
    --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
    --peerAddresses localhost:9051 \
    --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
    -c '{"function":"initAccount","Args":["A1","100"]}'
```
List the initial balance
```
peer chaincode query \
    -C mychannel \
    -n balance_transfer \
    -c '{"function":"listAccounts", "Args":[]}'
```

Repeat same invoke command with -c '{"function":"setBalance","Args":["A1","150"]}'
```
peer chaincode invoke \
    -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
    -C mychannel \
    -n balance_transfer \
    --peerAddresses localhost:7051 \
    --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
    --peerAddresses localhost:9051 \
    --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
    -c '{"function":"setBalance","Args":["A1","150"]}'
```
Now, list the new balance
```
peer chaincode query \
    -C mychannel \
    -n balance_transfer \
    -c '{"function":"listAccounts", "Args":[]}'
```
Change user to User1:
```
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp

peer chaincode invoke \
    -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
    -C mychannel \
    -n balance_transfer \
    --peerAddresses localhost:7051 \
    --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
    --peerAddresses localhost:9051 \
    --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
    -c '{"function":"initAccount","Args":["U1","150"]}'


peer chaincode invoke \
    -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
    -C mychannel \
    -n balance_transfer \
    --peerAddresses localhost:7051 \
    --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt \
    --peerAddresses localhost:9051 \
    --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt \
    -c   '{"function":"transfer","Args":["U1","A1", "100"]}'
```
List the final balance of U1
```
peer chaincode query \
    -C mychannel \
    -n balance_transfer \
    -c '{"function":"listAccounts", "Args":[]}'
```
And then the final balance of A1
```
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

peer chaincode query \
    -C mychannel \
    -n balance_transfer \
    -c '{"function":"listAccounts", "Args":[]}'
```

