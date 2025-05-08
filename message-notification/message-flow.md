# Message Notification Flow

```mermaid
flowchart TD
    A[Google Pub/Sub] -->|Subscribe| B[Pod Server]
    B -->|Receive Message| C[Parse & Validate Data]
    C -->|Validated| D{Check Message Type}
    D -->|Different Categories| E[Find User IDs from Server Cache]
    E -->|User IDs Collected| F{Check Server Cache}
    F -->|Cache Hit| H{Is User Online?}
    F -->|Cache Miss| G[Get Message from Redis]
    G -->|Message Retrieved| H
    H -->|Yes| I[Push via WebSocket]
    H -->|No| J{Check isOnlineOnly}
    J -->|isOnlineOnly=false| K[Batch Pool]
    K -->|Async Write| L[(Redis List Storage)]
    J -->|isOnlineOnly=true| M[Discard Message]
    
    style A fill:#E8F5E9
    style B fill:#FFF3E0
    style C fill:#FFF3E0
    style D fill:#FCE4EC
    style E fill:#E3F2FD
    style F fill:#FCE4EC
    style G fill:#FFF3E0
    style H fill:#FCE4EC
    style I fill:#E8F5E9
    style J fill:#FCE4EC
    style K fill:#FFF3E0
    style L fill:#E3F2FD
    style M fill:#FFF3E0

classDef decision fill:#FCE4EC,stroke:#333,stroke-width:2px;
classDef process fill:#FFF3E0,stroke:#333,stroke-width:2px;
classDef storage fill:#E3F2FD,stroke:#333,stroke-width:2px;
classDef io fill:#E8F5E9,stroke:#333,stroke-width:2px;

```

## Flow Description

1. **Message Receipt**
   - Data originates from Google Pub/Sub
   - Pod server creates a single subscription to receive messages

2. **Message Processing**
   - Received messages are parsed and validated
   - Message type is checked to determine the category

3. **User Identification**
   - System finds user IDs from server cache based on message category
   - Different categories may require different user ID lookup processes

4. **Content Retrieval**
   - System first checks server cache for message content
   - If cache miss occurs, content is fetched from Redis
   - Retrieved content is used for message delivery

5. **Message Delivery**
   - Online users receive messages via WebSocket
   - For offline users:
     - If isOnlineOnly=false: Messages are stored in Redis List
     - If isOnlineOnly=true: Messages are discarded

6. **Offline Storage**
   - Messages for offline users are processed through a batch pool
   - Batch pool performs asynchronous writes to Redis
   - Fixed batch size or time interval determines write frequency
