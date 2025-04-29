

# 📚 What is **Debezium Oracle Connector**?

---

# 🛠 What is Debezium?

- **Debezium** is an **open-source** platform for **Change Data Capture (CDC)**.
- It **monitors database changes** (INSERT, UPDATE, DELETE) and **streams them** to **Kafka topics** in real time.
- Think of Debezium as a **real-time replication agent** that captures *row-level changes*.

---

# 🎯 What is **Debezium Oracle Connector**?

✅ **Debezium Oracle Connector** is a special plugin inside Debezium to **capture changes from Oracle databases**.

It works by:
- Reading **Oracle REDO logs** (transaction logs)
- Extracting **DML events** (Insert/Update/Delete)
- Publishing those changes into **Apache Kafka** topics

✅ This enables **streaming Oracle database changes** to other systems like:
- PostgreSQL
- MongoDB
- Elasticsearch
- Data warehouses
- Microservices

---
# 🔥 High-level Architecture

```
Oracle DB  →  Debezium Oracle Connector  →  Kafka Topic
```

---
# 🧠 How Debezium Oracle Connector Works Internally

| Step | Action |
|------|--------|
| 1 | Connects to Oracle DB using JDBC |
| 2 | Mines Oracle REDO / ARCHIVE logs (using LogMiner or XStream) |
| 3 | Parses DML operations |
| 4 | Sends changes to Apache Kafka topics |
| 5 | Other apps consume from Kafka |

---

# 🔵 Modes of Operation

Debezium Oracle connector supports **two main ways** to read changes:

| Mode | How it works | Notes |
|------|--------------|-------|
| LogMiner mode | Uses Oracle's built-in **LogMiner** package to mine redo/archive logs | ✅ Works with Standard/Enterprise editions |
| XStream mode | Uses Oracle's **XStream API** (extra option, usually licensed separately) | ✅ Requires Oracle GoldenGate license |

✅ **LogMiner** is **free** and commonly used.

---

# ⚙️ Prerequisites to Use Debezium Oracle Connector

You need:
| Item | Details |
|------|---------|
| Oracle DB Version | 11g, 12c, 18c, 19c, 21c supported |
| Kafka | Running Apache Kafka cluster |
| Kafka Connect | Kafka Connect running with Debezium plugin installed |
| Permissions | An Oracle user with access to `DBA_LOGMNR_*` views, etc. |
| Log Retention | Archive REDO logs should be retained long enough for Debezium to read changes |
| Supplemental Logging | Must be enabled in Oracle to capture full row data |

---
# ✅ Minimal Steps to Set Up

### 1. Prepare Oracle DB
- Enable **ARCHIVELOG mode** (which you already asked earlier!)
- Enable **Supplemental Logging**:
  ```sql
  ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
  ```
- Create a special **Debezium user** with required privileges.

---

### 2. Set Up Kafka and Kafka Connect
- Install Apache Kafka
- Install Debezium Oracle connector plugin into Kafka Connect
- Start Kafka Connect cluster

---

### 3. Register Debezium Oracle Connector
POST a connector config like:

```json
POST http://localhost:8083/connectors

{
  "name": "oracle-connector",
  "config": {
    "connector.class": "io.debezium.connector.oracle.OracleConnector",
    "database.hostname": "localhost",
    "database.port": "1521",
    "database.user": "debezium",
    "database.password": "dbz",
    "database.dbname": "FREEPDB1",
    "database.server.name": "dbserver1",
    "database.connection.adapter": "logminer",
    "database.history.kafka.bootstrap.servers": "kafka:9092",
    "database.history.kafka.topic": "schema-changes.dbserver1"
  }
}
```

✅ This will start capturing changes from Oracle!

---

# 🏗 Real-world Usage Example

- Capture Oracle table changes
- Stream into Kafka topics
- Build real-time data pipelines for:
  - Analytics
  - Replication to PostgreSQL
  - Elasticsearch indexing
  - Event-driven architectures

---

# 📋 Quick Table of Advantages and Limitations

| Advantages | Limitations |
|------------|-------------|
| Near real-time data capture | Needs Kafka Connect and Kafka infra |
| Open source and powerful | Oracle setup (redo logs, archive, permissions) can be complex |
| Works with Standard Oracle licenses (LogMiner mode) | XStream mode requires Oracle GoldenGate license |

---

# 🚀 Quick Recap

| Item | Details |
|------|---------|
| Purpose | Stream Oracle DB changes into Kafka |
| Works by | Reading REDO logs via LogMiner |
| Needs | Kafka, Kafka Connect, Archive logs, Supplemental logging |
| Output | Real-time Kafka topics with database changes |
| Most common mode | `LogMiner` (free) |

---

# 📢 Bonus Tip:
Debezium **Oracle connector** in `LogMiner` mode is **"read-only"** — it **won't modify** your Oracle database at all. It just **listens** to log changes.

---

