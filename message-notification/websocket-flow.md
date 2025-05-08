# WebSocket Connection Flow

```mermaid
flowchart TD
    A[User Creates WebSocket Connection] -->|Connect| B{Verify User Identity}
    B -->|Failed| C[Break Connection]
    B -->|Passed| D[Get User Categories Info]
    D -->|Categories Retrieved| E[Store in Manager Module]
    E -->|Info Stored| F[Store User Online Status in Redis Hash]
    F -->|"Key: username:podname"| G{Check Offline Messages}
    G -->|No Messages| M[Wait for New Messages]
    G -->|Has Messages| H[Pull from Redis List]
    H -->|Messages Retrieved| I{Check Message Cache}
    I -->|Cache Hit| J[Push via WebSocket]
    I -->|Cache Miss| K[Get from Redis]
    K -->|Content Retrieved| J
    J -->|Message Pushed| L[Publish Exposure Event to Pub/Sub]
    L -->|Published| M

    subgraph Connection Management
        B1[WebSocket Disconnection] -->|Triggered| C1{Last Connection for User on Pod?}
        C1 -->|Yes| D1[Remove User:Pod from Redis Hash]
        C1 -->|No| E1[Keep Redis Record]
    end
    
    style A fill:#E8F5E9
    style B fill:#FCE4EC
    style C fill:#FFF3E0
    style D fill:#E3F2FD
    style E fill:#FFF3E0
    style F fill:#FFF3E0
    style G fill:#FCE4EC
    style H fill:#FFF3E0
    style I fill:#FCE4EC
    style J fill:#E8F5E9
    style K fill:#FFF3E0
    style L fill:#E8F5E9
    style M fill:#E8F5E9
    style B1 fill:#E8F5E9
    style C1 fill:#FCE4EC
    style D1 fill:#E3F2FD
    style E1 fill:#FFF3E0

classDef decision fill:#FCE4EC,stroke:#333,stroke-width:2px;
classDef process fill:#FFF3E0,stroke:#333,stroke-width:2px;
classDef storage fill:#E3F2FD,stroke:#333,stroke-width:2px;
classDef io fill:#E8F5E9,stroke:#333,stroke-width:2px;
```

## Flow Description

1. **WebSocket Connection**
   - User initiates WebSocket connection
   - System verifies user identity
   - Failed verification leads to connection rejection

2. **User Categories Setup**
   - System retrieves all categories information for the user
   - Information is stored in the manager module for session management

3. **User Online Status**
   - System stores user online status in Redis hash
   - Uses composite key of username:podname
   - Maintains one record per user per pod regardless of connection count

4. **Offline Message Check**
   - System checks for any pending offline messages
   - If messages exist, they are pulled from Redis List storage
   - If no messages, system waits for new incoming messages

5. **Message Content Retrieval**
   - System first checks message cache for content
   - On cache miss, content is fetched from Redis
   - Retrieved message is prepared for delivery

6. **Message Delivery**
   - Messages are pushed to user via WebSocket connection
   - System publishes exposure event to Google Pub/Sub for tracking
   - Exposure logs are handled by separate consumer process

7. **Connection Management**
   - System tracks WebSocket disconnections
   - Checks if it's the last connection for user on pod
   - Removes user:pod record from Redis only when last connection closes
   - Maintains efficient online user tracking per pod
