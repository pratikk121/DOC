# Technology Stack & Framework Selection Viva Preparation

## 1. Microcontroller & Embedded Hardware

### Q1: Why did you choose the ESP32 DevKit V1 over an Arduino Uno, Raspberry Pi, or STM32?
**Answer:**  
"1. **Versus Arduino Uno (ATmega328P):** The Arduino Uno is an 8-bit MCU running at $16\text{ MHz}$ with only $2\text{ KB}$ of SRAM and $32\text{ KB}$ Flash. It lacks the memory to buffer JSON payloads, execute SPI radio transfers at high speed, or support complex embedded stacks. The ESP32 provides a 32-bit dual-core Xtensa processor running at $240\text{ MHz}$, $520\text{ KB}$ SRAM, and $4\text{ MB}$ Flash.
2. **Versus Raspberry Pi (Single Board Computer):** A Raspberry Pi runs a full Linux OS, drawing $500\text{--}1500\text{ mA}$ continuously, has high boot times ($30\text{--}60\text{ s}$), and lacks native analog ADC pins (requiring external MCP3008 chips). It is too power-hungry for battery/solar forest deployments.
3. **Versus STM32:** While STM32 is power-efficient, the ESP32 has superior community support, native dual-core RTOS capabilities, and integrated hardware SPI for the Ra-02 LoRa module at lower cost."

---

### Q2: Why did you choose the Ai-Thinker Ra-02 (SX1278) LoRa module operating at 433 MHz?
**Answer:**  
"1. **Sub-GHz Physical Penetration:** 433 MHz RF waves have a long wavelength ($\lambda \approx 69\text{ cm}$) capable of diffracting around large tree trunks and penetrating thick foliage, unlike 2.4 GHz systems.
2. **High Receiver Sensitivity:** The Semtech SX1278 transceiver achieves sensitivity down to **$-148\text{ dBm}$**, allowing reception of signals weaker than ambient thermal noise.
3. **Low Power Consumption:** In sleep mode, the SX1278 draws less than $0.2\,\mu\text{A}$; during transmission, current is limited to $\approx 100\text{--}120\text{ mA}$ at $+20\text{ dBm}$."

---

## 2. Software Middleware & Backend Technologies

### Q3: Why did you choose Python for the serial bridge (`bridge.py`) instead of Node.js or C++?
**Answer:**  
"1. **Cross-Platform Serial Drivers:** Python's `pyserial` provides robust, rock-solid serial communication across Windows COM ports, Linux `/dev/ttyUSB*`, and macOS without compiling native C++ bindings.
2. **Resilient Exception Handling:** Python allows clean recovery loops and exponential backoff retry mechanisms when serial ports or network connections drop.
3. **Rapid Scripting & Testing:** Python's standard `unittest` framework allowed us to build 30 automated test cases verifying packet parsing, physical boundary checks, and API error simulations."

---

### Q4: Why did you choose Next.js 15 and React 19 for the frontend workstation?
**Answer:**  
"1. **Full-Stack Serverless Integration:** Next.js 15 App Router combines the tactical frontend dashboard and backend API ingestion routes (`/api/telemetry`, `/api/nodes`) in a single deployable repository.
2. **React 19 Concurrent Rendering:** High-frequency WebSocket telemetry streams update map pins and time-series charts smoothly without blocking user interactions or dropping frames.
3. **Vercel Edge Optimization:** Native asset optimization, fast route transitions, and immediate serverless edge deployment."

---

### Q5: Why did you choose Leaflet over Google Maps or Mapbox GL?
**Answer:**  
"1. **Zero API Cost & No Usage Quotas:** Google Maps and Mapbox require billing accounts and incur charges per map load. Leaflet is open-source and pairs seamlessly with free OpenStreetMap tile servers.
2. **Lightweight Bundle Size:** Leaflet's core library is only $\approx 42\text{ KB}$ gzipped, loading significantly faster than Mapbox GL ($> 500\text{ KB}$).
3. **Custom CSS Animated Markers:** Leaflet allows direct injection of HTML/CSS `DivIcon` elements, enabling our custom pulsing red threat rings for active wildfire nodes."

---

### Q6: Why did you choose Recharts for time-series visualization?
**Answer:**  
"1. **Declarative React SVG Architecture:** Recharts is built entirely on React components (`<ResponsiveContainer>`, `<LineChart>`, `<XAxis>`, `<Tooltip>`), eliminating direct imperative DOM manipulation.
2. **Multi-Metric Dynamic Toggling:** Easily supports layered line curves (Temperature, Humidity, Smoke, Flame, RSSI) with interactive legends and custom dark-theme tooltips.
3. **Hardware Acceleration:** Modern browser SVG rendering hardware-accelerates smooth transitions across historical data points."
