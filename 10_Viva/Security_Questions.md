# Security Architecture & Validation Viva Preparation

## 1. Authentication & API Security

### Q1: How is the `/api/telemetry` ingestion endpoint secured against unauthorized access?
**Answer:**  
"The ingestion endpoint implements a **Fail-Closed Pre-Shared Bearer Key Architecture**:
1. **Server Configuration Verification:** On every incoming request, the server inspects `process.env.GATEWAY_API_KEY`. If the key is missing, blank, or set to a known insecure placeholder (e.g. `replace_with_secure_gateway_token_here`), the server aborts immediately with `HTTP 500 Server Misconfiguration`.
2. **Header Extraction:** The server extracts the token from either the `Authorization: Bearer <token>` header or the `x-api-key` header.
3. **Constant-Time Validation:** If the token does not match the configured secret, the request is rejected with `HTTP 401 Unauthorized`.
4. **Service Role Key Isolation:** Direct database inserts (`SUPABASE_DIRECT` mode) use the secret Supabase Service Role JWT, which is kept strictly on the host bridge machine and never exposed to the client browser."

---

### Q2: How does the system defend against Denial of Service (DoS) and oversized payloads?
**Answer:**  
"1. **Strict 64 KB Payload Limit:** In `/api/telemetry/route.ts`, the server inspects the `Content-Length` header and string payload length:
   ```typescript
   if (contentLength > 65536 || text.length > 65536) {
     return NextResponse.json({ error: 'Payload too large: Ingestion packet exceeds 64 KB limit.' }, { status: 413 });
   }
   ```
   This prevents malicious or corrupted clients from flooding server memory with megabyte-sized buffers.
2. **Rate Limiting & Duplicate Deduplication:** Retransmitted packets with identical sequence numbers are intercepted by the database unique constraint and returned as lightweight ACKs (`HTTP 200 / 409`), preventing unneeded database allocations."

---

## 2. Data Validation & Boundary Sanitization

### Q3: Why does `bridge.py` validate physical sensor boundaries before sending data to the cloud?
**Answer:**  
"In field IoT deployments, sensor hardware can experience electrical shorts, floating ADC lines, broken solder joints, or RF bit-flips in memory. If an unvalidated reading such as $-999^\circ\text{C}$ or $+50,000\text{ ADC}$ reached the cloud, it would corrupt historical statistics and trigger false emergency alarms.  
In `bridge/bridge.py`, we enforce strict physical bounds:
* **Temperature:** $-50.0^\circ\text{C} \le T \le +100.0^\circ\text{C}$
* **Humidity:** $0.0\% \le H \le 100.0\%$
* **Smoke ADC:** $0 \le \text{ADC} \le 4095$
* **Flame ADC:** $0 \le \text{ADC} \le 4095$
* **RSSI / SNR:** $-150 \le \text{RSSI} \le 0\text{ dBm}$ and $-30 \le \text{SNR} \le 30\text{ dB}$  
Packets violating these boundaries are rejected at the edge with a logged warning."

---

## 3. Database Security & Credential Isolation

### Q4: How is SQL Injection prevented throughout the codebase?
**Answer:**  
"SQL Injection is prevented at two structural layers:
1. **Parameterized Queries:** All database interactions in `lib/db/service.ts` utilize the Supabase PostgREST query builder (e.g. `.from('telemetry').insert(...)` and `.select()`), which executes parameterized SQL statements prepared by the PostgreSQL query planner. No user input is ever concatenated into raw SQL strings.
2. **PL/pgSQL Trigger Isolation:** Internal stored functions (`fn_evaluate_telemetry_alert`, `fn_verify_telemetry_gateway`) reference record fields via strongly-typed `NEW.field_name` variables, completely isolated from dynamic string execution (`EXECUTE`)."

---

### Q5: Explain the difference between `NEXT_PUBLIC_SUPABASE_ANON_KEY` and `SUPABASE_SERVICE_ROLE_KEY`.
**Answer:**  
"1. **`NEXT_PUBLIC_SUPABASE_ANON_KEY` (Client-Side / Public):**
   * This JWT key is embedded directly into the browser's JavaScript bundle.
   * It is subject to **Row Level Security (RLS)** policies defined in PostgreSQL, permitting public users to perform only `SELECT` operations on active incidents and public telemetry.
2. **`SUPABASE_SERVICE_ROLE_KEY` (Server-Side / Secret):**
   * This secret JWT has elevated administrative privileges that bypass Row Level Security.
   * It is stored securely on server environment variables (`.env.local` / Vercel secrets / `bridge/.env`) and is never delivered to the client browser."
