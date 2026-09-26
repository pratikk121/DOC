# Technical Questions & Deep-Dive Viva Preparation

## 1. Embedded Systems & Sensor Hardware

### Q1: Why did you connect the analog sensors (MQ-2 and Flame IR) to GPIO 34 and GPIO 35 specifically?
**Answer:**  
"On the ESP32 microcontroller, analog-to-digital converter channels are split between **ADC1** and **ADC2**.
* **ADC2** (channels on GPIO 0, 2, 4, 12, 13, 14, 15, 25, 26, 27) is shared internally with the ESP32 Wi-Fi / RF subsystem driver. Whenever Wi-Fi is initialized or hardware interrupts fire, ADC2 readings fail or return corrupt values.
* **ADC1** (GPIO 32, 33, 34, 35, 36, 39) is completely independent of the RF subsystem.
* Furthermore, **GPIO 34 and GPIO 35** are input-only pins without internal software pull-up/pull-down resistors, making them ideal, high-impedance analog inputs for external analog sensor outputs."

---

### Q2: Why does the optical flame sensor output high values in normal ambient light and drop to near zero during a fire?
**Answer:**  
"The optical flame sensor utilizes an NPN-type infrared photodiode sensitive to the $760\text{ nm}$ to $1100\text{ nm}$ optical emission spectrum typical of hydrocarbon flames. In the absence of flame, the photodiode exhibits high internal reverse resistance, pulling the analog output close to the $3.3\text{V}$ rail ($4095\text{ ADC}$). When exposed to intense infrared radiation from an open flame, the photon-induced electron flow drops the internal resistance sharply, causing the analog voltage on GPIO 35 to collapse towards $0\text{V}$ ($0\text{--}100\text{ ADC}$). Our firmware and database triggers explicitly check for `flame_raw < 100` to register a positive detection."

---

### Q3: How does the DHT22 single-wire communication protocol work on GPIO 27?
**Answer:**  
"The DHT22 (AM2302) uses a proprietary bidirectional single-wire time-division multiplexing protocol:
1. **Start Signal:** The ESP32 pulls the data line low for at least $1\text{ ms}$ (typically $18\text{ ms}$) and then releases it high for $20\text{--}40\,\mu\text{s}$.
2. **Sensor Response:** The DHT22 responds by pulling the line low for $80\,\mu\text{s}$, followed by high for $80\,\mu\text{s}$.
3. **Data Transmission:** The sensor transmits 40 bits of data (16 bits humidity, 16 bits temperature, 8 bits parity checksum). A bit `'0'` is represented by a $50\,\mu\text{s}$ low pulse followed by a $26\text{--}28\,\mu\text{s}$ high pulse; a bit `'1'` is represented by a $50\,\mu\text{s}$ low pulse followed by a $70\,\mu\text{s}$ high pulse.
4. **Checksum Verification:** The final 8-bit checksum must equal the lower 8 bits of the sum of the data bytes; otherwise, the packet is discarded."

---

### Q4: Explain LoRa Modulation, Spreading Factor, and Bandwidth used in your SX1278 setup.
**Answer:**  
"LoRa (Long Range) is a physical-layer wireless modulation based on **Chirp Spread Spectrum (CSS)** technology.
* **Carrier Frequency ($433.0\text{ MHz}$):** A sub-GHz Industrial, Scientific, and Medical (ISM) frequency band with superior foliage penetration and diffraction characteristics compared to $2.4\text{ GHz}$.
* **Bandwidth ($\text{BW} = 125\text{ kHz}$):** Defines the frequency sweep width of the chirp signal. A $125\text{ kHz}$ bandwidth offers a proven balance between transmission data rate and receiver noise floor sensitivity.
* **Spreading Factor ($\text{SF} = 7$):** The number of chirps per symbol is $2^{\text{SF}} = 2^7 = 128$ chips per symbol. SF7 yields a fast on-air transmission time ($\approx 40\text{--}60\text{ ms}$ per packet), keeping power consumption low.
* **Coding Rate ($\text{CR} = 4/5$):** Cyclic error-correcting code where 4 bits of payload data include 1 redundant error-correction bit, allowing the receiver to reconstruct corrupted bits without retransmission."

---

## 2. Python Hardware Bridge & Middleware

### Q5: How does `bridge.py` maintain reliable serial communication without crashing when the gateway is unplugged?
**Answer:**  
"The bridge is engineered as a state-machine loop wrapped in a robust exception-handling hierarchy:
1. **Auto-Discovery:** On startup, `find_gateway_port()` scans system COM ports (identifying Silicon Labs CP210x / CH340 / FTDI USB-UART chips).
2. **Reconnection Loop:** If the gateway is unplugged during operation, the serial read raises a `serial.SerialException`. The bridge traps the exception, enters a safe `RECONNECTING` state, logs the event, and uses an exponential backoff retry loop ($2\text{s}, 4\text{s}, 8\text{s}\dots$) to poll for USB re-insertion without terminating the Python process.
3. **Line-Buffering & Framing:** The bridge reads lines using `serial.readline()` and validates that the line begins with the exact prefix `GATEWAY_PACKET:`. Any unstructured microcontroller boot logs or noise are filtered out before JSON parsing."

---

### Q6: How does the system prevent duplicate packet entries during RF retransmissions?
**Answer:**  
"Deduplication is enforced at two distinct tiers:
1. **Database Tier:** The `telemetry` table has a composite unique constraint: `UNIQUE (node_id, sequence_number)`. If the same packet sequence is inserted twice, PostgreSQL raises error code `23505 (unique_violation)`.
2. **Bridge / API Tier:**
   * In `SUPABASE_DIRECT` mode, Supabase returns `HTTP 409 Conflict`. `bridge.py` traps the 409 status and logs `[INFO] [SUPABASE] status=409 (Duplicate packet ignored)` without treating it as a failure.
   * In `API_GATEWAY` mode, the `/api/telemetry` handler detects the existing sequence and returns `HTTP 200 OK` with `{ duplicate: true }`, ensuring the edge bridge does not get stuck in an unnecessary retry loop."

---

## 3. Frontend Architecture & Real-Time Sync

### Q7: Why did you use Next.js 15 App Router instead of Pages Router or plain React Vite?
**Answer:**  
"Next.js 15 App Router provides several key architectural advantages:
1. **Unified Serverless API Routes:** Eliminates the need to maintain a separate Node.js Express server; all REST endpoints (`/api/telemetry`, `/api/nodes`, `/api/alerts`, `/api/export`) run directly on edge serverless functions.
2. **React 19 Server & Client Components:** Heavy static elements (schema viewer layouts, navigation headers) render as fast Server Components, while interactive dashboards (`OperatorMap.tsx`, `TelemetryCharts.tsx`) run as dedicated Client Components with `'use client'`.
3. **Zero-Configuration Production Builds:** Built-in TypeScript compiler, Tailwind CSS v4 engine, and seamless deployment on Vercel."

---

### Q8: How do you prevent Leaflet map crashes during Next.js Server-Side Rendering (SSR)?
**Answer:**  
"Leaflet depends heavily on browser window globals (`window`, `document`, `navigator`). During Next.js Server-Side Rendering on Node.js/Vercel, these objects do not exist, which causes Leaflet to throw a fatal `ReferenceError: window is not defined`.  
We resolved this by importing the map component dynamically with SSR explicitly disabled:
```typescript
const OperatorMap = dynamic(() => import('@/components/OperatorMap'), {
  ssr: false,
  loading: () => <div className=\"h-full w-full bg-slate-900 animate-pulse\" />
});
```
This guarantees Leaflet initializes only on the client browser after DOM hydration."

---

### Q9: How is the audio siren implemented without external audio files?
**Answer:**  
"In `lib/audio.ts`, we implemented a browser-native synthesizer using the **Web Audio API**:
1. It instantiates an `AudioContext` and creates an `OscillatorNode` generating a square waveform.
2. It modulates the frequency between $880\text{ Hz}$ (High tone) and $440\text{ Hz}$ (Low tone) at 2 Hz intervals to mimic a dual-tone emergency vehicle siren.
3. Audio gain is routed through a `GainNode` for smooth volume ramping and instantaneous muting.
This approach eliminates network bandwidth overhead, eliminates 404 missing asset errors, and operates completely offline."
