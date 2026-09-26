# Wildfire Operations Center (EOC) — Complete Project Handover Package

[![Next.js 15](https://img.shields.io/badge/Next.js-15.4-black?logo=next.js)](https://nextjs.org/)
[![React 19](https://img.shields.io/badge/React-19.1-blue?logo=react)](https://react.dev/)
[![Supabase PostgreSQL](https://img.shields.io/badge/Supabase-PostgreSQL%2015-emerald?logo=supabase)](https://supabase.com/)
[![ESP32 LoRa 433MHz](https://img.shields.io/badge/Hardware-ESP32%20%2B%20SX1278-orange)](https://espressif.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **Handover Recipient:** Final-Year B.Tech Candidate (Computer Science / ECE / IT)  
> **Project Scope:** Edge-to-Cloud IoT Early Wildfire Detection & Tactical Operations Center Platform  
> **Production Deployment:** [https://dashboard-mu-azure-bp2m313edn.vercel.app/](https://dashboard-mu-azure-bp2m313edn.vercel.app/)

---

## 🚀 Quick Start in 3 Minutes

### 1. Run the Web Dashboard
```bash
# Install dependencies
bun install   # or: npm install

# Start local Next.js development server
bun dev       # or: npm run dev
# Open http://localhost:3000
```

### 2. Run the Hardware Serial Bridge (Python)
```bash
cd bridge
pip install -r requirements.txt
python bridge.py
```

### 3. Run Automated Test Suites
```bash
# Run Frontend / API tests (37 passing tests)
bun test

# Run Python Bridge tests (30 passing tests)
python bridge/test_bridge.py
```

---

## 📚 Complete Handover Documentation Structure

| Document Path | Description |
| :--- | :--- |
| **[`01_Project_Overview/Project_Overview.md`](./01_Project_Overview/Project_Overview.html)** | Executive summary, problem definition, objectives, architecture, and scope. |
| **[`02_Technical_Documentation/Technical_Documentation.md`](./02_Technical_Documentation/Technical_Documentation.html)** | Hardware schematics, GPIO pinouts, LoRa RF framing, Python bridge, and triggers. |
| **[`03_User_Manual/User_Manual.md`](./03_User_Manual/User_Manual.html)** | Tactical operations manual, GIS map navigation, incident workflows, and audio alerts. |
| **[`04_Installation/Installation_and_Setup_Guide.md`](./04_Installation/Installation_and_Setup_Guide.html)** | Complete guide to flashing ESP32 firmware, configuring Supabase, and running the bridge. |
| **[`05_Database/Database_Documentation.md`](./05_Database/Database_Documentation.html)** | PostgreSQL 15 schema, ER diagrams, PL/pgSQL stored procedures, and indexes. |
| **[`06_API/API_Documentation.md`](./06_API/API_Documentation.html)** | Full REST API specification with endpoints, request/response bodies, and status codes. |
| **[`07_Deployment/Deployment_Guide.md`](./07_Deployment/Deployment_Guide.html)** | Vercel production deployment, Supabase Realtime setup, and Raspberry Pi daemons. |
| **[`08_Configuration/Configuration_and_Environment_Guide.md`](./08_Configuration/Configuration_and_Environment_Guide.html)** | Reference for all environment variables (`.env.local` and `bridge/.env`). |
| **[`09_Known_Issues/Known_Issues_and_Limitations.md`](./09_Known_Issues/Known_Issues_and_Limitations.html)** | Transparent documentation of system limitations, hardware quirks, and future roadmap. |
| **[`10_Viva/`](./10_Viva/Viva_Questions_and_Answers.html)** | **7 Dedicated Viva & Panel Defense Guides** (Core Q&A, Technical, Architecture, Database, Tech Stack, Security, and 20+ Tough Cross-Examinations). |
| **[`11_Handover/Project_Handover_and_Acceptance.md`](./11_Handover/Project_Handover_and_Acceptance.html)** | Formal deliverables verification matrix and supervisor sign-off sheet. |
| **[`12_Third_Party/Third_Party_Libraries_and_Licenses.md`](./12_Third_Party/Third_Party_Libraries_and_Licenses.html)** | Complete open-source license and third-party dependency directory. |
| **[`Documentation_Index.md`](./Documentation_Index.html)** | Quick-link master navigation index. |

---

## 🛠️ System Architecture at a Glance

```mermaid
flowchart TD
    subgraph Edge [Edge Sensing & LoRa Broadcast]
        SENSORS["DHT22 + MQ-2 + Flame IR"] --> ESP_NODE["ESP32 Edge Node"]
        ESP_NODE -->|433 MHz LoRa Radio| ESP_GW["ESP32 Base Station Gateway"]
    end

    subgraph Bridge [Host Middleware]
        ESP_GW -->|USB Serial @ 115200 Baud| PY_BRIDGE["Python Bridge (bridge.py)"]
        PY_BRIDGE -->|Physical Boundary Checks| VALIDATOR{"Valid?"}
    end

    subgraph Cloud [Cloud Persistence & Autonomics]
        VALIDATOR -->|HTTPS REST POST| SUPA_DB[("Supabase PostgreSQL 15")]
        SUPA_DB -->|PL/pgSQL Trigger| TRIGGER["trg_evaluate_telemetry\nAuto Alert & Status Engine"]
        SUPA_DB -->|PostgreSQL WAL| REALTIME["Supabase Realtime (WebSockets)"]
    end

    subgraph Presentation [Tactical EOC Workstation]
        REALTIME -->|wss:// stream| NEXT_APP["Next.js 15 / React 19 Workstation"]
        NEXT_APP --> MAP["Leaflet Tactical GIS Map"]
        NEXT_APP --> CHARTS["Recharts Multi-Metric Visualizer"]
        NEXT_APP --> AUDIO["Web Audio API Siren Synth"]
    end
```
