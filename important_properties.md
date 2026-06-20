# Important Properties

### min.insync.replicas
- The min number of replicas that must be in-sync for a partition of the topic to be considered available

- <img width="1181" height="698" alt="image" src="https://github.com/user-attachments/assets/ce71678d-5729-4900-b65d-3e044badb5a7" />

- in above diagram, the property is set to 2
- If  the data is in sync on atleast 2 brokers for a partition, then things are good
- If data is in sync on less than the specified number of replicas, then kafka will prevent the producer from producing data on that partition, the consumers can continue reading data from the partitions
- This property if not set correctly, means that your producer might not be able to send the messages to the partition at all
- Min-in0sync replicas = N - X
  - N : replication factor
  - X : The maximum number of broker failures you want the topic to tolerate for writes.
- This property is used to determine the number of replicas that must ack the message when acks = all
