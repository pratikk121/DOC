# Viva Preparation: Core Project Questions & Master Answers

> **Target Audience:** Final-Year B.Tech Viva & Capstone Defense Panel  
> **Approach:** Concise, technically rigorous, direct answers grounded exclusively in the actual codebase.

---

## 1. Basic Project Identification & Problem Definition

### Q1: What is the official title of your project?
**Answer:**  
"The project is titled **Edge-to-Cloud IoT Early Wildfire Detection and Tactical Operations Center Platform**."

---

### Q2: What exact problem does this project solve?
**Answer:**  
"It solves the critical latency and communication gap in early wildfire detection. Traditional systems rely on optical lookout towers or earth-observation satellites (like MODIS/VIIRS) which suffer from multi-hour orbital revisit intervals and cannot detect ground-level smoldering fires under thick tree canopies. Furthermore, remote forest environments lack cellular (4G/5G) or Wi-Fi infrastructure.  
Our system deploys rugged, multi-modal IoT sensor nodes directly beneath the forest canopy, transmits real-time environmental metrics over Sub-GHz 433 MHz LoRa radio (which penetrates dense foliage), and streams telemetry to a cloud-based Emergency Operations Center (EOC) with automated database triggers that dispatch alerts in under 2 seconds."

---

### Q3: What motivated you to choose this project?
**Answer:**  
"Wildfires cause catastrophic environmental degradation and loss of life every year. In fire dynamics, the first 15 to 30 minutes of a combustion event are the most critical window for containment. Once a ground fire transitions into a crowning canopy fire, containment costs multiply exponentially. Building a low-power, infrastructure-independent, sub-GHz telemetry network with sub-second cloud situational awareness directly addresses the root bottleneck in emergency forestry management."

---

### Q4: What were your primary technical objectives?
**Answer:**  
"We had four primary objectives:
1. **Edge Multi-Sensing:** Build an autonomous ESP32 sensor node capable of sampling ambient temperature, humidity, combustible gases/smoke, and optical infrared flame radiation simultaneously.
2. **Sub-GHz Long-Range Telemetry:** Transmit serialized environmental telemetry across non-line-of-sight wilderness using 433 MHz LoRa without relying on public cellular networks.
3. **Autonomic Cloud Event Processing:** Implement an automated database trigger engine in PostgreSQL that evaluates threat thresholds, debounces alarms, and manages node health states directly inside the database.
4. **Real-Time Tactical EOC Workstation:** Develop a single-page situational awareness dashboard using Next.js 15, Leaflet GIS, Recharts, and the Web Audio API that renders live updates via WebSockets without polling."

---

### Q5: Who are the primary target users of this platform?
**Answer:**  
"The primary end-users are:
1. **Emergency Operations Center (EOC) Dispatchers:** Who monitor regional fire sector maps and dispatch field personnel.
2. **Forest Rangers & Firefighters:** Who need precise GPS sector coordinates, fire intensity data (smoke density, temperature), and flame verification before entering a hazardous zone.
3. **Environmental Scientists:** Who export historical environmental telemetry (CSV/JSON) to study micro-climatic fire trends and fuel aridity."

---

### Q6: What are the main features actually implemented in the system?
**Answer:**  
"The implemented system features:
1. **Multi-Modal Edge Detection:** DHT22 (temp/humidity), MQ-2 (smoke ADC), and optical IR photodiode (flame ADC).
2. **LoRa 433 MHz Telemetry:** SX1278 transceiver integration with RSSI and SNR signal quality logging.
3. **Fault-Tolerant Serial Bridge (`bridge.py`):** Python middleware featuring auto COM port discovery, physical boundary verification, duplicate packet suppression, and exponential retry backoff.
4. **PL/pgSQL Trigger Engine:** Database-level automatic alert dispatching, duplicate deduplication, and device status management.
5. **Tactical GIS Map:** Real-time Leaflet map with status-coded markers, pulsing threat radii, and interactive telemetry modals.
6. **Live Multi-Metric Analytics:** Time-series visualizers with metric toggling and CSV/JSON data export.
7. **Synthesized Acoustic Siren:** Zero-asset browser-native Web Audio API dual-tone siren for critical emergencies.
8. **Device & Schema Inspector:** Comprehensive device inventory and live SQL schema viewer."

---

### Q7: What are the primary limitations of the current implementation?
**Answer:**  
"We identify three clear technical boundaries:
1. **Star Topology:** Node-to-gateway communication is currently single-hop star topology rather than dynamic multi-hop mesh routing.
2. **Fixed Database GPS:** Sensor geographic coordinates are assigned in the database registry during deployment rather than read from an onboard hardware GPS module (to save battery power and BOM cost).
3. **Continuous Power Draw:** The ESP32 currently operates in active polling mode without deep-sleep cycling, requiring external USB/battery power."

---

## 2. System Workflow & Data Journey

### Q8: Explain the complete end-to-end data flow from the moment a fire ignites to when the operator hears the alarm.
**Answer:**  
"The data flow follows six precise stages:
1. **Physical Transduction:** The flame emits 760nm–1100nm infrared radiation, causing the photodiode voltage on GPIO 35 to drop below $100\text{ ADC}$, while the MQ-2 sensor detects smoke particulates, raising GPIO 34 above $2000\text{ ADC}$.
2. **Edge RF Transmission:** Within 2000ms, the ESP32 node serializes this data into a JSON packet and broadcasts it over 433.0 MHz LoRa (SF7, BW 125kHz) using the SX1278 module.
3. **Gateway Demodulation & Serial Output:** The ESP32 Base Station receives the RF packet, extracts RSSI (e.g., $-88\text{ dBm}$) and SNR (e.g., $+7.5\text{ dB}$), and transmits a formatted `GATEWAY_PACKET:` string across USB UART at 115200 baud.
4. **Bridge Ingestion & Validation:** `bridge.py` reads the UART line, strips the framing prefix, validates that values are physically plausible (e.g., $0 \le \text{ADC} \le 4095$), and executes an HTTPS POST to Supabase's REST endpoint.
5. **Database Trigger Execution:** PostgreSQL commits the row into `telemetry`. The `trg_evaluate_telemetry` trigger fires immediately, transitions the node status to `ALERT`, and creates a new `CRITICAL` ticket in `incidents`.
6. **Realtime WebSocket Broadcast & UI Alert:** Supabase's Realtime engine detects the PostgreSQL WAL insert and pushes the event over WebSockets (`wss://`) to the Next.js `useRealtimeDashboard` hook. The UI updates the GIS map marker to a pulsing red indicator, displays the incident in the active rail, and triggers the Web Audio API siren."
