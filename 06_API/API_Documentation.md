# REST API Specification & Endpoint Documentation

## 1. Authentication & Security Model

The Wildfire Operations Center exposes both an **Application HTTP API** (Next.js App Router routes) and a **Direct Database REST API** (Supabase PostgREST).

### 1.1 Ingestion Authentication
* **Header Format:** `Authorization: Bearer <GATEWAY_API_KEY>` or `x-api-key: <GATEWAY_API_KEY>`.
* **Fail-Closed Policy:** If `GATEWAY_API_KEY` is not set or set to an insecure default placeholder, the server immediately returns `500 Server Misconfiguration` and refuses all writes.
* **Payload Size Limit:** Ingestion requests are strictly limited to **64 KB** (`Content-Length <= 65536 bytes`). Payloads exceeding this return `413 Payload Too Large`.

---

## 2. Telemetry Endpoints

### 2.1 Ingest Telemetry Packet
* **Method:** `POST`
* **Route:** `/api/telemetry`
* **Purpose:** Ingests live LoRa packets forwarded by the Python bridge or simulator.
* **Headers:**
  * `Content-Type: application/json`
  * `Authorization: Bearer <GATEWAY_API_KEY>`

**Request Body Example (Structured JSON):**
```json
{
  "gatewayId": "GW-DEMO-01",
  "nodeId": "NODE-DEMO-01",
  "packetSequence": 1025,
  "temperature": 29.4,
  "humidity": 68.2,
  "smokeAdc": 1820,
  "flameRaw": 4095,
  "flameDetected": false,
  "rssi": -85,
  "snr": 8.2,
  "batteryVoltage": 3.92
}
```

**Request Body Example (Raw Gateway Serial Framing):**
```json
{
  "rawPayload": "GATEWAY_PACKET:{\"gw_id\":\"GW-DEMO-01\",\"rssi\":-85,\"snr\":8.2,\"payload\":{\"node_id\":\"NODE-DEMO-01\",\"seq\":1025,\"temp\":29.4,\"hum\":68.2,\"smoke\":1820,\"flame\":4095,\"alert\":false}}"
}
```

**Success Response (HTTP 201 Created):**
```json
{
  "success": true,
  "reading": {
    "id": 14201,
    "nodeId": "NODE-DEMO-01",
    "gatewayId": "GW-DEMO-01",
    "packetSequence": 1025,
    "temperature": 29.4,
    "humidity": 68.2,
    "smokeAdc": 1820,
    "flameRaw": 4095,
    "flameDetected": false,
    "rssi": -85,
    "snr": 8.2,
    "timestamp": "2026-09-26T03:15:00.000Z"
  },
  "alertsTriggered": [],
  "timestamp": "2026-09-26T03:15:00.120Z"
}
```

**Duplicate Packet Response (HTTP 200 OK):**
```json
{
  "success": true,
  "duplicate": true,
  "message": "Duplicate packet acknowledged",
  "timestamp": "2026-09-26T03:15:00.120Z"
}
```

**Error Responses:**
* `400 Bad Request`: Malformed JSON or out-of-bounds physical sensor values.
* `401 Unauthorized`: Missing or invalid bearer token.
* `413 Payload Too Large`: Request body exceeds 64 KB.
* `500 Internal Server Error`: Database connection failure or unconfigured secret key.

---

### 2.2 Query Telemetry Readings
* **Method:** `GET`
* **Route:** `/api/telemetry`
* **Query Parameters:**
  * `nodeId` *(optional, string)*: Filter records by specific node ID.
  * `limit` *(optional, integer)*: Maximum number of rows to return (default: `80`, max: `1000`).

**Response Example (HTTP 200 OK):**
```json
{
  "readings": [
    {
      "id": 14201,
      "nodeId": "NODE-DEMO-01",
      "gatewayId": "GW-DEMO-01",
      "packetSequence": 1025,
      "temperature": 29.4,
      "humidity": 68.2,
      "smokeAdc": 1820,
      "flameRaw": 4095,
      "flameDetected": false,
      "rssi": -85,
      "snr": 8.2,
      "timestamp": "2026-09-26T03:15:00.000Z"
    }
  ],
  "totalBuffered": 14201,
  "packetCounter": 14201
}
```

---

## 3. Node Fleet Management Endpoints

### 3.1 List Sensor Nodes
* **Method:** `GET`
* **Route:** `/api/nodes`
* **Query Parameters:**
  * `gateway_id` *(optional, string)*: Filter nodes assigned to a specific base station.

**Response Example (HTTP 200 OK):**
```json
{
  "nodes": [
    {
      "node_id": "NODE-DEMO-01",
      "gateway_id": "GW-DEMO-01",
      "name": "North Pine Sector Alpha",
      "sector": "Sector Alpha",
      "latitude": 37.7749,
      "longitude": -122.4194,
      "elevation": 180,
      "status": "ONLINE",
      "battery_level": 94.5,
      "last_telemetry_at": "2026-09-26T03:15:00.000Z"
    }
  ],
  "stats": {
    "total": 4,
    "online": 3,
    "alert": 1,
    "warning": 0,
    "offline": 0
  },
  "timestamp": "2026-09-26T03:15:00.000Z"
}
```

---

### 3.2 Register New Sensor Node
* **Method:** `POST`
* **Route:** `/api/nodes`
* **Request Body:**
```json
{
  "node_id": "NODE-FOREST-02",
  "name": "Ridge Sector Bravo",
  "gateway_id": "GW-DEMO-01",
  "sector": "Sector Bravo",
  "latitude": 37.7785,
  "longitude": -122.4140,
  "elevation": 240,
  "status": "REGISTERED"
}
```
* **Success Response:** `HTTP 201 Created` with created node object.

---

### 3.3 Update Sensor Node
* **Method:** `PATCH`
* **Route:** `/api/nodes`
* **Request Body:**
```json
{
  "node_id": "NODE-FOREST-02",
  "name": "Ridge Sector Bravo (Renamed)",
  "latitude": 37.7790,
  "longitude": -122.4150
}
```
* **Success Response:** `HTTP 200 OK`.

---

### 3.4 Delete / Decommission Sensor Node
* **Method:** `DELETE`
* **Route:** `/api/nodes?node_id=NODE-FOREST-02`
* **Success Response (HTTP 200 OK):**
```json
{
  "success": true,
  "action": "deleted",
  "message": "Node NODE-FOREST-02 removed from fleet registry."
}
```

---

## 4. Incidents & Alert Management Endpoints

### 4.1 Get Active Incidents
* **Method:** `GET`
* **Route:** `/api/alerts`

**Response Example (HTTP 200 OK):**
```json
{
  "alerts": [
    {
      "id": 802,
      "nodeId": "NODE-DEMO-01",
      "type": "WILDFIRE_FLAME",
      "severity": "CRITICAL",
      "title": "Active Wildfire Flame Detected",
      "status": "ACTIVE",
      "metadata": {
        "flame_raw": 45,
        "smoke_raw": 2840,
        "temperature_c": 64.2
      },
      "triggeredAt": "2026-09-26T03:10:00.000Z",
      "acknowledgedAt": null,
      "resolvedAt": null
    }
  ],
  "activeCount": 1,
  "criticalCount": 1
}
```

---

### 4.2 Acknowledge or Resolve Incident
* **Method:** `POST`
* **Route:** `/api/alerts`
* **Request Body:**
```json
{
  "alertId": 802,
  "action": "acknowledge",
  "operatorName": "Dispatcher Johnson",
  "operatorNotes": "Ranger Unit 4 dispatched to Sector Alpha."
}
```

**Response Example (HTTP 200 OK):**
```json
{
  "success": true,
  "alert": {
    "id": 802,
    "status": "ACKNOWLEDGED",
    "acknowledgedAt": "2026-09-26T03:12:30.000Z"
  }
}
```

---

## 5. Data Export Endpoint

### 5.1 Export Telemetry
* **Method:** `GET`
* **Route:** `/api/export`
* **Query Parameters:**
  * `format` *(string)*: `csv` (default) or `json`.
  * `limit` *(integer)*: Row limit (default: `5000`).

**CSV Response Headers:**
* `Content-Type: text/csv`
* `Content-Disposition: attachment; filename="wildfire_telemetry_<timestamp>.csv"`

---

## 6. Direct Supabase Ingestion (PostgREST)

Used in `bridge.py` under `SUPABASE_DIRECT` mode:
* **Method:** `POST`
* **Route:** `https://<project-ref>.supabase.co/rest/v1/telemetry`
* **Headers:**
  * `apikey: <SUPABASE_SERVICE_ROLE_KEY>`
  * `Authorization: Bearer <SUPABASE_SERVICE_ROLE_KEY>`
  * `Content-Type: application/json`
  * `Prefer: return=minimal`
* **Payload:**
```json
{
  "node_id": "NODE-DEMO-01",
  "gateway_id": "GW-DEMO-01",
  "sequence_number": 1025,
  "temperature_c": 28.6,
  "humidity_pct": 72.3,
  "smoke_raw": 1781,
  "flame_raw": 4095,
  "alert_triggered": false,
  "rssi_dbm": -88,
  "snr_db": 7.5
}
```
* **Status Codes:**
  * `201 Created`: Telemetry successfully inserted.
  * `409 Conflict`: Duplicate sequence packet ignored by database uniqueness constraint.
