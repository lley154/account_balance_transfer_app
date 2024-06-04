# Simple Account Transfer Application in Hyperledger Fabric
Note: You must complete Lab #1 and Lab #2 first which will include the required binary files. 
```
sudo reboot ## restart your vm to free up resources
```
```
sudo dpkg --configure -a
```
```
sudo apt install jq
```
```
cd fabric-samples
```
```
git clone https://github.com/lley154/account_balance_transfer_app.git
```

## Part 1: Using peer command to interact with Chaincode
Set environment variables:
```
cd test-network
```
```
./network.sh down
```
```
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```
Start the network channel
```
sudo ./network.sh up createChannel -ca -s couchdb
```
```
sudo chmod a+rwx -R organizations  ## this is only done for lab env
```
```
sudo chmod a+rwx -R ../config  ## this is only done for lab env
```
Deploy the chaincode
```
./network.sh deployCC -ccn balance_transfer -ccv 1.0 -ccp ../account_balance_transfer_app/balance_transfer -ccl javascript
```

Test to see if you see the mychannel and installed chaincode
```
peer channel list
```
```
peer lifecycle chaincode queryinstalled
```

Invoke chaincode:
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
```
Initialize the account with some funds
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
    -c '{"function":"initAccount","Args":["U1","150"]}'
```
Now transfer between User1 (U1) and Admin (A1)
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

## Part 2: Using Fabric Application Gateway, Wallets to interact with Chaincode

Go into the account balance transfer app directory
```
cd ../account_balance_transfer_app/balance_transfer_app
```
Install node node modules
```
npm install
```
First you have to enroll the admin user
```
node enrollUser.js 'CAAdmin@org1.example.com' admin adminpw
```

Now register user as follows
```
node registerUser.js 'CAAdmin@org1.example.com' 'User1@org1.example.com' '{"secret": "userpw"}'
```
Then enroll user
```
node enrollUser.js 'User1@org1.example.com' 'User1@org1.example.com' userpw
```

Using User1 credentials create account acc1
```
node submitTransaction.js 'User1@org1.example.com' initAccount acc1 100
```

To check the balance
```
node submitTransaction.js 'User1@org1.example.com' listAccounts
```
Register and enroll User2
```
node registerUser.js 'CAAdmin@org1.example.com' 'User2@org1.example.com' '{"secret": "userpw2"}'

node enrollUser.js 'User2@org1.example.com' 'User2@org1.example.com' userpw2
```
Using User 2 create acc2
```
node submitTransaction.js 'User2@org1.example.com' initAccount acc2 200
```
```
node submitTransaction.js 'User2@org1.example.com' listAccounts
```
Now transfer 50 from acc2 to acc1
```
node submitTransaction.js 'User2@org1.example.com' transfer acc2 acc1 50
```
```
node submitTransaction.js 'User2@org1.example.com' listAccounts
```
```
node submitTransaction.js 'User1@org1.example.com' listAccounts
```
Look and confirm there are 3 wallets created and have the certificate and private key for each.
```
ls -l wallet
total 12
-rw-rw-r-- 1 ubuntu ubuntu 1139 Feb 10 21:25 CAAdmin@org1.example.com.id
-rw-rw-r-- 1 ubuntu ubuntu 1159 Feb 10 21:26 User1@org1.example.com.id
-rw-rw-r-- 1 ubuntu ubuntu 1159 Feb 10 21:30 User2@org1.example.com.id

cat wallet/User1@org1.example.com.id | jq
{
  "credentials": {
    "certificate": "-----BEGIN CERTIFICATE-----\nMIICHTCCAcSgAwIBAgIUZ5brwciSh1l1DFM65ynf0KOgiMYwCgYIKoZIzj0EAwIw\ncDELMAkGA1UEBhMCVVMxFzAVBgNVBAgTDk5vcnRoIENhcm9saW5hMQ8wDQYDVQQH\nEwZEdXJoYW0xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHDAaBgNVBAMTE2Nh\nLm9yZzEuZXhhbXBsZS5jb20wHhcNMjQwMjEwMjA1MTAwWhcNMjUwMjA5MjEyNzAw\nWjAyMQ8wDQYDVQQLEwZjbGllbnQxHzAdBgNVBAMMFlVzZXIxQG9yZzEuZXhhbXBs\nZS5jb20wWTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAARF+IznR6K0k4q7YvT/xRdP\nAZLSV2n9dgu2+wU2ICPWSIrDaJye5ftj11fNk44T+t41v8d0HjXRhnud2Nmwg8PU\no3oweDAOBgNVHQ8BAf8EBAMCB4AwDAYDVR0TAQH/BAIwADAdBgNVHQ4EFgQUTu8K\nARFWVKFvRF7MXjuxi5WOnRcwHwYDVR0jBBgwFoAUVnLbCgv2Rj5ZiV+uhgCd6ttH\na5MwGAYIKgMEBQYHCAEEDHsiYXR0cnMiOnt9fTAKBggqhkjOPQQDAgNHADBEAiAN\nPjnDeBziTuUNE8n9uvsyCKsZeaikLTcg9f4X2h+sJAIgPJwZzZ4IaNiqOrjAIue3\nNeLp2ZP8PTfPcR9LmYtAVE4=\n-----END CERTIFICATE-----\n",
    "privateKey": "-----BEGIN PRIVATE KEY-----\r\nMIGHAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBG0wawIBAQQgL6QmkRWcMjOhRYtP\r\nXD6lsQRRn8BoD79IANF5895L8dGhRANCAARF+IznR6K0k4q7YvT/xRdPAZLSV2n9\r\ndgu2+wU2ICPWSIrDaJye5ftj11fNk44T+t41v8d0HjXRhnud2Nmwg8PU\r\n-----END PRIVATE KEY-----\r\n"
  },
  "mspId": "Org1MSP",
  "type": "X.509",
  "version": 1
}
```
Note: If your network is restarted, you will need to remove the wallet directory (and regsiter and enroll again) because the public/private keys will no longer match the issuing CA on your network.


## Part 3: Viewing World State Data in CouchDB

We can port forward from our local machine to the virutal machine and then access the CouchDB UI using port 5984.

ssh usage:
```
ssh -L local_port:destination_server_ip:remote_port ssh_server_hostname
```
An example on how to connect is
```
ssh -i lab2.pem -L 5984:ec2-54-91-100-220.compute-1.amazonaws.com:5984 ubuntu@ec2-54-91-100-220.compute-1.amazonaws.com
```
Now, we can access the CouchDB UI locally by using the browser and go to:
```
http://localhost:5984/_utils/#login
username: admin
password: adminpw
```
Go to mychannel_balance_transfer

Select Account A1 to see the current state of A1, U1, acc1 & acc2




