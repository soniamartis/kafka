# Steps to install kafka in mac:

- download the latest binary from Kafka downloads
- extract the folder
- cd into the directory and type in below commnds from terminal

```unix


generate randomuuid:
bin/kafka-storage.sh random-uuid

update server.properties with this random id:
bin/kafka-storage.sh format -t G9cjlMY4RFSxPoiVN9_6oA -c config/server.properties --standalone


start kafka server:
bin/kafka-server-start.sh config/server.properties
```

In my current setup:
- kafka installation directory: Downloads/kafka_2.13-4.0.0/
