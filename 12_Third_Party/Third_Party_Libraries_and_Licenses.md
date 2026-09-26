# Third-Party Libraries, Dependencies, and Open Source Licenses

## 1. Embedded Firmware Libraries (Arduino / C++)

| Library | Author / Organization | License | Purpose in Project |
| :--- | :--- | :--- | :--- |
| **`LoRa` (v0.8.0)** | Sandeep Mistry | MIT | Semtech SX1276/77/78/79 LoRa transceiver driver via hardware SPI. |
| **`DHT sensor library` (v1.4.6)** | Adafruit | MIT | Aosong DHT22 / AM2302 temperature and humidity single-wire protocol reader. |
| **`Adafruit Unified Sensor` (v1.1.14)** | Adafruit | Apache 2.0 | Sensor abstraction layer for temperature and physical metric scaling. |

---

## 2. Python Serial Bridge Dependencies

| Package | Version Range | License | Purpose in Project |
| :--- | :--- | :--- | :--- |
| **`pyserial`** | `^3.5` | BSD-3-Clause | Multi-platform UART serial communication with ESP32 gateway over USB. |
| **`requests`** | `^2.31.0` | Apache 2.0 | HTTP REST client for direct Supabase and API Gateway telemetry forwarding. |
| **`python-dotenv`** | `^1.0.0` | BSD-3-Clause | Parses local `.env` configuration files for secret and port isolation. |

---

## 3. Web Workstation & API Dependencies (Next.js / TypeScript)

| Package | Version | License | Purpose in Project |
| :--- | :--- | :--- | :--- |
| **`next`** | `15.4.9` | MIT | React full-stack framework (App Router, Serverless API Routes, Edge builds). |
| **`react`** | `19.1.0` | MIT | Component-based UI library with Concurrent Mode rendering. |
| **`react-dom`** | `19.1.0` | MIT | React DOM renderer for modern web browsers. |
| **`@supabase/supabase-js`** | `^2.49.1` | MIT | Supabase client for PostgreSQL PostgREST queries and Realtime WebSockets. |
| **`tailwindcss`** | `^4.0.0` | MIT | Utility-first CSS engine for dark-mode tactical EOC workstation styling. |
| **`leaflet`** | `^1.9.4` | BSD-2-Clause | Lightweight interactive mapping engine for GIS sensor fleet visualization. |
| **`react-leaflet`** | `^5.0.0` | Hippocratic / MIT | React bindings and component wrappers for Leaflet map elements. |
| **`recharts`** | `^2.15.4` | MIT | Declarative React SVG charting library for environmental time-series graphs. |
| **`lucide-react`** | `^1.16.0` | ISC | Vector iconography for tactical buttons, alert badges, and telemetry indicators. |
| **`clsx` / `tailwind-merge`** | `^2.1.1` / `^3.0.2` | MIT | Conditional CSS class name concatenation and conflict resolution. |

---

## 4. Cloud Platforms & Open Data Providers

| Service / Platform | Provider | License / Terms | Role in Platform |
| :--- | :--- | :--- | :--- |
| **PostgreSQL 15** | PostgreSQL Global Development Group | PostgreSQL License | Primary relational database engine. |
| **Supabase Cloud** | Supabase Inc. | Apache 2.0 / Commercial | Managed PostgreSQL hosting, Auth, PostgREST, and Realtime WAL engine. |
| **Vercel Edge Network** | Vercel Inc. | Commercial SaaS | Production serverless hosting for Next.js frontend and API routes. |
| **OpenStreetMap** | OpenStreetMap Foundation | ODbL (Open Database License) | Cartographic base map tiles rendered inside Leaflet GIS map. |
