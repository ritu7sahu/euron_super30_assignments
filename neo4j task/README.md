# 🕵️ Project SENTINEL: Graph Fraud Detection with Neo4j & Python

> **Uncovering Hidden Fraud Networks in Relational Data Using Neo4j and Cypher Queries**

![Project SENTINEL Banner](thumbnail.jpg)

---

## 📌 Overview

Fraud networks frequently operate across shared infrastructure (such as shared devices, IP addresses, or payment terminals) to bypass account-level security checks. Finding these hidden linkages in standard relational tables requires complex SQL multi-table joins that do not scale well.

This repository demonstrates how to load transaction data from **Project SENTINEL** into **Neo4j** using Python, establishing graph connections between `Customer`, `Device`, and `Merchant` entities. With Cypher queries, we easily expose fraud clusters, shared device rings, and suspicious multi-hop pathways.

---

## 🎯 Key Capabilities & Cypher Queries

### 1. Unique Node Constraints & Idempotent Ingestion
To prevent duplicate nodes upon multiple script runs, unique constraints are applied prior to data loading using `MERGE`:

```cypher
CREATE CONSTRAINT customer_unique IF NOT EXISTS FOR (c:Customer) REQUIRE c.customer_id IS UNIQUE;
CREATE CONSTRAINT device_unique IF NOT EXISTS FOR (d:Device) REQUIRE d.device_id IS UNIQUE;
CREATE CONSTRAINT merchant_unique IF NOT EXISTS FOR (m:Merchant) REQUIRE m.merchant_id IS UNIQUE;
```

---

### 2. Cypher Query 1: Devices Shared by $\ge 3$ Customers in 7 Days
Identifies potential device-sharing fraud rings operating within a moving 7-day window.

```cypher
MATCH (d:Device {device_id: $device_id})<-[:USED_DEVICE]-(c:Customer)-[p:PAID]->(m:Merchant)
WHERE p.device_id = d.device_id
  AND p.time >= datetime($window_start)
  AND p.time < datetime($window_start) + duration({days: 7})
RETURN
    d.device_id AS device_id,
    c.customer_id AS customer_id,
    COUNT(p) AS transaction_count,
    COUNT(DISTINCT m) AS merchant_count,
    SUM(p.amount) AS total_amount,
    COLLECT(DISTINCT p.fraud_label) AS fraud_labels
ORDER BY transaction_count DESC;
```

---

### 3. Cypher Query 2: Customers Within 2 Hops of a Fraud Payment
Locates accounts within 2 relationship steps (connected via shared devices or shared merchants) from a flagged fraudulent payment.

```cypher
MATCH (fc:Customer)-[:PAID {fraud_label: 1}]->(:Merchant)
WITH DISTINCT fc
MATCH (fc)-[:USED_DEVICE|PAID*1..2]-(near:Customer)
WHERE near <> fc
RETURN count(DISTINCT near) AS total_nearby;
```

---

### 4. Cypher Query 3: Merchants Most Often Paid from Shared Devices
Ranks merchants processing the highest volume of payments originating from devices shared by 3 or more users.

```cypher
MATCH (c:Customer)-[p:PAID]->(:Merchant)
WITH p.device_id AS device_id, collect(DISTINCT c.customer_id) AS customers
WHERE size(customers) >= 3
WITH collect(device_id) AS shared_devices
MATCH (c:Customer)-[p:PAID]->(m:Merchant)
WHERE p.device_id IN shared_devices
RETURN m.merchant_id AS merchant,
       count(p) AS payments_from_shared_devices,
       count(DISTINCT c) AS customers,
       sum(CASE WHEN p.fraud_label = 1 THEN 1 ELSE 0 END) AS fraud_payments
ORDER BY payments_from_shared_devices DESC
LIMIT 10;
```

---

## 🛠️ Project Setup & Installation

### Step 1: Run Neo4j with Docker
Start a local instance of Neo4j with ports `7474` (HTTP Browser) and `7687` (Bolt API):

```bash
docker run -d \
  --name neo4j_sentinel \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/password123 \
  -v neo4j_data:/data \
  neo4j:latest
```

### Step 2: Environment Setup
Create a `.env` file based on `.env.example`:

```bash
cp .env.example .env
```

Set your credentials in `.env`:
```env
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=password123
```

### Step 3: Install Dependencies & Execute Notebook
```bash
pip install neo4j pandas glob2 python-dotenv
jupyter notebook neo4j2.ipynb
```

---

## 🖼️ Screenshots & Outputs

Place your exported screenshots inside an `assets/` folder in your repository:

| Screenshot Description | File Path |
| :--- | :--- |
| **Ingestion Script Execution & Neo4j Verification** | `assets/01_ingestion_output.png` |
| **Top Merchants Paid via Shared Devices Result** | `assets/02_top_merchants_table.png` |
| **Suspicious Cluster Graph Visual (Neo4j Browser)** | `assets/03_suspicious_cluster_graph.png` |

---

## 🔍 Suspicious Cluster Analysis

* **Device Identifier:** `D_c48c331b4ed19bce`
* **Cluster Overview:** Device `D_c48c331b4ed19bce` was linked to **3 separate customer accounts** (`C_a3fef1d1bb96c6fe`, `C_31b398af779a1a10`, and `C_2bd85847476a6871`) within a single 7-day period.
* **Fraud Investigation Findings:**
  1. Accounts `C_a3fef1d1bb96c6fe` and `C_2bd85847476a6871` both logged transactions labeled as confirmed fraud (`fraud_label = 1`).
  2. Account `C_31b398af779a1a10` has not yet been flagged with a direct fraud label, but shares the exact same hardware device and targeted merchant endpoint (`M_8613b75179e11c81`).
* **Risk Recommendation:** Account `C_31b398af779a1a10` should be immediately flagged for risk review due to high graph proximity (1-hop shared hardware) to confirmed fraudulent entities.

