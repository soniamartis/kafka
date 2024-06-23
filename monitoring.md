# Kafka Monitoring
- System Health
  - Are all brokers and topics available?
  - How much data is being processed?
  - What can be tuned to improve performance?
- E2E SLA monitoring
    - Does Kafka process all messages in < 15 secs
    - Are there duplicate events
    - Is 8 am report missing data
 
## Important metrics
- Number of active controllers: should always be 1
- Number of under-replicated partitions(URP): should always be 0 (if a producer is producing at a very high rate and the replication is not fast enough, we will see a lag)
- Number of offline partitions: should always be 0(if > 0, it means the topic is partially down)

NOTE: Set alerts on metrics, so that we get emails when something is not working as expected and we can remediate immediately

https://docs.confluent.io/platform/current/kafka/monitoring.html

## Broker metrics
- ActiveControllerCount: 1
- OfflinePartitionsCount
- UncleanLeaderElectionsPerSec : 0



  
