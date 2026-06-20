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
- A replicated partition is assigned to additional brokers, called followers of the partition. Replication provides redundancy of messages in the partition, such that one of the followers can take o[...]
- is a broker failure. All producers must connect to the leader in order to publish messages, but consumers may fetch from either the leader or one of the followers.

## Retention

- brokers are configured with default retention settings for topics either retaining messages for certain period of time(7 days) or until a partition reaches a certain size in bytes
- Once these limits are reached, messages are expired and deleted
- Topics can be configured as log-compacted, which means kafka will retain only the last message of a specific key

## Multiple clusters

- For DR(disaster recovery)


https://www.youtube.com/watch?v=UNUz1-msbOM
Kafka is capable of moving large amount of data in short period of time
The two factors that make kafka fast are:
1. sequential access pattern for read/writes
2. zero copy write

Sequential access:
Kafka uses the append-only log as its primary data structure
An append-only log adds data to the end of the file
This access pattern is sequential
Sequential write can reach 100MB/s whereas random writes can reach only upto a few 100 KB/s
Using HDD has its cost advantage too, compared to SSD, HDD comes at 1/3rd the price and with abt 3 times the capacity
The HDD is very efficient at doing sequential reads/writes as it just has to move its arm to the next sequential location instead of a random read/write pattern
The sequential access pattern is orders of magnitudes faster than the random access pattern


Zero copy writes:
Kafka moves a lot of data from network to disk and disk to network, so it is important to ensure that this data transfer is efficient
It is critically important to eliminate excess copy when moving pages of data between the disk and the network

Modern unix operating systems are highly optimised for transferring data between disk and network w/o excess data copy

When zero copy is not used:(the data on kafka broker partion, is first loaded into the kafka broker's jvm and then copied to the NIC buffer)
Data is copied from the disc to OS cache
It is then copied from OS cache to app buffer
then copied from app buffer to socket buffer
and then finally from socket buffer to the NIC buffer
and then sent to the consumer


With zero copy:
The data is directly copied from the OS cache into the NIC buffer using the sendfile system call
On modern network card, this optimised copy is done with DMA(Direct Memory Access)
When DMA is used, the cpu is not involved, making it more efficient






## TODO

- schema registry
- DR (disaster recovery) working
- GDPR for kafka
- Kafka streams
