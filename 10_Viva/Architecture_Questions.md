# Architecture Questions & System Design Viva Preparation

## 1. High-Level Architectural Decisions

### Q1: Explain the rationale behind the 4-tier architecture (Edge $\to$ Bridge $\to$ Database $\to$ EOC UI).
**Answer:**  
"The 4-tier architecture ensures strict separation of concerns, scalability, and physical fault isolation:
1. **Edge Sensing Layer (ESP32):** Dedicated exclusively to deterministic physical sensor sampling and low-power RF broadcast without memory-heavy TCP/IP networking stacks.
2. **Bridge / Ingestion Layer (Python on Base Station Host):** Decouples hardware serial protocols (UART) from cloud web APIs, handling COM port discovery, physical range boundary sanitization, and network retry backoff.
3. **Database & Autonomic Logic Layer (PostgreSQL / Supabase):** Serves as the single source of truth; stored triggers evaluate fire threats and debounce alarms at the database layer rather than relying on an intermediate Node.js daemon that could crash.
4. **Presentation Layer (Next.js 15):** High-performance tactical dashboard focused solely on rendering GIS maps, time-series charts, and acoustic alert dispatching via reactive WebSocket streams."

---

### Q2: Why did you choose 433 MHz LoRa over 2.4 GHz Wi-Fi, Zigbee, Bluetooth, or Cellular (4G/5G)?
**Answer:**  
"Wireless propagation in dense forest environments is governed by the **Friis Transmission Equation** and the **Modified Exponential Decay Model for Foliage Loss**:
1. **Foliage & Moisture Attenuation:** Trees and wet foliage absorb $2.4\text{ GHz}$ signals (Wi-Fi, Bluetooth, Zigbee) rapidly because the wavelength ($\lambda \approx 12.5\text{ cm}$) resonates closely with leaf dimensions and moisture droplets. In contrast, $433\text{ MHz}$ signals ($\lambda \approx 69.3\text{ cm}$) have a wavelength larger than typical leaves and needles, enabling diffraction around obstacles and penetrating deep canopy vegetation with minimal attenuation.
2. **Infrastructure Independence:** Cellular (4G/5G/NB-IoT) requires terrestrial base station towers, which do not exist in wilderness national parks and mountainous valleys.
3. **Power Consumption:** LoRa transmitters consume $\approx 100\text{--}120\text{ mA}$ during transmission for fractions of a second, compared to Cellular modules (SIM800/SIM7600) which require 2A burst currents, making solar-battery operation impractical."

---

### Q3: Why did you implement fire threat detection inside Database Triggers (`PL/pgSQL`) rather than a backend cron or polling service?
**Answer:**  
"Implementing business logic in database triggers (`trg_evaluate_telemetry`) delivers three major advantages:
1. **Zero Latency (Immediate Execution):** The trigger fires synchronously inside the database transaction *immediately* upon row insertion. There is zero polling interval or cron delay.
2. **Atomic Consistency:** The transition of `sensor_nodes.status = 'ALERT'` and the insertion into `incidents` occur within the exact same database transaction as the telemetry row insert, guaranteeing no inconsistent intermediary states.
3. **Resilience to Application Server Downtime:** Even if the web application or API server restarts or experiences high traffic, the database continues evaluating incoming telemetry from field gateways autonomously."

---

### Q4: Compare `SUPABASE_DIRECT` mode and `API_GATEWAY` mode in your Python Bridge.
**Answer:**  
"The bridge supports two configurable ingestion architectures:

| Feature / Metric | `SUPABASE_DIRECT` Mode | `API_GATEWAY` Mode |
| :--- | :--- | :--- |
| **Ingestion Target** | `https://<ref>.supabase.co/rest/v1/telemetry` | `https://<domain>/api/telemetry` |
| **Intermediary Hops** | 1 (Bridge $\to$ Cloud DB) | 2 (Bridge $\to$ Next.js Serverless $\to$ Cloud DB) |
| **Average Ingestion Latency** | $\approx 45\text{--}80\text{ ms}$ | $\approx 120\text{--}250\text{ ms}$ |
| **Authentication** | Supabase Service Role JWT | Pre-shared Bearer API Key (`GATEWAY_API_KEY`) |
| **Use Case** | High-throughput field operations & minimal latency | Enterprise setups requiring custom pre-processing or audit logging |

Both modes are implemented and verified in the codebase."

---

### Q5: Why did you use WebSocket Change-Data-Capture (CDC) instead of HTTP Polling or an external MQTT Broker?
**Answer:**  
"1. **Elimination of Polling Overhead:** HTTP polling (e.g., querying `/api/telemetry` every 2 seconds) wastes server bandwidth, floods the database with identical queries, and introduces artificial latency equal to the polling interval.
2. **Supabase Realtime vs MQTT:** Supabase Realtime listens directly to PostgreSQL's native **Write-Ahead Log (WAL)** replication stream. When a row is committed, PostgreSQL pushes the change directly over a WebSocket connection to connected browsers.
3. **Single Technology Stack:** Using PostgreSQL WAL replication eliminated the need to maintain, secure, and pay for an external MQTT broker (e.g., Mosquitto/HiveMQ) and separate bridge daemons."

---

### Q6: How are responsibilities divided between Edge Computing and Cloud Computing in your project?
**Answer:**  
"We apply an **Edge-Weighted Hybrid Model**:
* **Edge MCU Responsibilities (ESP32):**
  * Hardware sensor acquisition and electrical filtering.
  * Analog-to-digital conversion and JSON serialization.
  * Sub-GHz RF modulation, hardware CRC calculation, and packet transmission.
  * Physical signal quality measurement (RSSI / SNR calculation on gateway).
* **Cloud & Serverless Responsibilities (Supabase & Next.js):**
  * Long-term persistent storage and indexing of high-volume time-series data.
  * Complex multi-sensor trigger evaluation and incident ticketing.
  * Tactical GIS spatial rendering and dynamic boundary visualization.
  * Operator session management, audit trails, and multi-format data export (CSV/JSON)."
