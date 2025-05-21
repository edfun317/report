# Message Notification System Architecture

# System Components
The system consists of three main pipelines:
1. Message Notification Pipeline
2. Event Recording Pipeline
3. Data ETL Pipeline

## Message Notification Pipeline

1. API Server receives message notification requests
2. API Server publishes notification events to Pub/Sub
3. Message Consumer Server processes the Pub/Sub events:
   - Parses the JSON payload
   - Stores message content in Redis
   - Publishes recipient list to Pub/Sub
4. WebSocket Server handles message delivery:
   - Receives recipient list from Pub/Sub
   - Retrieves message content from Redis cache
   - Delivers messages to online users via WebSocket
   - Stores messages for offline users in Redis

## Event Recording Pipeline

1. WebSocket Server publishes delivery events to Pub/Sub
2. API Server handles user interaction events (view, click) and publishes to Pub/Sub
3. Event Consumer Server processes all events and stores them in MySQL

## Data ETL Pipeline

1. ETL Server subscribes to notification events from Pub/Sub
2. ETL Server processes and stores event data in MySQL

```mermaid
graph TB
    subgraph "Message Notification Pipeline"
        A[API Server] -->|Notification Request| B[Pub/Sub Events]
        B --> C[Message Consumer]
        C -->|Store Content| D[(Redis)]
        C -->|Recipient List| E[Pub/Sub]
        E --> F[WebSocket Server]
        F -->|Get Content| D
        F -->|Deliver Message| G[WebSocket Clients]
        F -->|Store Message| D
    end

    subgraph "Event Recording Pipeline"
        F -->|Delivery Event| H[Pub/Sub Events]
        I[API Server] -->|Interaction Event| J[Pub/Sub Events]
        K[Event Consumer] -->|Store Events| L[(MySQL)]
        H --> K
        J --> K
    end

    subgraph "Data ETL Pipeline"
        B --> M[ETL Server]
        M -->|Store Data| L
    end

    style A fill:#D4E6F1,stroke:#2E86C1,stroke-width:2px,color:#000000
    style B fill:#FCE4CC,stroke:#E67E22,stroke-width:2px,color:#000000
    style C fill:#D4E6F1,stroke:#2E86C1,stroke-width:2px,color:#000000
    style D fill:#F5B7B1,stroke:#C0392B,stroke-width:2px,color:#000000
    style E fill:#FCE4CC,stroke:#E67E22,stroke-width:2px,color:#000000
    style F fill:#D4E6F1,stroke:#2E86C1,stroke-width:2px,color:#000000
    style G fill:#D4EFDF,stroke:#27AE60,stroke-width:2px,color:#000000
    style H fill:#FCE4CC,stroke:#E67E22,stroke-width:2px,color:#000000
    style I fill:#D4E6F1,stroke:#2E86C1,stroke-width:2px,color:#000000
    style J fill:#FCE4CC,stroke:#E67E22,stroke-width:2px,color:#000000
    style K fill:#D4E6F1,stroke:#2E86C1,stroke-width:2px,color:#000000
    style L fill:#F5B7B1,stroke:#C0392B,stroke-width:2px,color:#000000
    style M fill:#D4E6F1,stroke:#2E86C1,stroke-width:2px,color:#000000

    classDef default font-size:14px,font-weight:500
