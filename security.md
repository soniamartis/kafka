# Kafka Security

Adding security to a system comes with a performance cost, notably the cpu overhead of encrypting data

## What places of kafka need to secured:
- broker authenticates the client to make sure the message is coming from the configured producer
- producer should have an authenticated connection with broker
- encryption of data in transit (over the network)
- encryption of data at rest on datastores (brokers)
- broker needs to verify that producer has the permission to write to topic
- same applies to consumer, does it have permissions to read from topic

 ## Authentication basics

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

