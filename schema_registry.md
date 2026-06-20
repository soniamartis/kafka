# Schema Registry

Confluent schema registry is open-source
It helps in validating schemas of messages that are being sent
Enables to reduce the size of payloads/messages(as we do no need to send the schema with the message)
Helps in schema evolution
Highly recommended component but not mandatory to use
We can have a separate process/jvm for schema registry and can be kept out of the current kafka cluster

Supported data formats:
Avro
Json
Protocol Buffers

If we do not use schema registry , then we may end up storing garbage in kafka for long term
The consumers will start consuming bad data and may fail for those bad messages
Therefore we need schema validation
