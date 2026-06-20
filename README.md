# High level overview

Distributed commit log or distributed streaming platform
Reduces the complexity of the systems from O(n^2) to O(n)

## Messages

- Unit of data within kafka
- Simply an array of bytes from kafka's perspective
- Can have optional metadata called key, and is also an array of bytes

## Batches

- Collection of messages written to kafka into the same topic and the same partition
- The larger the size of the batch, more messages are processed per unit time, but longer it takes for each message to propagate
- An individual round trip around the network would be an excessive overhead

## Schemas

- Imposing a schema allows the producer and consumer to be decoupled
- Using well defined schemas and storing them in a common repo reduces the co-ordination between the producer and consumer

## Topics and partitions

- Partition is a single log
- Messages are written to it in an append only fashion and are read in order from beginning to end
- There is no guarantee of message ordering across the topic, only at a single partition level
- Each partition can be hosted on a different server, which means that a topic can be scaled horizontally across servers
- Partitions can be replicated, such that if a server fails, a copy will be provided from another server

## Producers and consumers

### Producers

- Producers aka publisher and writers
- produce new messages
- responsible for distributing the messages across the partitions in a topic

### Consumers

- aka subscribers or readers
- Reads messages
- Keeps track of which messages it has already consumed by keeping track of offset of messages
- Offset is an int value that continually increases
- By storing  the next possible offset for each partition, a consumer can stop and restart without losing its place

### Consumer group

- Consumers work as part of the consumer group which is one or more consumers that work together to consume a topic
- The group ensures that there is only one consumer that consumes a partition within the group
- Mapping of consumer to partition is called ownership

## Brokers and clusters

- A single kafka server is called a broker
- Receives messages from the producer, assigns offset to them and writes them to disk
- A single broker can easily handle thousands of partitions and millions of messages per second
- Brokers operate as part of the cluster
- Within a cluster of brokers, one broker will function as the cluster conrtoller
- It is responsible for admin operations like assigning partitions to brokers and monitoring for broker failures
- A part is owned by single broker in the cluster called as leader of the part
- A replicated partition is assigned to additional brokers, called followers of the partition. Replication provides redundancy of messages in the partition, such that one of the followers can take over in case of a broker failure. All producers must connect to the leader in order to publish messages, but consumers may fetch from either the leader or one of the followers.

## Retention

- brokers are configured with default retention settings for topics either retaining messages for certain period of time(7 days) or until a partition reaches a certain size in bytes
- Once these limits are reached, messages are expired and deleted
- Topics can be configured as log-compacted, which means kafka will retain only the last message of a specific key

## Multiple clusters

- For DR(disaster recovery)

## Why Kafka is Fast

Kafka is capable of moving large amounts of data in a short period of time. Two factors make Kafka fast:

1. **Sequential access pattern for read/writes**
2. **Zero copy write**

### Sequential Access

Kafka uses the append-only log as its primary data structure. An append-only log adds data to the end of the file, which creates a sequential access pattern.

- Sequential writes can reach **100MB/s** whereas random writes can reach only up to a few **100 KB/s**
- HDDs have a cost advantage over SSDs—HDDs cost 1/3rd the price with 3x the capacity
- HDDs are very efficient at sequential reads/writes—the arm only needs to move to the next sequential location instead of performing random read/write operations
- Sequential access patterns are orders of magnitude faster than random access patterns

### Zero Copy Writes

Kafka moves a lot of data from network to disk and disk to network, so it's important to ensure that this data transfer is efficient. It's critically important to eliminate excess copies when moving pages of data between the disk and the network.

Modern Unix operating systems are highly optimized for transferring data between disk and network without excess data copying.

#### Without Zero Copy (traditional approach):

Data on a Kafka broker partition goes through multiple copies:
1. Data is copied from disk to OS cache
2. Copied from OS cache to app buffer
3. Copied from app buffer to socket buffer
4. Finally copied from socket buffer to the NIC buffer
5. Sent to the consumer

#### With Zero Copy:

- Data is directly copied from the OS cache into the NIC buffer using the `sendfile` system call
- On modern network cards, this optimized copy is done with **DMA (Direct Memory Access)**
- When DMA is used, the CPU is not involved, making the transfer more efficient

**Reference**: [How Apache Kafka Works on YouTube](https://www.youtube.com/watch?v=UNUz1-msbOM)

## TODO

- schema registry
- DR (disaster recovery) working
- GDPR for kafka
- Kafka streams
