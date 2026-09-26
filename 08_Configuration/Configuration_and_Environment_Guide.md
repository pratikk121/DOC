# Configuration & Environment Variables Reference Guide

## 1. Overview of Environment Configuration

The Wildfire Operations Center system utilizes environment variable isolation across two distinct application boundaries:
1. **Next.js Web Workstation:** Configured via `.env.local` (local) or Vercel Environment Variables (cloud).
2. **Python Hardware Serial Bridge:** Configured via `bridge/.env` on the host machine connected to the gateway.

> [!WARNING]
> **Secret Key Protection:** Never commit `.env` or `.env.local` files to public version control repositories. Use `.env.example` templates and populate secrets only on deployment servers or local environments.

---

## 2. Web Frontend & API Environment Variables (`.env.local`)

### `NEXT_PUBLIC_SUPABASE_URL`
* **Purpose:** The HTTPS REST and WebSocket URL for connecting to the Supabase backend project.
* **Required:** Yes
* **Example:** `https://xyzprojectid.supabase.co`
* **Where Used:** `lib/supabase/client.ts`, `lib/supabase/server.ts`, `lib/hooks/useRealtimeDashboard.ts`
* **Security Sensitivity:** **Public** (Exposed to client-side browser bundle).

---

### `NEXT_PUBLIC_SUPABASE_ANON_KEY`
* **Purpose:** The public anonymous JWT token used by the browser client to subscribe to Realtime WebSocket channels and query permitted public tables under Row Level Security.
* **Required:** Yes
* **Example:** `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Inh5...`
* **Where Used:** `lib/supabase/client.ts`, `lib/hooks/useRealtimeDashboard.ts`
* **Security Sensitivity:** **Public** (Exposed to client-side browser bundle; restricted by PostgreSQL RLS policies).

---

### `SUPABASE_SERVICE_ROLE_KEY`
* **Purpose:** Administrative PostgreSQL service role secret key. Bypasses Row Level Security (RLS) to perform server-side writes, database migrations, and administrative fleet updates.
* **Required:** Yes
* **Example:** `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Inh5...` (Service Role JWT)
* **Where Used:** `lib/supabase/server.ts`, `app/api/telemetry/route.ts`, `app/api/nodes/route.ts`, `app/api/alerts/route.ts`
* **Security Sensitivity:** **Strict Secret / Critical** (Must never be prefixed with `NEXT_PUBLIC_` or leaked to client browser).

---

### `GATEWAY_API_KEY`
* **Purpose:** Pre-shared bearer token used by the Python Bridge or external edge devices to authenticate requests sent to `/api/telemetry` when running under `API_GATEWAY` mode.
* **Required:** Yes (if using API Gateway mode)
* **Example:** `wfc_telemetry_secure_key_778899`
* **Where Used:** `app/api/telemetry/route.ts`
* **Security Sensitivity:** **Secret** (Known only to server and field gateways).

---

## 3. Python Serial Bridge Environment Variables (`bridge/.env`)

### `SERIAL_PORT`
* **Purpose:** Operating system COM port or device node where the ESP32 Gateway is connected.
* **Required:** No (If omitted, `bridge.py` automatically scans available USB-UART devices).
* **Example:** `COM4` (Windows) or `/dev/ttyUSB0` (Linux/Raspberry Pi).
* **Where Used:** `bridge/bridge.py`
* **Security Sensitivity:** **None** (Hardware path).

---

### `BAUD_RATE`
* **Purpose:** UART baud rate for serial communication with the ESP32 gateway.
* **Required:** No (Default: `115200`).
* **Example:** `115200`
* **Where Used:** `bridge/bridge.py`
* **Security Sensitivity:** **None**.

---

### `INGESTION_MODE`
* **Purpose:** Selects cloud ingestion pathway.
  * `SUPABASE_DIRECT`: Bridges serial directly to Supabase REST endpoint (`/rest/v1/telemetry`).
  * `API_GATEWAY`: Bridges serial to Next.js API route (`/api/telemetry`).
* **Required:** No (Default: `SUPABASE_DIRECT`).
* **Example:** `SUPABASE_DIRECT`
* **Where Used:** `bridge/bridge.py`
* **Security Sensitivity:** **None**.

---

### `SUPABASE_URL`
* **Purpose:** Target Supabase project endpoint for direct database insertion.
* **Required:** Yes (When `INGESTION_MODE=SUPABASE_DIRECT`).
* **Example:** `https://xyzprojectid.supabase.co`
* **Where Used:** `bridge/bridge.py`
* **Security Sensitivity:** **Public**.

---

### `SUPABASE_KEY`
* **Purpose:** Supabase service role secret key used by `bridge.py` to authenticate direct REST writes.
* **Required:** Yes (When `INGESTION_MODE=SUPABASE_DIRECT`).
* **Example:** `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`
* **Where Used:** `bridge/bridge.py`
* **Security Sensitivity:** **Strict Secret / Critical**.

---

### `LOG_LEVEL`
* **Purpose:** Controls verbosity of console output (`DEBUG`, `INFO`, `WARNING`, `ERROR`).
* **Required:** No (Default: `INFO`).
* **Example:** `INFO`
* **Where Used:** `bridge/bridge.py`
* **Security Sensitivity:** **None**.
