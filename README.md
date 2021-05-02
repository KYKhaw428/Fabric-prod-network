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

#############################################################################

Registering and enrolling identities instruction steps.

1. Start the fabric-ca-server-org1 as stated in step 11 above, then open a new terminal tab:

   `cd Fabric-prod-network/fabric-ca-client`

   `export FABRIC_CA_CLIENT_HOME=$PWD`

2. To register an identity

   `./fabric-ca-client register -d --id.name org1admin --id.secret org1adminpw -u https://kky-dell:7055 --mspdir ./org1-ca/rcaadmin/msp --id.type admin --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

3. To enroll an identity

   `./fabric-ca-client enroll -u https://org1admin:org1adminpw@kky-dell:7055 --mspdir ./org1.example.com/msp --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

4. create config.yaml file inside _org1.example.com/msp_ folder with values of:

   ```
   NodeOUs:
       Enable: true
       ClientOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: client
       PeerOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: peer
       AdminOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: admin
       OrdererOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: orderer
   ```

5. i) rename _org1.example.com/msp/signcerts/cert.pem_ to _org1-admin-cert.pem_

   ii) rename file inside _org1.example.com/msp/keystore_ to _org1-admin-key.pem_

6. Create a tlscacerts folder and copy the tls-root-cert into this folder.

   `mkdir org1.example.com/msp/tlscacerts`

   `cp tls-root-cert/tls-ca-cert.pem org1.example.com/msp/tlscacerts`

7. Create orgMsp and localMsp folder.

   `mkdir -p org1.example.com/orgMsp/msp org1.example.com/localMsp/msp`

8. Prepare the orgMsp files

   `cp org1.example.com/msp/config.yaml org1.example.com/orgMsp/msp && cp -R org1.example.com/msp/cacerts org1.example.com/orgMsp/msp && cp -R org1.example.com/msp/tlscacerts org1.example.com/orgMsp/msp`

9. Prepare the localMsp files

   `cp org1.example.com/msp/config.yaml org1.example.com/localMsp/msp && cp -R org1.example.com/msp/cacerts org1.example.com/localMsp/msp && cp -R org1.example.com/msp/tlscacerts org1.example.com/localMsp/msp && cp -R org1.example.com/msp/keystore org1.example.com/localMsp/msp && cp -R org1.example.com/msp/signcerts org1.example.com/localMsp/msp`

NOTES UP TO THIS POINT:

1. Things are looking okay and similar to files returned from the documentations.
2. If the next few steps don't work, then might have to register and enroll peer identity instead of --id.type admin identities as in step (2) above.

#############################################################################

Deploy the peer

1. Get binary and config files.

   Download the peer binaries and configuration files from https://github.com/hyperledger/fabric/releases and extract into _Fabric-prod-network_ and rename the file as _fabric_.

2. Add the location of the peer binary to PATH environment variable

   `cd Fabric-prod-network`

   `export PATH=$PWD/fabric/bin:$PATH`

3. Create the following folder structure

   ```
   ├── organizations
        └── peerOrganizations
            └── org1.example.com
                ├── msp
                └── peers
                    └── peer0.org1.example.com
                        ├── msp
                        └── tls
   ```

   `mkdir -p organizations/peerOrganizations/org1.example.com/msp organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls`

4. Prepare TLS certificates and peer local MSP files.

   `cp -R fabric-ca-client/org1.example.com/localMsp/msp/tlscacerts/tls-ca-cert.pem organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tls-cert.pem`

   `cp -R fabric-ca-client/org1.example.com/localMsp/msp/signcerts/org1-admin-cert.pem organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/peer0-cert.pem`

   `cp -R fabric-ca-client/org1.example.com/localMsp/msp/keystore/org1-admin-key.pem organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/peer0-key.pem`

   `cp -R fabric-ca-client/org1.example.com/localMsp/msp organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com`

5. Edit _core.yaml_ in _fabric/config_ with the respective values:

   ```
   peer.tls.enabled: true

   peer.tls.rootcert.file: ../../organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tls-cert.pem

   peer.tls.cert.file: ../../organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/peer0-cert.pem

   peer.tls.key.file: ../../organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/peer0-key.pem

   peer.mspConfigPath: ../../organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp

   peer.fileSystemPath: ../../organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/storage

   operations.listenAddress: 127.0.0.1:9445
   ```

6. Create a peer storage directory for your ledger

   `mkdir -p organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/storage`

7. Start the peer

   `export FABRIC_CFG_PATH=$PWD/fabric/config`

   `cd fabric/bin`

   `./peer node start`

NOTES UP TO THIS POINT:

1. Peer is able to start, but code is [nodeCmd] serve -> INFO 01f instead of [nodeCmd] serve -> INFO 017 as stated in documentations. Might not be a big issue, but something to check upon.

2. Output from peer states there are No active channels passed. (which is understandable because we have not joined our peer to any channel yet, but hope it is not an error for the next steps ahead.)

3. Might need to edit core.yaml's External endpoint, as it is a warning that peer will not be accessible outside of its organization.

4. Will only push to sub branch, as we are not 100% confirmed if anything here would break or change when next step is carried out.

#############################################################################

Register and enrolling orderer identity.

1. Start the org CA

   `cd fabric-ca-server-org1`

   `./fabric-ca-server start`

2. Open a new terminal

   `cd Fabric-prod-network/fabric-ca-client`

   `export FABRIC_CA_CLIENT_HOME=$PWD`

   `./fabric-ca-client register -d --id.name orderer0 --id.secret orderer0pw -u https://kky-dell:7055 --mspdir ./org1-ca/rcaadmin/msp --id.type orderer --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

   `./fabric-ca-client enroll -u https://orderer0:orderer0pw@kky-dell:7055 --mspdir ./ordererOrg1.example.com/orderer0/msp --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'kky-dell,*localhost'`

3. create _config.yaml_ file inside _ordererOrg1.example.com/orderer0/msp_ folder with values of:

   ```
   NodeOUs:
       Enable: true
       ClientOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: client
       PeerOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: peer
       AdminOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: admin
       OrdererOUIdentifier:
           Certificate: cacerts/kky-dell-7055.pem
           OrganizationalUnitIdentifier: orderer
   ```

4. i) rename ordererOrg1.example.com/orderer0/msp/signcerts/cert.pem to orderer0-cert.pem

   ii) rename file inside ordererOrg1.example.com/orderer0/msp/keystore to orderer0-key.pem

5. Create a tlscacerts folder and copy the tls-root-cert into this folder.

   `mkdir ordererOrg1.example.com/orderer0/msp/tlscacerts`

   `cp tls-root-cert/tls-ca-cert.pem ordererOrg1.example.com/orderer0/msp/tlscacerts`

6. Create orgMsp and localMsp folder.

   `mkdir -p ordererOrg1.example.com/orderer0/orgMsp/msp ordererOrg1.example.com/orderer0/localMsp/msp`

7. Prepare the orgMsp files

   `cp ordererOrg1.example.com/orderer0/msp/config.yaml ordererOrg1.example.com/orderer0/orgMsp/msp && cp -R ordererOrg1.example.com/orderer0/msp/cacerts ordererOrg1.example.com/orderer0/orgMsp/msp && cp -R ordererOrg1.example.com/orderer0/msp/tlscacerts ordererOrg1.example.com/orderer0/orgMsp/msp`

8. Prepare the localMSP files

   `cp ordererOrg1.example.com/orderer0/msp/config.yaml ordererOrg1.example.com/orderer0/localMsp/msp && cp -R ordererOrg1.example.com/orderer0/msp/cacerts ordererOrg1.example.com/orderer0/localMsp/msp && cp -R ordererOrg1.example.com/orderer0/msp/tlscacerts ordererOrg1.example.com/orderer0/localMsp/msp && cp -R ordererOrg1.example.com/orderer0/msp/keystore ordererOrg1.example.com/orderer0/localMsp/msp && cp -R ordererOrg1.example.com/orderer0/msp/signcerts ordererOrg1.example.com/orderer0/localMsp/msp`

9. Add the location of the orderer binary to PATH environment variable

   `cd Fabric-prod-network`

   `export PATH=$PWD/fabric/bin:$PATH`

10. Create the following folder structure

    ```
    ├── organizations
      └── ordererOrganizations
            └── ordererOrg1.example.com
               ├── msp
                  ├── cacerts
                  └── tlscacerts
               ├── orderers
                  └── orderer0.ordererOrg1.example.com
                        ├── msp
                        └── tls
    ```

    `mkdir -p organizations/ordererOrganizations/ordererOrg1.example.com/msp/cacerts organizations/ordererOrganizations/ordererOrg1.example.com/msp/tlscacerts organizations/ordererOrganizations/ordererOrg1.example.com/orderers/orderer0.ordererOrg1.example.com/msp organizations/ordererOrganizations/ordererOrg1.example.com/orderers/orderer0.ordererOrg1.example.com/tls`

11. Prepare i)TLS certificates, ii) orderer local MSP files, and iii) orderer org MSP files.

    `cp -R fabric-ca-client/ordererOrg1.example.com/orderer0/localMsp/msp/tlscacerts/tls-ca-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/orderer0.ordererOrg1.example.com/tls/tls-cert.pem`

    `cp -R fabric-ca-client/ordererOrg1.example.com/orderer0/localMsp/msp/signcerts/orderer0-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/orderer0.ordererOrg1.example.com/tls/orderer0-cert.pem`

    `cp -R fabric-ca-client/ordererOrg1.example.com/orderer0/localMsp/msp/keystore/orderer0-key.pem organizations/ordererOrganizations/ordererOrg1.example.com/orderers/orderer0.ordererOrg1.example.com/tls/orderer0-key.pem`

    `cp -R fabric-ca-client/ordererOrg1.example.com/orderer0/localMsp/msp organizations/ordererOrganizations/ordererOrg1.example.com/orderers/orderer0.ordererOrg1.example.com`

    `cp -R fabric-ca-client/ordererOrg1.example.com/orderer0/orgMsp/msp/cacerts/kky-dell-7055.pem organizations/ordererOrganizations/ordererOrg1.example.com/msp/cacerts`

    `cp -R fabric-ca-client/ordererOrg1.example.com/orderer0/orgMsp/msp/tlscacerts/tls-ca-cert.pem organizations/ordererOrganizations/ordererOrg1.example.com/msp/tlscacerts/tls-cert.pem`

    `cp -R fabric-ca-client/ordererOrg1.example.com/orderer0/orgMsp/msp/config.yaml organizations/ordererOrganizations/ordererOrg1.example.com/msp`

NOTES UP TO THIS POINT:

1.  Things are looking okay and similar to files returned when registering and enrolling identities for the peer.

#############################################################################

Create the ordering service genesis block

1. Set environment variable:

   `cd Fabric-prod-network`

   `export PATH=$PWD/fabric/bin:$PATH`

   `export FABRIC_CFG_PATH=$PWD/fabric/config`

   `configtxgen --help`

2. Edit _configtx.yaml_ in _fabric/config_ to configure Organizations section with the following values:

   ```
   Organizations:

    - &OrdererOrg
        Name: OrdererOrg
        SkipAsForeign: false
        ID: OrdererMSP
        MSPDir: ../../organizations/ordererOrganizations/ordererOrg1.example.com/msp
        Policies: &OrdererOrgPolicies
            Readers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('OrdererMSP.admin')"
        OrdererEndpoints:
            - "127.0.0.1:7050"
   ```

3. Edit _configtx.yaml_ in _fabric/config_ to configure Orderer section with the following values:

   ```
   Orderer: &OrdererDefaults
    OrdererType: etcdraft
    Addresses:
        # - 127.0.0.1:7050
    BatchTimeout: 2s
    BatchSize:
        MaxMessageCount: 500
        AbsoluteMaxBytes: 10 MB
        PreferredMaxBytes: 2 MB
    MaxChannels: 0
    EtcdRaft:
        Consenters:
            - Host: orderer0.ordererOrg1.example.com
              Port: 7050
              ClientTLSCert: ../../organizations/ordererOrganizations/ordererOrg1.example.com/orderers/orderer0.ordererOrg1.example.com/tls/tls-cert.pem
              ServerTLSCert: ../../organizations/ordererOrganizations/ordererOrg1.example.com/orderers/orderer0.ordererOrg1.example.com/tls/tls-cert.pem
              # Note: If the above ClientTLSCert & ServerTLSCert does not work, would have to replace tls-cert.pem with orderer0-cert.pem instead.
            # Raft 1 and Raft 2 commented out for now as the orderer identity has not been registered and enrolled yet.
            # - Host: raft1.example.com
            #   Port: 7050
            #   ClientTLSCert: path/to/ClientTLSCert1
            #   ServerTLSCert: path/to/ServerTLSCert1
            # - Host: raft2.example.com
            #   Port: 7050
            #   ClientTLSCert: path/to/ClientTLSCert2
            #   ServerTLSCert: path/to/ServerTLSCert2
        Options:
            TickInterval: 500ms
            ElectionTick: 10
            HeartbeatTick: 1
            MaxInflightBlocks: 5
            SnapshotIntervalSize: 16 MB
    Organizations:
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        BlockValidation:
            Type: ImplicitMeta
            Rule: "ANY Writers"
    Capabilities:
        <<: *OrdererCapabilities
   ```

4. Edit _configtx.yaml_ in _fabric/config_ to configure Profiles section with the following values:

   ```
   Profiles:

    EtcdRaftOrdererGenesis:
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            OrdererType: etcdraft
            Organizations:
                - <<: *OrdererOrg
                  Policies:
                      <<: *OrdererOrgPolicies
                      Admins:
                          Type: Signature
                          Rule: "OR('OrdererMSP.member')"
        Application:
            <<: *ApplicationDefaults
            Organizations:
        Consortiums:
            SampleConsortium:
                Organizations:
   ```

5. Generate the system genesis block:

   `configtxgen -profile EtcdRaftOrdererGenesis -channelID system-channel-testrun-1 -outputBlock ./system-genesis-block/genesis.block`

NOTES UP TO THIS POINT:

1. Block generated, but might need to add participating orgs in consortium, but in order to do that:
   i) have to get all necessary files as the ordere has for the org1, especially orgMsp part.

2. Once all these are done, delete the current test genesis block and create a new genesis block again. Maybe can try starting the orderer to see if it works in the first place or not, but that would require:
   1. some configuring of the orderer.yaml file
   2. folder creation for orderer ledger storage
   3. WALDir configuration

#############################################################################

Deploy the ordering service

1.

Upcoming to do:

1. Deploy the ordering service
2. Documenting the instruction steps.

#############################################################################

Configuration of orderer.yaml

1. Edit _orderer.yaml_ in _fabric/config_ with the respective values:

   ```
   General.TLS.Enabled: true

   General.TLS.PrivateKey: ../../organizations/ordererOrganizations/ordererOrg1.example.com/orderers/orderer0.ordererOrg1.example.com/tls/orderer0-key.pem

   General.TLS.Certificate: ../../organizations/ordererOrganizations/ordererOrg1.example.com/orderers/orderer0.ordererOrg1.example.com/tls/orderer0-cert.pem

   General.TLS.RootCAs: ../../organizations/ordererOrganizations/ordererOrg1.example.com/orderers/orderer0.ordererOrg1.example.com/tls/tls-cert.pem

   General.LocalMSPDir: ../../organizations/ordererOrganizations/ordererOrg1.example.com/orderers/orderer0.ordererOrg1.example.com/msp
   ```
