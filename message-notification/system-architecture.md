# System Architecture for Message Notification

# The system can be split into 3 parts:
1. System Message Notification Pipeline
2. Push Result Record Pipeline
3. Message Event ETL

## System Message Notification Pipeline

1. An external API server handles the message push requests 
2. The server publishes message events into Pub/Sub
3. A consumer server subscribes to the topic, and when it receives a message:
   - Deconstructs the JSON object
   - Saves the message into Redis
   - Saves push object list to Pub/Sub
4. The WebSocket server listens to the Pub/Sub trigger:
   - When it receives the push object list, checks the cache for message content
   - If content doesn't exist, gets value from Redis
   - Checks if the user is online:
     - If online: pushes message to user via WebSocket
     - If offline: stores list in Redis

## Push Result Record Pipeline

1. When the WebSocket server successfully pushes a message, it publishes the exposure event into Pub/Sub
2. An external API handles the push result events (click, close) and publishes the records into Pub/Sub
3. A server listens to the push result topic and inserts data into MySQL when messages are received

## Message Event ETL

1. An ETL server subscribes to the system message notification topic
2. When a message is received, it inserts the data into MySQL

```mermaid
graph TB
    subgraph "System Message Notification Pipeline"
        A[External API Server] -->|Push Request| B[Pub/Sub Message Events]
        B --> C[Consumer Server]
        C -->|Save Message| D[(Redis)]
        C -->|Push Object List| E[Pub/Sub]
        E --> F[WebSocket Server]
        F -->|Check Cache| D
        F -->|Online Users| G[WebSocket Clients]
        F -->|Offline Messages| D
    end

    subgraph "Push Result Record Pipeline"
        F -->|Success Event| H[Pub/Sub Exposure Events]
        I[External API] -->|Click/Close Events| J[Pub/Sub Result Events]
        K[Result Server] -->|Insert Data| L[(MySQL)]
        H --> K
        J --> K
    end

    subgraph "Message Event ETL"
        B --> M[ETL Server]
        M -->|Insert Data| L
    end

    style A fill:#D4E6F1,stroke:#2E86C1,stroke-width:2px
    style B fill:#FCE4CC,stroke:#E67E22,stroke-width:2px
    style C fill:#D4E6F1,stroke:#2E86C1,stroke-width:2px
    style D fill:#F5B7B1,stroke:#C0392B,stroke-width:2px
    style E fill:#FCE4CC,stroke:#E67E22,stroke-width:2px
    style F fill:#D4E6F1,stroke:#2E86C1,stroke-width:2px
    style G fill:#D4EFDF,stroke:#27AE60,stroke-width:2px
    style H fill:#FCE4CC,stroke:#E67E22,stroke-width:2px
    style I fill:#D4E6F1,stroke:#2E86C1,stroke-width:2px
    style J fill:#FCE4CC,stroke:#E67E22,stroke-width:2px
    style K fill:#D4E6F1,stroke:#2E86C1,stroke-width:2px
    style L fill:#F5B7B1,stroke:#C0392B,stroke-width:2px
    style M fill:#D4E6F1,stroke:#2E86C1,stroke-width:2px

    classDef default font-size:14px,font-weight:bold
