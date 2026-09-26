# Technical Documentation & Architecture Manual

## 1. System Architecture Overview

The Wildfire Operations Center (EOC) platform is engineered as a decoupled, multi-tier Edge-to-Cloud telemetry and situational awareness system. It consists of four distinct operational layers:
1. **Edge Sensing Layer:** ESP32 microcontrollers polling physical transducers and modulating RF data.
2. **Sub-GHz RF & Ingestion Layer:** 433 MHz LoRa transceiver base stations coupled with a host-level Python hardware bridge.
3. **Cloud Database & Reactive Rules Engine:** Supabase PostgreSQL 15 instance executing stored procedures, database triggers, and change-data-capture (CDC) streaming.
4. **Presentation & Command Layer:** Next.js 15 single-page tactical operations workstation.

```mermaid
graph TD
    subgraph Layer1_Edge [Layer 1: Edge Sensing Layer]
        DHT[DHT22: Temp/Humidity\nGPIO 27] --> ESP_NODE[ESP32 Edge Node MCU\n240MHz Xtensa Dual-Core]
        MQ2[MQ-2: Smoke ADC\nGPIO 34] --> ESP_NODE
        FLAME[Flame IR Photodiode\nGPIO 35] --> ESP_NODE
        ESP_NODE -->|SPI Bus: 18,19,23,5| LORA_TX[SX1278 Ra-02 LoRa\n433.0 MHz / SF7 / BW125]
    end

    subgraph Layer2_Transport [Layer 2: Transport & Bridge Layer]
        LORA_TX -->|433 MHz RF Broadcast| LORA_RX[SX1278 Ra-02 LoRa\nBase Station Receiver]
        LORA_RX -->|SPI Bus: 18,19,23,5| ESP_GW[ESP32 Gateway MCU]
        ESP_GW -->|UART USB @ 115200 Baud\nGATEWAY_PACKET:...| PY_BRIDGE[Python Serial Bridge\nbridge.py]
        PY_BRIDGE -->|Physical Boundary Checks\n-50C to 100C / 0-4095 ADC| PY_VAL[Validation Engine]
    end

    subgraph Layer3_Cloud [Layer 3: Cloud Database & Triggers]
        PY_VAL -->|HTTPS REST POST\nSUPABASE_DIRECT Mode| DB_TEL[(telemetry Table\nPrimary Ingest)]
        DB_TEL -->|BEFORE INSERT Trigger| TRG_GW[trg_verify_telemetry_gateway\nValidates Node-GW Relationship]
        DB_TEL -->|AFTER INSERT Trigger| TRG_ALERT[trg_evaluate_telemetry\nAuto Alerts & Status Updates]
        TRG_ALERT -->|Upsert Incident| DB_INC[(incidents Table)]
        TRG_ALERT -->|Update Status| DB_NODES[(sensor_nodes Table)]
        DB_TEL -->|PostgreSQL WAL| CDC[Supabase Realtime\nWebSocket Engine]
    end

    subgraph Layer4_UI [Layer 4: Tactical EOC Workstation]
        CDC -->|wss:// Channels: telemetry, incidents| HOOK[useRealtimeDashboard Hook]
        HOOK --> REACT_STATE[React 19 State Container]
        REACT_STATE --> MAP_COMP[Leaflet GIS Tactical Map]
        REACT_STATE --> CHART_COMP[Recharts Multi-Metric Visualizer]
        REACT_STATE --> AUDIO_ALARM[Web Audio API Siren Synth]
        REACT_STATE --> INCIDENT_RAIL[Active Incident Rail]
    end
```

---

## 2. Hardware & Embedded Firmware Architecture

### 2.1 Sensor Node Hardware Specifications & Pinout (`firmware/sensor_node/sensor_node.ino`)
The edge node operates on an **ESP32 DevKit V1 (30-pin)** board. Sensors sample the ambient environment every 2000 milliseconds:
* **DHT22 (AM2302) Temperature & Humidity Sensor:**
  * Power: 3.3V / GND
  * Signal Pin: **GPIO 27** (Digital bidirectional single-bus with internal 10k pull-up).
  * Measurement Range: $-40^\circ\text{C}$ to $+80^\circ\text{C}$ ($\pm 0.5^\circ\text{C}$ accuracy), $0\text{--}100\%$ RH ($\pm 2\%$ accuracy).
* **MQ-2 Smoke & Combustible Gas Sensor:**
  * Heater Power: 5.0V (VBUS / External rail), GND.
  * Analog Output Pin: **GPIO 34** (Input-only ADC1 channel 6).
  * Conversion: 12-bit Successive Approximation Register (SAR) ADC ($0\text{--}4095$ range).
* **Optical Infrared Flame Sensor:**
  * Power: 3.3V / GND.
  * Analog Photodiode Output: **GPIO 35** (Input-only ADC1 channel 7).
  * Characteristics: Inverted analog logic ($4095$ represents ambient dark infrared; values drop towards $0\text{--}500$ upon detecting direct 760nm–1100nm infrared hydrocarbon flame radiation).
* **Ai-Thinker Ra-02 (Semtech SX1278) LoRa Radio Module:**
  * Interface: Hardware SPI (Standard VSPI pins).
  * Chip Select (`NSS`): **GPIO 5**
  * Reset (`RST`): **GPIO 14**
  * Interrupt (`DIO0`): **GPIO 26**
  * Clock (`SCK`): **GPIO 18**
  * Master In Slave Out (`MISO`): **GPIO 19**
  * Master Out Slave In (`MOSI`): **GPIO 23**

```
+-------------------------------------------------------------+
|                     ESP32 DEVKIT V1                         |
|                                                             |
|   [GPIO 27] <---- Data ----- [DHT22 Temp & Humidity Sensor] |
|   [GPIO 34] <---- Analog --- [MQ-2 Gas & Smoke Sensor (5V)] |
|   [GPIO 35] <---- Analog --- [Optical Flame Photodiode]     |
|                                                             |
|   [GPIO  5] ---- NSS -----\                                 |
|   [GPIO 14] ---- RST ------\                                |
|   [GPIO 26] <--- DIO0 -----\  [Ai-Thinker Ra-02 SX1278]     |
|   [GPIO 18] ---- SCK ------/  (433.0 MHz LoRa Module)       |
|   [GPIO 19] <--- MISO ----/                                 |
|   [GPIO 23] ---- MOSI ---/                                  |
+-------------------------------------------------------------+
```

### 2.2 RF Modulation Parameters
* **Carrier Frequency:** `433.0 MHz`
* **Bandwidth (BW):** `125 kHz`
* **Spreading Factor (SF):** `7` (Configured for high data rate and low latency within canopy bounds; expandable up to SF12 for multi-kilometer line of sight).
* **Coding Rate (CR):** `4/5`
* **Sync Word:** `0x12` (Default private network sync word).
* **Preamble Length:** `8 symbols`
* **CRC:** Hardware CRC enabled on all packets.

### 2.3 Edge Packet Formatting
The edge node formats readings into a compact JSON object broadcasted over LoRa:
```json
{
  "node_id": "NODE-DEMO-01",
  "seq": 1024,
  "temp": 28.6,
  "hum": 72.3,
  "smoke": 1781,
  "flame": 4095,
  "alert": false
}
```

### 2.4 Base Station Gateway Firmware (`firmware/gateway/gateway.ino`)
The base station listens continuously for incoming LoRa packets. When a packet passes hardware CRC:
1. It reads the raw RF payload from the SX1278 FIFO buffer.
2. It queries the SX1278 internal registers for Physical Layer metrics:
   * **RSSI (Received Signal Strength Indicator):** Value in dBm (e.g., `-88 dBm`).
   * **SNR (Signal-to-Noise Ratio):** Value in dB (e.g., `+7.5 dB`).
3. It packages the raw JSON string with the physical metrics and outputs a framed string over USB UART at **115200 baud**:
```
GATEWAY_PACKET:{"gw_id":"GW-DEMO-01","rssi":-88,"snr":7.5,"payload":{"node_id":"NODE-DEMO-01","seq":1024,"temp":28.6,"hum":72.3,"smoke":1781,"flame":4095,"alert":false}}
```

---

## 3. Hardware Serial Bridge Architecture (`bridge/bridge.py`)

The bridge is a resilient Python 3 daemon designed to bridge physical UART serial data into cloud web infrastructure.

```mermaid
stateDiagram-v2
    [*] --> SCANNING: Startup
    SCANNING --> OPENING: Identify USB COM Port (Silicon Labs / CH340)
    OPENING --> LISTENING: Baud 115200 Configured
    OPENING --> SCANNING: Port Open Failure (Backoff 2s)

    state LISTENING {
        [*] --> READ_LINE: serial.readline()
        READ_LINE --> FRAME_CHECK: Prefix == 'GATEWAY_PACKET:'?
        FRAME_CHECK --> PARSE_JSON: Strip prefix & json.loads()
        FRAME_CHECK --> READ_LINE: Ignore debug logs
        PARSE_JSON --> VALIDATE_PHYSICAL: Physical Boundary Checks
        VALIDATE_PHYSICAL --> POST_CLOUD: Boundary Valid
        VALIDATE_PHYSICAL --> DROP_PACKET: Out of bounds
        POST_CLOUD --> HTTP_SUCCESS: 200/201 Created
        POST_CLOUD --> HTTP_DUPLICATE: 409 Conflict (Ignored)
        POST_CLOUD --> HTTP_RETRY: 5xx / Network Error (Exp Backoff)
        HTTP_SUCCESS --> READ_LINE
        HTTP_DUPLICATE --> READ_LINE
        HTTP_RETRY --> READ_LINE
    }

    LISTENING --> SCANNING: SerialException (Unplugged)
```

### 3.1 Validation Engine Boundaries
To prevent garbled memory, cosmic rays, or floating ADC pins from poisoning cloud analytics, `bridge.py` validates every field against hard physical boundaries before transmission:
* Temperature: $-50.0^\circ\text{C} \le T \le +100.0^\circ\text{C}$
* Humidity: $0.0\% \le H \le 100.0\%$
* Smoke ADC: $0 \le \text{ADC} \le 4095$
* Flame ADC: $0 \le \text{ADC} \le 4095$
* RSSI: $-150 \le \text{RSSI} \le 0\text{ dBm}$
* SNR: $-30.0 \le \text{SNR} \le +30.0\text{ dB}$

### 3.2 Ingestion Modes
1. **`SUPABASE_DIRECT` (Default Production Mode):**
   * Transmits directly to `https://<supabase_id>.supabase.co/rest/v1/telemetry`.
   * Authenticates using the Supabase Service Role Key (`apikey` and `Authorization: Bearer` headers).
   * Benefits: Zero intermediary API latency, sub-50ms cloud ingest times.
2. **`API_GATEWAY` Mode:**
   * Transmits to Next.js API route `POST http://localhost:3000/api/telemetry`.
   * Authenticates using a pre-shared bearer token (`TELEMETRY_INGEST_KEY`).

---

## 4. Cloud Database & Backend Stored Procedures (`lib/schema.sql`)

The database is built on PostgreSQL 15 and leverages native relational constraints and PL/pgSQL triggers to handle edge ingestion logic inside the database engine.

```mermaid
erDiagram
    gateways ||--o{ sensor_nodes : "monitors"
    gateways ||--o{ telemetry : "receives"
    sensor_nodes ||--o{ telemetry : "originates"
    sensor_nodes ||--o{ incidents : "triggers"

    gateways {
        varchar(64) gateway_id PK
        varchar(128) name
        decimal latitude
        decimal longitude
        varchar(20) status
        timestamptz last_seen_at
    }

    sensor_nodes {
        varchar(64) node_id PK
        varchar(64) gateway_id FK
        varchar(128) name
        decimal latitude
        decimal longitude
        varchar(20) status
        timestamptz last_telemetry_at
        float battery_level
    }

    telemetry {
        bigserial id PK
        varchar(64) node_id FK
        varchar(64) gateway_id FK
        bigint sequence_number
        float temperature_c
        float humidity_pct
        int smoke_raw
        int flame_raw
        boolean alert_triggered
        int rssi_dbm
        float snr_db
        timestamptz created_at
    }

    incidents {
        bigserial id PK
        varchar(64) node_id FK
        varchar(32) alert_type
        varchar(20) severity
        varchar(20) status
        jsonb metadata
        timestamptz triggered_at
        timestamptz resolved_at
    }
```

### 4.1 Database Trigger Logic

#### A. Gateway-Node Association Trigger (`trg_verify_telemetry_gateway`)
Before any telemetry row is committed, this `BEFORE INSERT` trigger ensures that the `gateway_id` transmitting the telemetry is valid and updates the gateway's `last_seen_at` timestamp.

#### B. Automated Incident Evaluation Trigger (`trg_evaluate_telemetry`)
This `AFTER INSERT` trigger is the core autonomic alert engine:
1. **Status Evaluation:**
   * If `flame_raw < 100` (Infrared flame detected) OR `smoke_raw > 2000` (Dense smoke detected), the node is flagged in emergency condition.
   * If emergency condition is true, `sensor_nodes.status` is updated to `'ALERT'`.
   * If values return to normal ($T < 45^\circ\text{C}$, $\text{Smoke} \le 1500$, $\text{Flame} \ge 1000$), status transitions back to `'ONLINE'`.
2. **Incident Debouncing:**
   * Checks for any existing `'ACTIVE'` incident on the same `node_id`.
   * If no active incident exists, it inserts a new incident record with severity `'CRITICAL'`, capturing the trigger metrics in `metadata`.
   * If an active incident already exists, it avoids duplicate ticketing while updating incident logs.

---

## 5. Tactical EOC Frontend Architecture (Next.js 15 & React 19)

### 5.1 Component Hierarchy

```
app/layout.tsx (Theme Provider, Global CSS)
└── app/page.tsx (Workstation Orchestrator)
    ├── components/Header.tsx (Status Badges, Connection State, Workspace Switcher, Mute Toggle)
    ├── View: 'operations' (Tactical Operations Mode)
    │   ├── components/OperatorMap.tsx (Leaflet GIS Tactical Grid)
    │   │   ├── Dynamic Pulsing Map Markers
    │   │   ├── Coverage Radii Overlays
    │   │   └── Interactive Node Inspection Popups
    │   ├── components/ActiveIncidentsPanel.tsx (Real-time Incident Dispatch Rail)
    │   ├── components/DeviceHealthPanel.tsx (Collapsible Node/Gateway Quick Status)
    │   └── components/FleetHardware.tsx (Hardware Specifications & Quick Commands)
    ├── View: 'telemetry' (Data Analytics Mode)
    │   ├── components/TelemetryCharts.tsx (Recharts Multi-Metric Time Series)
    │   └── components/LiveTelemetryPanel.tsx (Streaming Ingestion Log & CSV/JSON Export)
    └── View: 'system' (Engineering & Diagnostic Mode)
        ├── components/DeviceManagement.tsx (Gateway/Node Registration & Firmware Specs)
        └── components/SchemaViewer.tsx (Live Database Schema & SQL Inspector)
```

### 5.2 Realtime Data Subscription Engine (`lib/hooks/useRealtimeDashboard.ts`)
The application subscribes to Supabase Realtime via WebSockets:
```typescript
const channel = supabase
  .channel('eoc_realtime_stream')
  .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'telemetry' }, (payload) => {
    handleIncomingTelemetry(payload.new as TelemetryRecord);
  })
  .on('postgres_changes', { event: '*', schema: 'public', table: 'incidents' }, (payload) => {
    handleIncidentUpdate(payload);
  })
  .on('postgres_changes', { event: 'UPDATE', schema: 'public', table: 'sensor_nodes' }, (payload) => {
    handleNodeStatusChange(payload.new as SensorNode);
  })
  .subscribe();
```

### 5.3 Zero-Dependency Browser Audio Synthesizer (`lib/audio.ts`)
Rather than relying on large MP3 audio assets that may fail to load or be blocked by cross-origin policies, the application utilizes the native browser **Web Audio API**:
* Creates an `AudioContext` and dual `OscillatorNode` instances.
* Synthesizes an emergency modulation: primary frequency $880\text{ Hz}$ modulating with an LFO at $440\text{ Hz}$ across square waveforms.
* Controls volume through a `GainNode` and safely shuts down upon operator acknowledgment or mute toggle.
