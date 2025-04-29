# 📄 CDC Implementation Plan: Oracle ➔ Kafka ➔ PostgreSQL (using Debezium)

---

## 🧩 Project Goal
> Implement a **Change Data Capture (CDC)** pipeline to **replicate data changes in real-time** from **Oracle database** to **PostgreSQL database**, using **Apache Kafka** as the central messaging bus, and **Debezium** as the CDC engine.

---

## 📦 Components Used

| Component | Purpose |
|:---|:---|
| Oracle Database | Source of truth (where original data is created/updated/deleted) |
| Debezium Oracle Connector | Captures changes from Oracle and pushes them to Kafka |
| Apache Kafka | Message broker to carry CDC events |
| Kafka Connect | Platform to run connectors (Debezium Source, JDBC Sink) |
| PostgreSQL Database | Target system to replicate Oracle changes |
| JDBC Sink Connector | Reads Kafka events and writes them into PostgreSQL |

---

## 🛠 Step-by-Step Implementation

---

### 🥇 Step 1: Prepare Oracle for CDC

✅ Enable **Archive Log Mode** and **Supplemental Logging**:

```sql
-- Check current mode
ARCHIVE LOG LIST;

-- Enable archive log if needed
SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
ALTER DATABASE ARCHIVELOG;
ALTER DATABASE OPEN;

-- Enable minimal supplemental logging
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
```

✅ Create a **Replication User** in Oracle:

```sql
CREATE USER debezium IDENTIFIED BY dbz;
GRANT CONNECT, RESOURCE TO debezium;
GRANT SELECT ANY TABLE TO debezium;
GRANT SELECT_CATALOG_ROLE TO debezium;
GRANT EXECUTE_CATALOG_ROLE TO debezium;
GRANT SELECT ON V$LOGMNR_CONTENTS TO debezium;
GRANT LOGMINING TO debezium;
```

✅ Create a **sample table**:

```sql
CREATE TABLE debezium.test_table (
  id NUMBER(10) PRIMARY KEY,
  name VARCHAR2(50),
  description VARCHAR2(100)
);

INSERT INTO debezium.test_table VALUES (1, 'First record', 'Initial Insert');
COMMIT;
```

---

### 🥈 Step 2: Install and Configure Kafka

✅ Install and Start Kafka + Zookeeper:

```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
bin/kafka-server-start.sh config/server.properties
```

✅ Install Kafka Connect (already part of Kafka distribution).

✅ Install **Debezium Oracle Connector**:

- Download Debezium Oracle Connector zip.
- Extract it into Kafka Connect's `plugin.path` directory.
- Example:
  ```bash
  unzip debezium-connector-oracle-2.x.zip -d /opt/kafka/plugins/
  ```

✅ Restart Kafka Connect:

```bash
bin/connect-distributed.sh config/connect-distributed.properties
```

---

### 🥉 Step 3: Deploy Debezium Oracle Source Connector

✅ Create `oracle-source.json`:

```json
{
  "name": "oracle-source-connector",
  "config": {
    "connector.class": "io.debezium.connector.oracle.OracleConnector",
    "tasks.max": "1",
    "database.hostname": "oracle_host",
    "database.port": "1521",
    "database.user": "debezium",
    "database.password": "dbz",
    "database.dbname": "ORCL",
    "database.pdb.name": "XEPDB1",
    "database.server.name": "oracle_cdc",
    "table.include.list": "DEBEZIUM.TEST_TABLE",
    "database.history.kafka.bootstrap.servers": "kafka_host:9092",
    "database.history.kafka.topic": "schema-changes.oracle",
    "log.mining.strategy": "online_catalog"
  }
}
```

✅ Post it to Kafka Connect:

```bash
curl -X POST -H "Content-Type: application/json" --data @oracle-source.json http://kafka_connect_host:8083/connectors
```

✅ Result:  
- Oracle CDC changes will start streaming into a Kafka topic named `oracle_cdc.DEBEZIUM.TEST_TABLE`.

---

### 🏅 Step 4: Verify Kafka CDC Topics

✅ Check topics:

```bash
bin/kafka-topics.sh --bootstrap-server kafka_host:9092 --list
```

✅ Consume from topic:

```bash
bin/kafka-console-consumer.sh --bootstrap-server kafka_host:9092 --topic oracle_cdc.DEBEZIUM.TEST_TABLE --from-beginning
```

✅ You should see JSON change events like:

```json
{
  "payload": {
    "op": "c",
    "after": {
      "id": 1,
      "name": "First record",
      "description": "Initial Insert"
    }
  }
}
```
Where `op=c` means **create**.

---

### 🥇 Step 5: Prepare PostgreSQL for Target

✅ Connect to PostgreSQL.

✅ Create the target table:

```sql
CREATE TABLE test_table (
  id INT PRIMARY KEY,
  name TEXT,
  description TEXT
);
```

✅ Alternatively: Allow Kafka Sink Connector to **auto-create** the table.

---

### 🥈 Step 6: Deploy Kafka Connect JDBC Sink Connector (for PostgreSQL)

✅ Create `postgres-sink.json`:

```json
{
  "name": "postgres-sink-connector",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
    "tasks.max": "1",
    "connection.url": "jdbc:postgresql://postgres_host:5432/your_database",
    "connection.user": "postgres_user",
    "connection.password": "postgres_password",
    "topics": "oracle_cdc.DEBEZIUM.TEST_TABLE",
    "insert.mode": "upsert",
    "pk.mode": "record_key",
    "pk.fields": "ID",
    "auto.create": "true",
    "auto.evolve": "true"
  }
}
```

✅ Post it to Kafka Connect:

```bash
curl -X POST -H "Content-Type: application/json" --data @postgres-sink.json http://kafka_connect_host:8083/connectors
```

✅ Result:
- Kafka Connect will read CDC events from Kafka topics and apply them to the PostgreSQL `test_table`.

---

### 🏆 Step 7: End-to-End Testing

✅ Insert a new row in Oracle:

```sql
INSERT INTO debezium.test_table VALUES (2, 'Second record', 'Welcome!');
COMMIT;
```

✅ Check PostgreSQL:

```sql
SELECT * FROM test_table;
```

✅ You will see both Oracle and PostgreSQL tables match — **real-time CDC working!** 🎯

---

# 🔥 Architecture Diagram

```
[ Oracle DB (test_table) ]
        ↓
Debezium Oracle Connector (inside Kafka Connect)
        ↓
Kafka Topic (oracle_cdc.DEBEZIUM.TEST_TABLE)
        ↓
Kafka Connect Sink Connector (PostgreSQL JDBC)
        ↓
[ PostgreSQL DB (test_table) ]
```

---

# ✅ Checklist for Production Readiness

| Task | Status |
|:---|:---|
| Debezium Oracle Connector installed | ✅ |
| Kafka Connect secured and monitored | ✅ |
| Archive logs retention properly set | ✅ |
| Connector offset management tested | ✅ |
| PostgreSQL replication lag monitored | ✅ |
| Dead Letter Queue (DLQ) configured (optional) | ✅ |

---

# 📢 Conclusion

✅ This architecture enables **real-time**, **reliable**, and **scalable** CDC from Oracle to PostgreSQL.

✅ It leverages **open-source tools** (Kafka, Debezium) to avoid expensive proprietary software.

✅ It’s flexible to extend later to multiple consumers (e.g., ElasticSearch, BigQuery, etc.) without impacting Oracle.

✅ Ideal for microservices, reporting, and real-time analytics pipelines.

---


# 🚀 Timeline

| Task | Estimate |
|:---|:---|
| Install/Set Up Kafka & Connect | 1 day |
| Oracle configuration and testing | 1 day |
| Deploy Debezium and CDC Source Connector | 1 day |
| Set up PostgreSQL Sink Connector | 0.5 day |
| Testing and validation | 1 day |

✅ Full CDC setup possible within **4–5 working days**.

---


