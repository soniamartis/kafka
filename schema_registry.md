# Schema Registry

Confluent Schema Registry is an open-source component that provides a central repository for message schemas used by producers and consumers.

## Overview

The Schema Registry stores a versioned history of schemas and allows applications to ensure that data produced to Kafka topics conforms to expected schemas. It decouples schema management from Kafka brokers and supports schema evolution and compatibility checks.

## Benefits

- Validates message schemas before they are produced or consumed.
- Reduces message payload size because the schema does not need to be sent with every message.
- Enables schema evolution (versioning and compatibility checks).
- Helps prevent consumers from receiving and processing invalid or unexpected data.

## Deployment

- Schema Registry can run as a separate process/JVM and be hosted outside the Kafka cluster.
- It is typically deployed independently so schema management is isolated from Kafka broker operations.

## Supported Data Formats

- Avro
- JSON
- Protocol Buffers

## Why Use a Schema Registry

If you do not use a schema registry, you risk:

- Storing garbage or malformed data in Kafka long-term.
- Downstream consumers failing when they encounter unexpected or invalid messages.

Schema validation helps maintain data quality and reliability across producers and consumers.

## Recommendations

- Schema Registry is highly recommended but not strictly mandatory.
- Configure compatibility rules (e.g., backward/forward/full) according to your evolution strategy to avoid breaking consumers when schemas change.
