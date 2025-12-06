

## how Kafka components communicate

- Kafka uses a custom **binary, request/response TCP protocol**. Clients open TCP connections to brokers and send framed request messages; the broker replies with framed responses. The protocol is _versioned_ so each API (Produce, Fetch, ApiVersions, DescribeTopicPartitions, etc.) can evolve
    
- There are two important families of bytes you must handle correctly: the _transport framing_ (message size, correlation id, etc.) and the _API-specific payloads_ (fields for that request/response)



## Overview of a Kafka Producer ↔ Broker communication cycle

### Sequence:

1. **TCP Connection Established**
    
2. **ApiVersions Request / Response**
    
3. **Metadata Request / Response**
    
4. **Produce Request / Response**
    
5. **Acknowledgment (acks = 1 or all)**
    
6. (Optional) **Heartbeat / Fetch / Close**





Kafka Project implemented : [/home/abdallah-selim/Desktop/Work/Projects/codecrafters/codecrafters-kafka-go]