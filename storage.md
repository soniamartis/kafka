# Kafka Topics Storage

---
### Storage layout for a partition in broker
- Topic contains N partiitions and a partition is made up of X segments
- Only 1 segment is an active segment ie. it is open in read-write mode. Messages written by producer are appended to the active segment
- Once the condition occurs to roll the segment, it is closed, opened in read-only mode and a new segment is created that is opened in read-write mode
- Closed segments are read-only and are used by consumers for reading old messages
- 
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

---
### How indexing works

<img width="665" height="253" alt="image" src="https://github.com/user-attachments/assets/708229c0-d10e-4d00-9e01-469da65a0451" />

- the index file doesnt store all offsets and their message's byte offset, even if the log file contains messages from offsets 0 to N
- It stores only a few offsets based on the property: `log.index.interval.bytes`  ie. it adds an entry in the index file after every 4096 bytes that are written to the .log file
- If this value is decreased, we will have more entries, which could make search more faster, if this value is increased, we will have lesser entries, that will slow down the search
- Each entry in the index file is 8 bytes in size: 4 bytes for the relative offset from base offset and 4 bytes for the byte offset of the record in .log file
- Kafka stores relative from base offset values instead of entire offset value to reduce file size. suppose the base offset is 1000000101, we dont need to store 1000000101, 1000000102 etc. it is sufficient to store 101, 102

---
 ### How time indexing works

 <img width="863" height="415" alt="image" src="https://github.com/user-attachments/assets/7afc7f3d-27af-4eb6-bacf-8b5d3e32e5bb" />

- Kafka supports searching messages by time, this is where .timeindex comes into the picture
- The .timeindex file contains a mapping of timestamp to the relative offset from base offset
- .index serves as an indirection from timestamp to offset to address in the log file
- .timeindex is also used to determine the retention of the messages within the segment
- Each entry is 12 bytes in size, 8 bytes for timestamp and 4 bytes for offset

---
### Log Segment Rolling
- A segment is rolled based on the 2 conditions occuring in time and size, whichever happens first:
  - `log.segment.bytes`: the file size exceeds this value, default is 1 GB
  - `log.roll.hours` or `log.roll.ms` : when the creation time for first record in the segment has elapsed this interval
  - This is not to be confused with log retention, this is simply log rolling
  - There is another condition, if the .index or .timeindex becomes full, the segment is rolled too
 
---

### Log Retention
- A segment can be deleted only when it is closed, an active segment is not deleted, even if it meets the criteria for deletion
- 
