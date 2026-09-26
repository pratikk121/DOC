# Database Questions & Schema Design Viva Preparation

## 1. Relational Schema & Normalization

### Q1: Why did you choose PostgreSQL over NoSQL (like MongoDB) or dedicated Time-Series databases (like InfluxDB)?
**Answer:**  
"1. **Structured Relational Integrity:** Our domain requires strict relational relationships: a `gateway` manages multiple `sensor_nodes`, a node produces `telemetry`, and a node triggers `incidents`. Relational foreign keys and cascade rules prevent orphaned data.
2. **ACID Transactions & Triggers:** PostgreSQL supports full ACID transactions and `PL/pgSQL` triggers. This allowed us to execute autonomic incident detection and state transitions atomically within the database engine itself.
3. **Native JSONB & Realtime Capabilities:** PostgreSQL provides `JSONB` support for semi-structured incident metadata alongside native Write-Ahead Log (WAL) logical decoding for zero-latency WebSocket streaming via Supabase Realtime."

---

### Q2: Is your database schema normalized? Explain any intentional denormalization.
**Answer:**  
"The schema is strictly in **Third Normal Form (3NF)** with one intentional engineering optimization:
* `gateways`, `sensor_nodes`, and `incidents` are in 3NF with unique primary keys and foreign key references.
* In the `telemetry` table, we store both `node_id` and `gateway_id`. While `gateway_id` could theoretically be derived by joining through `sensor_nodes`, storing `gateway_id` directly in `telemetry` allows:
  1. Instant querying of RF gateway performance (e.g., gateway load and RSSI distributions) without performing expensive `JOIN` operations across millions of time-series rows.
  2. Future support for multi-gateway reception where multiple gateways might hear the same node broadcast with different RSSI/SNR values."

---

### Q3: What is the purpose of the `UNIQUE (node_id, sequence_number)` constraint on the `telemetry` table?
**Answer:**  
"In wireless radio communications, packet retransmissions or serial bridge retries frequently occur. Without deduplication, retransmitted packets would duplicate historical time-series entries and skew analytics.  
The composite unique constraint `UNIQUE (node_id, sequence_number)` guarantees that every packet sequence from a specific node is inserted exactly once. If a duplicate arrives, PostgreSQL enforces uniqueness by throwing error code `23505 (unique_violation)`, which the bridge handles cleanly without inserting duplicate rows."

---

## 2. Triggers & Stored Logic

### Q4: Explain the step-by-step logic inside the `fn_evaluate_telemetry_alert()` trigger function.
**Answer:**  
"When a new row is inserted into `telemetry`, the `AFTER INSERT` trigger executes the following deterministic logic:
1. **Threat Assessment:** It evaluates whether the physical sensor readings exceed fire thresholds:
   $$\text{Flame IR} < 100 \quad\lor\quad \text{Smoke ADC} > 2000 \quad\lor\quad \text{Temperature} > 55.0^\circ\text{C}$$
2. **State Transition:** If true, it immediately executes:
   ```sql
   UPDATE sensor_nodes SET status = 'ALERT' WHERE node_id = NEW.node_id;
   ```
3. **Incident Debouncing:** It queries `incidents` for an existing ticket with `status = 'ACTIVE'` for that `node_id`.
   * If **none exists**, it inserts a new `incidents` record with `severity = 'CRITICAL'`, dynamically determines `alert_type` (`WILDFIRE_FLAME`, `HIGH_SMOKE`, or `CRITICAL_HEAT`), and stores a JSONB snapshot of the sensor readings.
   * If an active incident **already exists**, it skips creating a duplicate ticket, debouncing the alarm.
4. **Auto-Recovery:** If the fire condition is false and readings have normalized ($T < 45^\circ\text{C}$, $\text{Smoke} < 1500$, $\text{Flame} > 1000$), it returns the node status to `'ONLINE'`."

---

### Q5: What is the purpose of `trg_verify_telemetry_gateway`?
**Answer:**  
"It is a `BEFORE INSERT` trigger on the `telemetry` table that performs autonomic health tracking:
* It executes `UPDATE gateways SET last_seen_at = NOW(), status = 'ACTIVE' WHERE gateway_id = NEW.gateway_id;`
* It executes `UPDATE sensor_nodes SET last_telemetry_at = NOW() WHERE node_id = NEW.node_id;`  
This guarantees that device liveness timestamps are updated synchronously whenever valid data arrives, without requiring separate heartbeat polling."

---

## 3. Indexing & Realtime Streaming

### Q6: What indexes did you create and why?
**Answer:**  
"We created three targeted B-Tree indexes:
1. `idx_telemetry_node_created ON telemetry (node_id, created_at DESC)`: Optimizes dashboard time-series queries which filter by a specific sensor node and order by timestamp descending.
2. `idx_telemetry_node_seq ON telemetry (node_id, sequence_number)`: Accelerates unique constraint lookups during high-frequency packet ingestion.
3. `idx_incidents_status ON incidents (status, triggered_at DESC)`: Accelerates the active incident rail query on the tactical map by indexing only active/unresolved tickets."

---

### Q7: How does Supabase Realtime stream database inserts to the web browser?
**Answer:**  
"Supabase Realtime leverages PostgreSQL's **Logical Decoding** feature:
1. PostgreSQL writes every `INSERT`, `UPDATE`, and `DELETE` operation to its **Write-Ahead Log (WAL)**.
2. Supabase's Realtime server (built in Elixir/Phoenix) reads this logical replication stream in memory.
3. It broadcasts the change as a JSON payload across subscribed WebSocket channels (`wss://`) to connected client browsers running the `supabase-js` client in under 50 milliseconds."
