## Transactions in Kafka

- Producer side
- Consumer side


### Chaining of Kafka + DB transactions:
- The listener container starts the Kafka transaction and the @Transactional annotation starts the DB transaction. 
- The DB transaction is committed first; if the Kafka transaction fails to commit, the record will be redelivered so the DB update should be idempotent.

https://community.ibm.com/community/user/integration/blogs/kim-clark1/2024/06/19/kafka-transactionalid#:~:text=The%20transactional.id%20doesn't,is%20in%20fact%20still%20running.

