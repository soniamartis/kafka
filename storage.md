# Kafka Topics Storage

### Storage layout for a partition in broker
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
  
- the index file doesnt store all offsets and their message's byte offset, even if the log file contains messages from offsets 0 to N
- It stores only a few offsets based on the property: `log.index.interval.bytes`  ie. it adds an entry in the index file after every 4096 bytes that are written to the .log file
- If this value is decreased, we will have more entries, which could make search more faster, if this value is increased, we will have lesser entries, that will slow down the search
- Each entry in the index file is 8 bytes in size: 4 bytes for the relative offset from base offset and 4 bytes for the byte offset of the record in .log file 
