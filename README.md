# Fabric-prod-network

A simple HyperLedger Fabric Production sample network.

CA Deployment setup instruction steps.

1. `mkdir Fabric-prod-network`

2. Download the latest CA binaries from https://github.com/hyperledger/fabric-ca/releases and extract it into the folder created in step 1.

3. Setup Fabric CA client:

   `cd Fabric-prod-network`

   `mkdir fabric-ca-client`

   `cd fabric-ca-client`

   `mkdir tls-ca org1-ca tls-root-cert`

   Copy the fabric-ca-client binary into the fabric-ca-client folder.

4. Deploy TLS CA:

   `cd ..`

   `mkdir fabric-ca-server-tls`

   copy the fabric-ca-server binary into fabric-ca-server-tls folder.

   `cd fabric-ca-server-tls`

   `./fabric-ca-server init -b tls-admin:tls-adminpw`

5. Open _fabric-ca-server-tls/fabric-ca-server-config.yaml_ and modify it:

   1. change tls.enabled: false to true
   2. assign ca.name: tls-ca
   3. edit csr.hosts if needed
   4. remove signing.profiles.ca block and leave only tls block under signing.profiles.
   5. save

6. delete _fabric-ca-server-tls/ca-cert.pem_ file & entire _fabric-ca-server-tls/msp_ folder, then run `./fabric-ca-server start`

7. copy _fabric-ca-server-tls/ca-cert.pem_ to _fabric-ca-client/tls-root-cert/tls-ca-cert.pem_ (file name changed for clarity)

8. open new terminal tab:

   `cd Fabric-prod-network/fabric-ca-client`

   `export FABRIC_CA_CLIENT_HOME=$PWD`

   `./fabric-ca-client enroll -d -u https://tls-admin:tls-adminpw@localhost:7054 --tls.certfiles tls-root-cert/tls-ca-cert.pem --enrollment.profile tls --csr.hosts 'kky-dell,*localhost' --mspdir tls-ca/tlsadmin/msp`

   `./fabric-ca-client register -d --id.name rcaadmin --id.secret rcaadminpw -u https://localhost:7054 --tls.certfiles tls-root-cert/tls-ca-cert.pem --mspdir tls-ca/tlsadmin/msp`

   `./fabric-ca-client enroll -d -u https://rcaadmin:rcaadminpw@localhost:7054 --tls.certfiles tls-root-cert/tls-ca-cert.pem --enrollment.profile tls --csr.hosts 'kky-dell,*localhost' --mspdir tls-ca/rcaadmin/msp`

   rename the file in _fabric-ca-client/tls-ca/rcaadmin/msp/keystore_ folder to _key.pem_

9. Deploy orgnization CA:

   `cd ..`

   `mkdir fabric-ca-server-org1`

   copy the fabric-ca-server binary in _fabric-ca-server-org1_ folder

   `cd fabric-ca-server-org1`

   `mkdir tls`

   `cp ../fabric-ca-client/tls-ca/rcaadmin/msp/signcerts/cert.pem tls && cp ../fabric-ca-client/tls-ca/rcaadmin/msp/keystore/key.pem tls`

   `./fabric-ca-server init -b rcaadmin:rcaadminpw`

10. Open _fabric-ca-server-org1/fabric-ca-server-config.yaml_ and modify it:

    1. change port from 7054 to 7055
    2. change tls.enabled: false to true
    3. assign tls.certfile: tls/cert.pem
    4. assign tls.keystore: tls/key.pem
    5. assign ca.name: org1-ca
    6. edit csr.hosts if needed
    7. operations.listenAddress: port change from 9443 to 9444
    8. save

11. delete _fabric-ca-server-org1/ca-cert.pem_ file & entire _fabric-ca-server-org1/msp_ folder, then run `./fabric-ca-server start`

12. open new terminal tab:

    `cd Fabric-prod-network/fabric-ca-client`

    `export FABRIC_CA_CLIENT_HOME=$PWD`

    `./fabric-ca-client enroll -d -u https://rcaadmin:rcaadminpw@localhost:7055 --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost' --mspdir org1-ca/rcaadmin/msp`

    rename the file in _fabric-ca-client/org1-ca/rcaadmin/msp/keystore_ folder to _org1-key.pem_

Upcoming to do:

1. Registering and enrolling identities with CA
2. Documenting the instruction steps.
