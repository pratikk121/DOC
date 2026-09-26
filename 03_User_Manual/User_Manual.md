# User Manual: Wildfire Tactical Operations Center

## 1. Introduction & Workstation Purpose
The **Wildfire Emergency Operations Center (EOC) Dashboard** is a mission-critical web application designed for emergency dispatchers, forest rangers, and environmental monitoring teams. It provides real-time situational awareness, rapid wildfire alert triage, sensor fleet health monitoring, and environmental data analysis.

---

## 2. System Requirements & Access

### 2.1 Recommended Hardware & Browser Specifications
* **Display Resolution:** Minimum $1366 \times 768$ (Optimized for $1920 \times 1080$ Full HD or dual-monitor tactical desks).
* **Supported Browsers:** Google Chrome (v110+), Microsoft Edge (v110+), Mozilla Firefox (v115+), or Apple Safari (v16+).
* **Audio Capabilities:** Enabled speakers or headphones (required for acoustic siren alerts during critical wildfire emergencies).
* **Network Connectivity:** Stable broadband or 4G/5G mobile uplink with outbound WebSocket support (`wss://`).

### 2.2 Accessing the Live Dashboard
* **Production Cloud Workstation URL:** `https://dashboard-mu-azure-bp2m313edn.vercel.app/`
* **Local Development URL:** `http://localhost:3000/`

---

## 3. Workstation Navigation & Interface Tour

The EOC Workstation is organized into three primary operational workspaces accessible via the top navigation bar:
1. **Operations Workspace (Tactical Map & Alert Dispatch)**
2. **Telemetry Workspace (Multi-Metric Time-Series & Ingest Logs)**
3. **System Workspace (Device Management, Pinouts & Database Schema)**

```
+-----------------------------------------------------------------------------------------+
| [WILDFIRE EOC]  [LIVE WS CONNECTED]   [● Operations] [Telemetry] [System]   [🔊 Mute]   |
+-----------------------------------------------------------------------------------------+
|                                           |                                             |
|                                           |            ACTIVE INCIDENTS RAIL            |
|                                           |  [CRITICAL: Fire Detected on NODE-01]       |
|            TACTICAL GIS MAP               |  Trigger: Smoke 2840 ADC | Flame 45 ADC     |
|   - Node Markers (Green / Amber / Red)    |  [ACKNOWLEDGE]     [DISPATCH RANGERS]       |
|   - Pulsing Threat Radii                  |---------------------------------------------|
|   - Interactive Telemetry Popups          |             FLEET HEALTH DOCK               |
|                                           |  - NODE-DEMO-01: ONLINE (-88dBm, 28.6°C)    |
|                                           |  - GW-DEMO-01:   ACTIVE (COM4 Connected)    |
+-----------------------------------------------------------------------------------------+
```

---

## 4. Workspace Features & Capabilities

### 4.1 Header Bar & Global Controls
* **System Status Indicator:** Displays live WebSocket connection status (`CONNECTED` in emerald, `CONNECTING` in amber, `DISCONNECTED` in red).
* **Audio Alarm Toggle (`Mute / Unmute`):** Silences or enables the Web Audio API synthesizer siren during active fire alerts.
* **Active Alert Banner:** Flashes high-visibility crimson across the top bar whenever an unresolved wildfire incident exists in the system.

### 4.2 Operations Workspace
* **Tactical GIS Map (Leaflet):**
  * **Map Centering & Navigation:** Pan and zoom across the forest sector.
  * **Marker Color Semantics:**
    * 🟢 **Green (Online):** Node is healthy; all sensor readings are within baseline environmental parameters.
    * 🔴 **Red Pulse (Alert):** Active fire detected (Smoke $> 2000\text{ ADC}$ or Flame Photodiode $< 100\text{ ADC}$).
    * 🟡 **Amber (Warning):** Elevated thermal readings ($T > 45^\circ\text{C}$) or degraded RF signal quality ($\text{RSSI} < -115\text{ dBm}$).
    * ⚪ **Gray (Offline):** No telemetry received within the last 5 minutes.
  * **Node Inspection:** Clicking any map marker reveals an inspection modal showing live Temperature, Humidity, Smoke ADC, Flame ADC, RSSI, SNR, Sequence Number, and exact timestamp.
* **Active Incidents Rail:**
  * Displays real-time wildfire tickets created automatically by the database trigger engine.
  * Provides one-click action buttons: **Acknowledge**, **Deploy Field Team**, and **Resolve Incident**.
* **Fleet Health Dock:**
  * Lists all registered gateways and nodes with battery percentage, signal quality, and last-seen telemetry times.

### 4.3 Telemetry Workspace
* **Multi-Metric Time-Series Graphs (Recharts):**
  * Interactive line charts graphing:
    1. **Temperature ($^\circ\text{C}$):** Displays ambient heat trends with safe baseline thresholds.
    2. **Humidity ($\%$):** Tracks relative moisture drops.
    3. **Smoke Density (ADC Raw):** 12-bit analog gas concentration curve.
    4. **Flame IR Radiation (ADC Raw):** Photodiode voltage drop tracking.
    5. **RF Quality (RSSI dBm & SNR dB):** Wireless link performance analysis.
  * **Controls:** Toggle metrics on/off, filter by specific Node ID, and adjust the time window (Last 15m, 1h, 6h, 24h).
* **Live Ingestion Log:**
  * Real-time scrolling terminal display of every incoming packet passing the bridge.
* **Data Export Suite:**
  * **Export as CSV:** Generates structured spreadsheet data for environmental compliance reporting.
  * **Export as JSON:** Downloads raw telemetry payloads for machine learning research.

### 4.4 System Workspace
* **Device & Gateway Directory:** View hardware IDs, MAC addresses, GPS coordinates, and assigned base stations.
* **Hardware & Pinout Reference:** Embedded visual reference of ESP32 GPIO wiring for DHT22, MQ-2, Flame IR, and SX1278 LoRa.
* **Database Schema Inspector:** Live viewer displaying table definitions (`gateways`, `sensor_nodes`, `telemetry`, `incidents`) and active SQL triggers.

---

## 5. Step-by-Step Operator Workflows

### Workflow 1: Responding to a Live Wildfire Incident
1. **Acoustic & Visual Alert:**
   * The dashboard top banner turns flashing red, the map marker for the affected node pulses red, and the synthesizer siren sounds.
2. **Locate Threat on Tactical Map:**
   * Inspect the pulsing red node on the GIS map. Note the sector and GPS coordinates.
3. **Verify Multi-Sensor Telemetry:**
   * Click the node marker to open the inspection card.
   * Verify whether both Flame IR ($< 100$) and Smoke ($> 2000$) are elevated (confirms an active combustion event rather than sensor drift).
4. **Silence Siren:**
   * Click **Mute Siren** on the top header to acknowledge the audio notification while maintaining visual tracking.
5. **Dispatch Field Rangers:**
   * In the **Active Incidents Rail**, click **Acknowledge Incident**.
   * Note the GPS coordinates and relay them to ground firefighting units.
6. **Incident Resolution:**
   * Once field rangers confirm the fire is extinguished and sensor values return to normal, click **Resolve Incident** to archive the ticket.

---

### Workflow 2: Exporting Environmental Data for Investigation
1. Navigate to the **Telemetry** workspace using the top navigation bar.
2. In the **Node Filter** dropdown, select the target node (e.g., `NODE-DEMO-01`).
3. Select the desired historical time range (e.g., `Last 24 Hours`).
4. Click **Export CSV** to download `telemetry_export_<date>.csv` to your computer.

---

## 6. Operator Troubleshooting & Common Scenarios

| Issue Observed | Potential Cause | Recommended Operator Action |
| :--- | :--- | :--- |
| Top status shows **DISCONNECTED** (Red) | Lost internet connection or cloud WebSocket outage. | 1. Check local internet connectivity.<br>2. Refresh the browser tab (`F5`).<br>3. Verify Supabase project status. |
| Node marker displays **OFFLINE** (Gray) | ESP32 node battery depleted or out of LoRa range. | 1. Check gateway last-seen time in Fleet Dock.<br>2. Dispatch field technician to inspect node power and antenna. |
| Siren does not sound during active fire | Browser autoplay security policy blocked audio context. | Click anywhere on the dashboard interface or toggle the **Mute/Unmute** button once to grant browser audio permission. |
| Map tiles fail to load | OpenStreetMap tile server latency or DNS failure. | Verify outbound internet access on port 443; reload browser page. |
