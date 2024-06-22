# Kafka Security

- Adding security to a system comes with a performance cost, notably the cpu overhead of encrypting data
- Authn
- Authz
- Encryption

## What places of kafka need to secured:
- broker authenticates the client to make sure the message is coming from the configured producer
- producer should have an authenticated connection with broker
- encryption of data in transit (over the network)
- encryption of data at rest on datastores (brokers)
- broker needs to verify that producer has the permission to write to topic
- same applies to consumer, does it have permissions to read from topic

 ## Authentication
 - Authentication identifies clients to brokers, brokers to brokers and brokers to zookeeper

 ### KafkaPrincipal
 - Maintains the data for authenticating the client with kafka
 - Ex. if we are using username-password based authentication, this kafkaprincipal will respresent the username
 - This KafkaPrincipal is then further used for authorization too to determine access control mechanims
 - All requests are associated with a principal, if authentication has not been enabled, then the principal associated with the connection is anonymous
 - We can enable authentication to 2 types of interactions:
   -  client to brokers
   -  broker to brokers
  
###  Listeners and Security Protocols
- The brokers are configured to listen on a host/port along with security protocol needed for authentication like SSL or SASL_SSL
- A broker can be configured with more than 1 listeners with different combinations of host:port and security protocols

- Sample broker listener configuration

```
listeners=EXTERNAL://:9092,INTERNAL://10.0.0.2:9093,BROKER://10.0.0.2:9094
advertised.listeners=EXTERNAL://broker1.example.com:9092,INTERNAL:// broker1.local:9093,BROKER://broker1.local:9094
listener.security.protocol.map=EXTERNAL:SASL_SSL,INTERNAL:SSL,BROKER:SSL 
inter.broker.listener.name=BROKER
```
- Sample client configuration
```
security.protocol=SASL_SSL
bootstrap.servers=broker1.example.com:9092,broker2.example.com:9092
```  
- Security protocols supported by Kafka
  - PLAINTEXT (not secure)
  - SASL_PLAINTEXT (not secure)
  - SSL (secure)
  - SASL_SSL (secure)
 
### Kafka Authentication with SSL and SASL_SSL

#### SSL
- When SSL is enabled for a kafka listener, all traffic on that channel will be encrypted with TLS
- When client opens connection with broker, it verfies the broker certificate to confirm broker identity
- the broker may also need to authenticate the client, so it will need client's certificate, This is called mTLS. (if all u want to d in encrypt and not check client certs, then set ssl.client.auth=none on broker)
- Since this mechanims uses certificates, we will have to generate certificates and periodically update them, before they expire to avoid TLS handshake failures
- Similar mechanism applies even to broker-to-broker authentication

![image](https://github.com/soniamartis/kafka/assets/12456295/fd7bf4d7-07d6-44cd-8dde-7e38cdba8c15)

#### SASL-SSL
- Uses TLS encryption but differs in authentication mechanism
- Uses one of the 4 mechanisms:
   - GSSAPI
   - PLAIN
   - SCRAM-SHA
   - OAUTHBEARER
 
 ![image](https://github.com/soniamartis/kafka/assets/12456295/25bdb973-f6db-4c8e-8b41-16f22f3440f6)

 ## Authorization

- Determines what an entity is permitted to do on the system once the entity has been authenticated
- In kafka, we configure which client has permissions to which resurces and carry out actions using ACLs(access control lists )
- KafkaPrincipal contains user info and acl then configures whether that user has permission to perform a certain action on a resource
- KafkaPrincipal: (User:Alice)
- Resource type and name: (topic:customer)
- Permission type: (allow/deny)
- Operation: (write)
- ACL : (User:Alice) has write permission on topic:customer
- When we create ACLS using kafka-acl command, they are saved in zookeeper and then cached in memory in every broker to enable fast lookups when authorizing requests
- Kafka uses a server plugin known as authorizer to apply ACLs to requests. An authorizer allows a requested action if there is atleast one 'Allow ACL' that matches the action and no explicit 'Deny ACL'

## Encryption

### Places to implement encryption
- client to broker
- broker to broker
- broker to zookeeper

## Encryption strategies
- Encrypt data in transit
- Encrypt data at rest
- E2E encryption

### Encrypt data in transit
- Out-of-the-box, kafka doesnt use encryption, but it can be enabled by using security.protocol of ssl/sasl_ssl in order to TLS encrypt data in transit
- If we just want to encrypt data in transit and do not want broker to check client certs, then set ssl.client.auth=none on brokers
- Enabling TLS impacts performance as it adds cpu overhead for data encryption/decryption

### Encrypt data at rest
- Kafka does not provide any support for encrypting data at rest
- Public cloud providers provide whole volume encryption eg: AWS EBS volumes can be encrypted with keys from AWS KMS

### E2E encryption
- Use a key mgmt service in kafka serializer/deseriliazer while sending/receiving messages, so that the broker will never see the unencrypted contents of messages
- This way we can prevent messages from being displayed in broker heap dumps and logs

![image](https://github.com/soniamartis/kafka/assets/12456295/63982b67-6092-4b4e-aadc-a3eea7842735)



## Project notes
- On cloud, we use aws provided aws_iam_msk that is used for both authn/authz
- The security protocol is SASL_SSL and the sasl_mechanism is aws_msk_iam with the IamLoginModule
- This eliminates the need to use one mechanism for authn and another for authz
- https://docs.aws.amazon.com/msk/latest/developerguide/iam-access-control.html

```
security.protocol=SASL_SSL
sasl.mechanism=AWS_MSK_IAM
sasl.jaas.config=software.amazon.msk.auth.iam.IAMLoginModule required;
sasl.client.callback.handler.class=software.amazon.msk.auth.iam.IAMClientCallbackHandler
```
- In real world aws projects that use iam to authn/authz kafka, the Principal will be the aws service like 'lambda.amazonaws.com'
- And acl will look like this:
```
{
            "Effect": "Allow",
            "Action": [
                "kafka-cluster:*Topic*",
                "kafka-cluster:WriteData",
                "kafka-cluster:ReadData"
            ],
            "Resource": [
                "arn:aws:kafka:us-east-1:0123456789012:topic/MyTestCluster/abcd1234-0123-abcd-5678-1234abcd-1/TestTopic"
            ]
        }
```



