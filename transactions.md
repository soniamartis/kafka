## Transactions in Kafka

- Producer side
- Consumer side


### Chaining of Kafka + DB transactions:
- The listener container starts the Kafka transaction and the @Transactional annotation starts the DB transaction. 
- The DB transaction is committed first; if the Kafka transaction fails to commit, the record will be redelivered so the DB update should be idempotent.

