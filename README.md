# Real-Time Streaming: Kafka + Debezium + Spark Structured Streaming

**Tagline:** *Change Data Capture → Streaming Analytics in minutes, not months.*

This is a production-style **end-to-end data pipeline**:
- **Debezium (Kafka Connect)** streams **row-level changes (CDC)** from OLTP databases (MySQL/Postgres)
- Events land in **Apache Kafka** topics (durable, replayable)
- **Spark Structured Streaming** performs **real-time ETL, enrichment, windowed aggregations** with **watermarking** + **exactly-once sinks**
- Results feed a **notification service** + **Real-Time-App** UI for live insights

> Designed to demonstrate **distributed systems + big-data engineering**: CDC semantics, event design, stateful streaming, recovery, backpressure, and horizontal scaling.

---

## 📌 Architecture (High-Level)

![Pipeline Overview](images/architecture.png)

```mermaid
flowchart LR
  %% Cleaned for GitHub Mermaid: no parentheses, no \n, no special chars in labels

  subgraph OLTP [Source Databases]
    M[MySQL]:::db
    P[Postgres]:::db
  end

  subgraph Connect [Kafka Connect and Debezium]
    D1[MySQL Connector]:::debezium
    D2[Postgres Connector]:::debezium
  end

  subgraph Kafka [Kafka Cluster]
    T1[dbserver1.inventory.tableA]
    T2[dbserver1.inventory.tableB]
    T3[pgserver1.public.tableX]
    DLQ[dead letter topic]:::warn
  end

  subgraph Spark [Structured Streaming Job]
    J1[Parse and Normalize keyed by PK]:::spark
    J2[Watermark and Windows]:::spark
    J3[Aggregations and Joins]:::spark
    J4[Idempotent Sink Upsert MERGE]:::spark
  end

  subgraph Sinks [Serving and Apps]
    ST[Sink OLAP]:::db
    NS[notification service]:::svc
    UI[Real Time App]:::ui
  end

  %% flows
  M --> D1 --> T1
  M --> D1 --> T2
  P --> D2 --> T3

  T1 --> J1
  T2 --> J1
  T3 --> J1
  J1 --> J2 --> J3 --> J4 --> ST
  J3 -. alerts .-> NS
  ST --> UI

  %% styles
  classDef db fill:#f3f8ff,stroke:#4e79a7,stroke-width:1px;
  classDef debezium fill:#fff8ef,stroke:#f28e2b,stroke-width:1px;
  classDef spark fill:#f7fff3,stroke:#59a14f,stroke-width:1px;
  classDef ui fill:#fdf1ff,stroke:#b07aa1,stroke-width:1px;
  classDef svc fill:#ffffff,stroke:#e15759,stroke-width:1px;
  classDef warn fill:#fff3f3,stroke:#e15759,stroke-dasharray:3 3;
```
