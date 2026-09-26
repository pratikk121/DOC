# Known Issues, Limitations & Future Scope

## 1. System Nuances & Operational Behaviors

### 1.1 Browser Web Audio Autoplay Restrictions
* **Description:** Modern browsers (Chrome, Edge, Firefox, Safari) enforce security policies prohibiting web pages from playing audio until the user has performed at least one interactive gesture (click, tap, or keypress) on the document.
* **Impact on EOC Workstation:** If the dashboard opens and an active wildfire occurs before the operator clicks anywhere on the page, the Web Audio siren synthesizer will remain suspended.
* **Mitigation / Workaround:** The UI displays a persistent audio toggle button in the header. The operator can click the button once upon launching the station to grant full audio permissions.

### 1.2 MQ-2 Gas Sensor Pre-Heat Stabilization Delay
* **Description:** The MQ-2 semiconductor sensor uses an internal $\text{SnO}_2$ heating element that requires 2 to 3 minutes from power-on to reach thermal equilibrium ($350^\circ\text{C}$).
* **Impact on EOC Workstation:** During the first 120 seconds after powering on the ESP32 sensor node, analog readings on GPIO 34 may spike or fluctuate before settling into baseline ambient air values ($\sim 1600\text{--}1800\text{ ADC}$).
* **Mitigation / Workaround:** In production deployments, edge firmware should include a 120-second startup warm-up delay before arming the automated smoke alarm trigger.

---

## 2. Technical & Architecture Limitations

### 2.1 Single-Hop Star Topology vs. Multi-Hop Mesh
* **Current State:** The deployed firmware (`firmware/sensor_node/sensor_node.ino`) uses point-to-point unslotted LoRa broadcasts to the gateway.
* **Limitation:** Nodes must be within direct RF transmission range of the gateway (typically $1\text{--}5\text{ km}$ depending on terrain and foliage density).
* **Future Scope:** Implementation of a multi-hop routing protocol (e.g., Reticulum, ESP-MESH, or custom flooding algorithm as conceptualized in `docs/SWARMING_MESH_CONCEPT.md`) where intermediate nodes retransmit packets from deep-forest sensors.

### 2.2 Static Database Coordinates vs. Dynamic Hardware GPS
* **Current State:** Sensor node geographic positions (latitude, longitude, elevation) are stored in the database registry table `sensor_nodes` during deployment.
* **Limitation:** If a physical node is moved without updating the database, the GIS map will display its original location.
* **Rationale:** Fixed deployment is standard for stationary forest infrastructure, saving battery power, BOM costs, and eliminating GPS lock acquisition delays under dense tree canopies.

### 2.3 Dashboard Authentication & Access Control
* **Current State:** The frontend dashboard (`app/page.tsx`) provides open tactical viewing without an individual user login screen. Ingestion and database mutations are secured via API keys and service-role JWTs.
* **Limitation:** Any user with the dashboard URL can view the live tactical map.
* **Future Scope:** Implement Supabase Auth (Email/Password or OAuth) with Role-Based Access Control (RBAC) separating Dispatchers, Field Rangers, and System Administrators.

### 2.4 Power Management & Continuous RF Broadcasting
* **Current State:** The ESP32 node runs continuously in active mode, sampling sensors and transmitting every 2000ms.
* **Limitation:** Power consumption is approximately $80\text{--}150\text{ mA}$, draining a standard 18650 lithium battery within 24 to 36 hours without solar harvesting.
* **Future Scope:** Implement ESP32 Deep Sleep with dynamic wake-up cycles (e.g., sleep for 60 seconds during baseline conditions; wake up immediately if analog comparator detects smoke/flame surge).

---

## 3. External Service Dependencies

| External Dependency | Component Impacted | Risk & Mitigation |
| :--- | :--- | :--- |
| **Supabase Cloud (PostgreSQL & Realtime)** | Backend storage and live WebSocket telemetry streaming. | If Supabase undergoes maintenance, `bridge.py` buffers packets locally with exponential backoff and retries once connectivity is restored. |
| **Vercel Serverless Hosting** | Web application hosting and Next.js API routes. | Free-tier execution timeout is capped at 10 seconds; edge functions handle requests within $< 200\text{ ms}$. |
| **OpenStreetMap Tile CDN** | Leaflet tactical map base layer rendering. | If OpenStreetMap CDN experiences latency, offline tile caching or MapLibre vector fallback can be integrated. |

---

## 4. Summary of Future Enhancements

1. **Solar Energy Harvesting Unit:** Integration of a 5W monocrystalline solar panel, MPPT charge controller (TP4056 / CN3791), and LiFePO4 battery pack.
2. **LoRaWAN / TTN Compliance:** Transitioning from raw proprietary LoRa packets to standard LoRaWAN 1.0.4 protocols for integration with regional public gateway networks.
3. **On-Device Edge AI (TinyML):** Running lightweight TensorFlow Lite Micro models on the ESP32 to compute a localized Canadian Forest Fire Weather Index (FWI) using temperature, humidity, and atmospheric pressure.
4. **SMS / WhatsApp Emergency Dispatch:** Integrating Twilio or AWS SNS webhooks inside Supabase database triggers to dispatch immediate SMS alerts with GPS map links to ranger mobile phones.
