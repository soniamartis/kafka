Article:
https://www.confluent.io/blog/transactions-apache-kafka/


Earlier kafka provided the following delivery guarantee:
Atleast once, in order delivery per partition
As kafka is built on commodity hardware, there is a chance that something fails , leading to duplicate messages on topics

Basic flow while sending:
Producer sends the message to the leader of the partition
Leader writes it to the log
Leader sends an ack back to the producer

Suppose this scenario:
Producer sends message to leader
leader appends it to log
Some failure occurs in the cluster and the leader could no send an ack back to producer(maybe the isr condition was not satisfied due to some partition failure etc)

Now, the producer will retry and send the message again
The leader will blindly append it to the log
Producer retries can lead to duplicate messages

Why we need exactly once?
Typical transfers app which uses read-process-write mechanism
Suppose Alice transfers $10 to Bob, the flow will be like this:
Fund transfer will read the transfer message($10, Alice, Bob) from transfer topic
It will write 2 messages to the balance-update topic (alice,debit($10)) and (bob,credit($10)) to same or different partitions of balance update
It will add the consumer offset  of the transfer message to the internally managed offsets-topic

![Screenshot 2022-09-16 at 9 11 32 AM](https://user-images.githubusercontent.com/12456295/209444268-deb8f1ff-7336-4c0d-9d04-f23f3baf7032.png)


What can go wrong?
Crash occurs while committing the transfer message offset to the offsets topic
In this case, when we restart the fund transfer process, it will again re-process the transfer message and create another debit-credit entry pairs for Alice and Bob

And many other scenarios....

Whats new?
Exactly once in-order delivery per partition
Atomic writes across multiple partitions
Performance considerations

---
The idempotent producer - Exactly once delivery per partition
---
There is a sequence number and a Producer ID(pid)
Sequence number starts with 0
For every successful acknowledgement from the leader, the sequence number is incremented
The pid is assigned by the broker, every new producer will get assigned a new pid

Suppose this is the very first message sent by the producer

The leader appends it to the log

Leader sends an ack to the producer

As producer got a successful ack, it will bump the sequence number by 1 and will send the next message

The next message gets written to the log, but now something failed while sending ack

Since the producer did not get an ack, it is going to send the same message again with the same sequence number.
A validation will run at the broker which will check that it just appended a message with seq no 1 and was expecting a seq no 2, so it just return an ack to the producer but will not append the message



Sequence numbers and producer ids:
enable de-dupe
are in the log

Hence de-dupe works transparently across leader changes
Will not de-dup application level resends as every send has its own sequence number
Works transparently - no API changes

---
Multi-partition atomic writes
---
Consumer - process -produce
Introducing transactions:

```
producer.initTransactions();
try{
 producer.beginTransaction();
 producer.send(record0);
 producer.send(record1);
 producer.sendOffsetsToTxn();
 producer.commitTransaction();
}catch(ProducerFencedException e){
  producer.close();
}catch(KafkaException e){
  producer.abortTransaction();
}
```

Whats new in transactions:
Transaction co-ordinator and the txn log
The Txn co-ordinator is a module running inside every kafka broker
Txn log is an internal kafka topic
Each coordinator owns some subset of the partitions in the transaction log, ie. the partitions for which its broker is the leader.
Every transactional.id is mapped to a specific partition of the transaction log through a simple hashing function. This means that exactly one coordinator owns a given transactional.id.
The transaction log just stores the latest state of a transaction and not the actual messages in the transaction. The messages are stored solely in the actual topic-partitions. 

The transaction could be in various states like “Ongoing,” “Prepare commit,” and “Completed.” It is this state and associated metadata that is stored in the transaction log.


producer.initTransactions()
This is called exactly once per producer
It registers the transactional.id with the txn co-ordinator and the co-ordinator bumps the epoch for that txn id, to ensure that only this producer can be active in your system at that moment
A txn id identifies a producer across any instance of the producer

Another thing that this method does is, if there was an open txn for that txn.id, it will either be rolled forward(complete transaction) or rolled back
This could happen if the producer failed mid txn and restarted



producer.beginTransaction()
Marks the beginning of transactional writes
producer.send() *N
A log will be written to the txn log for every send() that writes to a unique partition within the topic
Example producer.send(r1) may write to topic partition 0, producer.send(r2) will write to topic partition 1, producer.send(r3) may write to topic partition 1 again, in this case , there will be two entries in the txn log, once for partition 0 and other for partition 1



producer.commitTransaction()
its a two-phase commit
Phase 1
Write a prepare commit marker to the txn log
If something goes wrong after this phase 1 commit step, you can be rest assured, that when you restart the producer again, the txn will be rolled forward

Phase 2 commit:
Write commit markers to the partitions which you have written the transactional messages to.
Suppose your topic has N partitions, and you wrote to parts 0 and 1 as part of the txn, the txn co-ordinator will write commit markers to the parts 0 and 1 as it had a record of those partitions from the producer.send() stage
These markers play an important role on the consumer side to read only the committed messages at the consumer
We do not have to handle anything at the application level for these markers, they are handled by the kafka client logic only



A commit message is written to the txn log

Co-ordinator returns success to the producer, which can then start another txn


Transactions at the consumer side:
Consumers can be configured to be either txn aware or txn not aware
Read uncommitted reads all messages except commit messages. Even open/aborted messages will be returned in uncommitted mode
Read committed :  only committed transactional or non-transactional messages will be returned to consumer



---
doubts related to spring kafka
---
behaviour of kafka txn at producer only
behaviour of kafka txn at consume-process-produce
