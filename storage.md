# Kafka Topics Storage


```
├── __events-0
│   ├── 00000000000000000000.index
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.timeindex
│   ├── 00000000000000000109.index
│   ├── 00000000000000000109.log
│   ├── 00000000000000000109.snapshot
│   ├── 00000000000000000109.timeindex
```

- events-0 represents the topic-partition folder on the broker
- 00000000000000000000.log contains data sent by the producer, the name of the file indicates that it contains data from base offset 0, it is a closed segment
- 00000000000000000000.index is the corresponding index file that is used to search for the message at a given offset efficiently. it stores the relative offset value from the base offset and the byte offset of the message in the correspondingly named .log file. It doesnt store every mapping, rather it stores few mappings, we can efficiently find the message with this index file using binary search on the relaive offsets
- 00000000000000000000.timeindex stores the mapping of the timestamp vs the relative offset and is used for efficiently looking up messages by time and to determine message retention
- 00000000000000000109.log is the active log segment ie. this file is opened in read-write mode. it contains messages starting from base offset 109

### How indexing works
<img width="665" height="253" alt="image" src="https://github.com/user-attachments/assets/708229c0-d10e-4d00-9e01-469da65a0451" />
  
