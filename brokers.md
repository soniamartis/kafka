## Inside the Kafka Brokers

https://developer.confluent.io/courses/architecture/broker/

- Kafka cluster comprises of
  - Control plane: handles cluster metadata
  - Data plane: handles actual data
 
- Kafka broker handles 2 types of client requests
  - Produce request: request that a batch of data is writen to a topic
  - Fetch request: request data from kafka topic
 
- Kafka Producer client request flow
  - Determine the partition the data needs to be written to
  - Batch the records before sending them out to avoid too many network requests, draining the record batch depends on size of batch(batch.size)or by time(liner.ms)
  - Once the batch is drained, it is sent to the socket receive buffer of the leader partition
  - A network thread is assigned to a client's request, will convert that event record into a produce request and write it to the kafka broker pool's request queue
  - From there, one of the I/O threads will pick the request and write to disk
  - Kafka's storage data structure is a commit log that consists of .index and .log
  - The .log file stores the actual data whereas the .index file maintains a mapping of the message offset with the actual disk location
  - Kafka broker dos not send back an ack till this data is replicated to the follower partitions, and at the same time, we do not want the I/O threads blocked till replication for this request is complete
  - So such requst is then drained to the purgatory map, till the replication completes
  - Once replication is done, the broker will take the request out of purgatory, create a response object and will put it on the response queue
  - The network thread then picks the response and sends it to the socket send buffer before processing the next response from that client
 
  ![Screenshot 2024-08-17 at 9 42 53 AM](https://github.com/user-attachments/assets/a662c67d-0d12-4dcf-9945-5b89cfd06b34)

- Kafka Consumer client request flow
  - Same as above to a very good extent
  - Consumer's request arrives on socket receive buffer
  - Network thread converts it to a fetc request and put's it in request queue
  - It is picked by one of the I/O threads that fecthes the data based on request offset by referring the .index file
  - If the offset has no data, and if the broker returns, the client will keep bombarding the broker for data at that offset
  - So we can enable a request config based on either max wait time/ response size, till this criteria is met, the request will be in purgatory map
  - Once it is fulfilled, the response will be returned
 
  ![Screenshot 2024-08-17 at 10 19 25 AM](https://github.com/user-attachments/assets/377bf1da-f3a8-46ef-8b72-7457f96fc466)
