# Fabric-CA

There are 2 tools fabric provides for managing Certificates -OpenSSL,Fabric-ca,cryptogen

Lets run through fabric-ca
http://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#overview

Fabric-CA is an docker image that has 2 items -FABRIC-CA-SERVER & FABRIC-CA-CLIENT
server comamnds -Init,Start,Verison
Client -Register,Enroll,Revoke,CRL,CSR 

Designing a CA
First thing we need to check is if fabric ca should be in a cluster -If yes then select load balancer such has HA Server which manages various fabric ca server
next decide if these should connec to a db like mysql or LDAP
Next decide how certifcates will be stored -In a PEM file or HSM(Hardware)
Next decide what ECDSA with hashing we will use and curve -Fabric supports 3 variants(SHA235,SHA 384,SHA512 with curves as prime256,secp384,secp521v1)

How all this works -
Both server and client has config files that can be confiured to describe CSR parameters like EXPIRY, ca locaiton.name and hash functions to use
First bring up fabric ca server using init and start- Here you can specify how CSR(Certificate signing requests).
Now use fabric ca client to register users and enroll them
each enrollment gives a ECERT,Private key which can be used later for siging transactions

The enroll command stores an enrollment certificate (ECert), corresponding private key and CA certificate chain PEM files in the subdirectories of the Fabric CA client’s msp directory. You will see messages indicating where the PEM files are stored.

http://hyperledger-fabric-ca.readthedocs.io/en/latest/servercli.html


http://hyperledger-fabric-ca.readthedocs.io/en/latest/clientcli.html

REGISTERING A USER-
export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
fabric-ca-client register --id.name admin2 --id.affiliation org1.department1 --id.attrs 'hf.Revoker=true,admin=true:ecert'

The password, also known as the enrollment secret, is printed. This password is required to enroll the identity. This allows an administrator to register an identity and give the enrollment ID and the secret to someone else to enroll the identity.

Enrolling a peer identity--
Now that you have successfully registered a peer identity, you may now enroll the peer given the enrollment ID and secret (i.e. the password from the previous section). This is similar to enrolling the bootstrap identity except that we also demonstrate how to use the “-M” option to populate the Hyperledger Fabric MSP (Membership Service Provider) directory structure.

The following command enrolls peer1. Be sure to replace the value of the “-M” option with the path to your peer’s MSP directory which is the ‘mspConfigPath’ setting in the peer’s core.yaml file. You may also set the FABRIC_CA_CLIENT_HOME to the home directory of your peer.

export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer1
fabric-ca-client enroll -u http://peer1:peer1pw@localhost:7054 -M $FABRIC_CA_CLIENT_HOME/msp
Enrolling an orderer is the same, except the path to the MSP directory is the ‘LocalMSPDir’ setting in your orderer’s orderer.yaml file.

All enrollment certificates issued by the fabric-ca-server have organizational units (or “OUs” for short) as follows:

The root of the OU hierarchy equals the identity type
An OU is added for each component of the identity’s affiliation
For example, if an identity is of type peer and its affiliation is department1.team1, the identity’s OU hierarchy (from leaf to root) is OU=team1, OU=department1, OU=peer.


http://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#attribute-based-access-control
