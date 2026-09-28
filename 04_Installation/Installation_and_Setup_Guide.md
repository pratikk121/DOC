# Installation, Flashing, and Environment Setup Guide

## 1. Prerequisites & Required Toolchains

Ensure your development workstation has the following software installed before proceeding:
* **Operating System:** Windows 10/11, macOS Sonoma/Ventura, or Ubuntu 22.04 LTS.
* **Node.js / JavaScript Runtime:** Node.js v18.17+ or **Bun v1.1+** (Recommended for ultra-fast package resolution).
* **Python Runtime:** Python 3.10, 3.11, or 3.12 (with `pip` and `venv`).
* **Embedded Toolchain:** Arduino IDE 2.3+ (with ESP32 Board Core v2.0.14+).
* **Cloud Database Account:** Free-tier Supabase account at [supabase.com](https://supabase.com).
* **Version Control:** Git 2.40+.

---

## 2. Hardware Bill of Materials (BOM) & Wiring Reference

### 2.1 Component Checklist
| Item | Component Description | Quantity | Purpose |
| :--- | :--- | :--- | :--- |
| 1 | ESP32 DevKit V1 (30-pin, CP2102/CH340) | 2 | 1x Edge Sensor Node, 1x Base Station Gateway |
| 2 | Ai-Thinker Ra-02 LoRa Module (SX1278, 433 MHz) | 2 | RF Transceiver for edge and gateway |
| 3 | DHT22 / AM2302 Sensor | 1 | High-precision temperature and humidity sensing |
| 4 | MQ-2 Combustible Gas & Smoke Sensor | 1 | Smoke and flammable hydrocarbon gas ADC |
| 5 | Optical Infrared Flame Photodiode Sensor | 1 | Direct infrared flame detection (760nm–1100nm) |
| 6 | 433 MHz Spring / Whip Antennas (or 17.3cm solid wire) | 2 | RF impedance matching (prevents PA damage) |
| 7 | Micro-USB Data Cables & Breadboards / Jumpers | — | Power and interconnects |

---

### 2.2 Edge Sensor Node Hardware Wiring Blueprint (`firmware/sensor_node/sensor_node.ino`)

```
                      +================================================+
                      |         ESP32 DEVKIT V1 (30-PIN MCU)           |
                      |                                                |
[ 5V POWER BANK ] ===>| [VIN (5V)] ───────┬────────────────────────────|───► [MQ-2 VCC (Heater 5V)]
                      | [3.3V OUT] ──┬────┼────────────────────────────|───► [DHT22 VCC (3.3V)]
                      |              │    │                            |───► [Flame IR VCC (3.3V)]
                      |              │    │                            |───► [SX1278 LoRa VCC (3.3V ONLY!)]
                      |              │    │                            |
                      | [GND] ───────┴────┴─── [COMMON GROUND BUS] ────|───► [All Sensor & LoRa GNDs]
                      |                                                |
                      | [GPIO 27] <── (Digital Single-Wire) ──────────|──── [DHT22 DATA]
                      | [GPIO 34] <── (Analog ADC1_CH6) ───────────────|──── [MQ-2 Analog A0]
                      | [GPIO 35] <── (Analog ADC1_CH7) ───────────────|──── [Flame Photodiode A0]
                      |                                                |
                      | [GPIO  5] ─── (SPI NSS / Chip Select) ─────────|───► [SX1278 NSS]
                      | [GPIO 14] ─── (Reset) ─────────────────────────|───► [SX1278 RST]
                      | [GPIO 26] <── (DIO0 / IRQ Interrupt) ──────────|──── [SX1278 DIO0]
                      | [GPIO 18] ─── (VSPI SCK Clock) ────────────────|───► [SX1278 SCK]
                      | [GPIO 19] <── (VSPI MISO Master In) ───────────|──── [SX1278 MISO]
                      | [GPIO 23] ─── (VSPI MOSI Master Out) ──────────|───► [SX1278 MOSI]
                      |                                                |
                      | [ANT PIN] ─────────────────────────────────────|───► [17.3cm 433MHz Antenna]
                      +================================================+
```

### 2.3 Edge Node Pin Interconnect Reference Table

```
+-------------------+--------------------+------------------------+
| Component         | Sensor / Module Pin| ESP32 DevKit V1 Pin    |
+-------------------+--------------------+------------------------+
| DHT22             | VCC                | 3.3V                   |
| DHT22             | GND                | GND                    |
| DHT22             | DATA (Out)         | GPIO 27                |
+-------------------+--------------------+------------------------+
| MQ-2 Gas Sensor   | VCC (Heater)       | 5V (VIN / External 5V) |
| MQ-2 Gas Sensor   | GND                | GND                    |
| MQ-2 Gas Sensor   | A0 (Analog Out)    | GPIO 34 (ADC1_CH6)     |
+-------------------+--------------------+------------------------+
| Optical Flame IR  | VCC                | 3.3V                   |
| Optical Flame IR  | GND                | GND                    |
| Optical Flame IR  | A0 (Analog Out)    | GPIO 35 (ADC1_CH7)     |
+-------------------+--------------------+------------------------+
| SX1278 Ra-02 LoRa | VCC (3.3V ONLY!)   | 3.3V                   |
| SX1278 Ra-02 LoRa | GND                | GND                    |
| SX1278 Ra-02 LoRa | NSS (CS)           | GPIO 5                 |
| SX1278 Ra-02 LoRa | RST                | GPIO 14                |
| SX1278 Ra-02 LoRa | DIO0 (IRQ)         | GPIO 26                |
| SX1278 Ra-02 LoRa | SCK                | GPIO 18 (VSPI SCK)     |
| SX1278 Ra-02 LoRa | MISO               | GPIO 19 (VSPI MISO)    |
| SX1278 Ra-02 LoRa | MOSI               | GPIO 23 (VSPI MOSI)    |
+-------------------+--------------------+------------------------+
```

> [!CAUTION]
> **SX1278 Voltage Level Warning:** Connect the Ra-02 LoRa module `VCC` **ONLY to 3.3V**. Connecting to 5V will permanently destroy the Semtech SX1278 silicon chip.
> **Antenna Requirement:** Never power on the LoRa module without an antenna attached. Operating without an antenna causes RF power reflection that can destroy the Power Amplifier (PA). For 433 MHz, an exact **17.3 cm** piece of wire soldered to the `ANT` pin serves as a tuned quarter-wave monopole antenna.

---

### 2.3 Base Station Gateway Wiring Table (`firmware/gateway/gateway.ino`)

```
+-------------------+--------------------+------------------------+
| Component         | Module Pin         | ESP32 DevKit V1 Pin    |
+-------------------+--------------------+------------------------+
| SX1278 Ra-02 LoRa | VCC                | 3.3V                   |
| SX1278 Ra-02 LoRa | GND                | GND                    |
| SX1278 Ra-02 LoRa | NSS (CS)           | GPIO 5                 |
| SX1278 Ra-02 LoRa | RST                | GPIO 14                |
| SX1278 Ra-02 LoRa | DIO0 (IRQ)         | GPIO 26                |
| SX1278 Ra-02 LoRa | SCK                | GPIO 18                |
| SX1278 Ra-02 LoRa | MISO               | GPIO 19                |
| SX1278 Ra-02 LoRa | MOSI               | GPIO 23                |
+-------------------+--------------------+------------------------+
```

---

## 3. Firmware Flashing Guide (Arduino IDE)

### 3.1 Setup Arduino IDE 2.x
1. Open Arduino IDE and open **File $\to$ Preferences**.
2. Add the following URL into **Additional Boards Manager URLs**:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
3. Navigate to **Tools $\to$ Board $\to$ Boards Manager**, search for `esp32` by Espressif Systems, and install version **2.0.14** (or latest 2.x).
4. Navigate to **Tools $\to$ Manage Libraries...** and install the following exact libraries:
   * **`LoRa`** by Sandeep Mistry (v0.8.0+)
   * **`DHT sensor library`** by Adafruit (v1.4.6+)
   * **`Adafruit Unified Sensor`** by Adafruit (v1.1.14+)

### 3.2 Flash Edge Sensor Node
1. Open `firmware/sensor_node/sensor_node.ino`.
2. Connect the Sensor Node ESP32 board to your computer via USB.
3. In Arduino IDE, configure:
   * **Board:** `DOIT ESP32 DEVKIT V1`
   * **Upload Speed:** `921600`
   * **CPU Frequency:** `240MHz (WiFi/BT)`
   * **Flash Frequency:** `80MHz`
   * **Port:** Select your board's COM port (e.g., `COM3`).
4. Click **Upload**. (If the terminal displays `Connecting........_____`, press and hold the physical **BOOT** button on the ESP32 board until uploading starts).

### 3.3 Flash Base Station Gateway
1. Open `firmware/gateway/gateway.ino`.
2. Connect the Gateway ESP32 board to your computer via USB.
3. Select the Gateway's COM port (e.g., `COM4`).
4. Click **Upload**.
5. Open Serial Monitor at **115200 baud** to verify the startup message:
   ```
   [GW] Initializing SX1278 LoRa receiver on 433.0 MHz...
   [GW] LoRa receiver initialized successfully. Listening...
   ```

---

## 4. Cloud Database Setup (Supabase)

1. Log in to [Supabase](https://supabase.com) and create a new project named `wildfire-eoc`.
2. In the Supabase dashboard, navigate to the **SQL Editor**.
3. Open `lib/schema.sql` from this repository, copy its entire contents, paste it into the Supabase SQL Editor, and click **Run**.
4. Confirm in the **Table Editor** that the 4 tables are created:
   * `gateways`
   * `sensor_nodes`
   * `telemetry`
   * `incidents`
5. Navigate to **Database $\to$ Replication** and ensure that **Realtime** is toggled **ON** for all 4 tables.
6. Navigate to **Project Settings $\to$ API** and copy:
   * **Project URL:** `https://<your-project-id>.supabase.co`
   * **Anon / Public Key:** `eyJhbGciOi...`
   * **Service Role Key (Secret):** `eyJhbGciOi...`

---

## 5. Hardware Serial Bridge Setup (`bridge/bridge.py`)

1. Open a terminal in the project root and navigate to the `bridge/` directory:
   ```bash
   cd bridge
   ```
2. Create and activate a Python virtual environment:
   ```bash
   # Windows PowerShell
   python -m venv venv
   .\venv\Scripts\Activate.ps1

   # Linux / macOS
   python3 -m venv venv
   source venv/bin/activate
   ```
3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
4. Create the configuration file `bridge/.env` based on `bridge/config.example.env`:
   ```env
   # Serial Port Configuration
   SERIAL_PORT=COM4
   BAUD_RATE=115200

   # Ingestion Mode: SUPABASE_DIRECT (Recommended) or API_GATEWAY
   INGESTION_MODE=SUPABASE_DIRECT

   # Supabase Direct Mode Credentials
   SUPABASE_URL=https://<your-project-id>.supabase.co
   SUPABASE_KEY=<your-service-role-secret-key>

   # Logging
   LOG_LEVEL=INFO
   ```
5. Run the Bridge Test Suite to verify logic:
   ```bash
   python test_bridge.py
   ```
   *(Expected result: 30 tests passing).*
6. Start the Bridge:
   ```bash
   python bridge.py
   ```

---

## 6. Frontend Web Application Setup (Next.js 15)

1. Open a terminal in the root repository directory.
2. Install dependencies:
   ```bash
   bun install
   # OR: npm install
   ```
3. Create the local environment file `.env.local`:
   ```env
   NEXT_PUBLIC_SUPABASE_URL=https://<your-project-id>.supabase.co
   NEXT_PUBLIC_SUPABASE_ANON_KEY=<your-supabase-anon-key>
   SUPABASE_SERVICE_ROLE_KEY=<your-service-role-secret-key>
   TELEMETRY_INGEST_KEY=demo_ingest_key_2026
   ```
4. Run the frontend test suite:
   ```bash
   bun test
   ```
   *(Expected result: 37 passing unit tests).*
5. Launch the development server:
   ```bash
   bun dev
   # OR: npm run dev
   ```
6. Open your browser and navigate to `http://localhost:3000`.
