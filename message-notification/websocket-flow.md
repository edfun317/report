# WebSocket Connection Flow

```mermaid
flowchart TD
    A[User Creates WebSocket Connection] -->|Connect| B{Verify User Identity}
    B -->|Failed| C[Break Connection]
    B -->|Passed| D[Get User Categories Info]
    D -->|Categories Retrieved| E[Store in Manager Module]
    E -->|Info Stored| F{Check Offline Messages}
    F -->|No Messages| L[Wait for New Messages]
    F -->|Has Messages| G[Pull from Redis List]
    G -->|Messages Retrieved| H{Check Message Cache}
    H -->|Cache Hit| I[Push via WebSocket]
    H -->|Cache Miss| J[Get from Redis]
    J -->|Content Retrieved| I
    I -->|Message Pushed| K[Publish Exposure Event to Pub/Sub]
    K -->|Published| L
    
    style A fill:#E8F5E9
    style B fill:#FCE4EC
    style C fill:#FFF3E0
    style D fill:#FFF3E0
    style E fill:#FFF3E0
    style F fill:#FCE4EC
    style G fill:#FFF3E0
    style H fill:#FCE4EC
    style I fill:#FFF3E0
    style J fill:#E8F5E9
    style K fill:#FFF3E0
    style L fill:#E8F5E9

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

3. **Offline Message Check**
   - System checks for any pending offline messages
   - If messages exist, they are pulled from Redis List storage
   - If no messages, system waits for new incoming messages

4. **Message Content Retrieval**
   - System first checks message cache for content
   - On cache miss, content is fetched from Redis
   - Retrieved message is prepared for delivery

5. **Message Delivery**
   - Messages are pushed to user via WebSocket connection
   - System publishes exposure event to Google Pub/Sub for tracking
   - Exposure logs are handled by separate consumer process
