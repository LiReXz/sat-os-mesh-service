# AGENT DIRECTIVES & SYSTEM CONSTRAINTS (`GEMINI.md`)
> **MANDATORY READING:** This file contains absolute architectural rules and constraints for any AI Agent (Gemini, Claude, Cursor, Copilot) operating on this repository.

---

## 1. ISOLATION PRINCIPLE (ZERO TRUST)
- This repository is strictly the **`sat-os-mesh-service`** component.
- **PROHIBITED:** Implementing client business logic, image processing, AOCS physics calculations, or direct external internet connectivity.
- **RESTRICTED IPC:** Inbound commands are accepted **ONLY** from `sat-os-core-api` via authenticated internal gRPC/IPC. Direct socket exposure to client runtimes is strictly forbidden.
- Physical transceiver hardware communication MUST be strictly confined within the Hardware Abstraction Layer (`src/hal/`).

---

## 2. COMPONENT-SPECIFIC RULES
1. **Safety-Critical Transmission Limits:** All outbound RF/optical transmissions must check thermal headroom and power bus availability before acquiring link lock.
2. **Buffer Overflow & OOM Prevention:** Store-and-forward bundle storage MUST enforce a hard disk/memory cap. If storage reaches 90% capacity, drop lowest-priority bundles and log telemetry alerts.
3. **No Direct Ground Routing:** This service handles inter-satellite links ONLY. Downlink transmission to ground stations is the responsibility of dedicated communications modules.
4. **Ephemeris Stale-Check:** Do NOT initiate laser tracking or RF directional lock if local relative ephemeris data is older than 24 hours without explicit override from `sat-os-core-api`.
5. **Fail-Safe Disconnect:** On packet corruption rate exceeding 40% over 10 consecutive frames, terminate active link session and enter discovery scan mode.

---

## 3. CODE STANDARDS & TESTING
- **Language:** English ONLY for code, comments, documentation, and commit messages.
- **C++ Standard:** Modern C++20 with strict compilation flags (`-Wall -Wextra -Werror -pedantic`).
- **Memory Safety:** Prefer smart pointers, RAII, and value semantics. Naked pointers and raw memory allocations (`malloc/free`) are strictly forbidden.
- **Coverage:** Unit tests for packet serialization, bundle routing, and protocol encoding MUST exceed 90% branch coverage.
- **Deterministic Clocks:** Always use monotone spacecraft clock abstractions for time-difference calculations; never rely on system wall-clock time.

---

## 4. FORBIDDEN PATTERNS
- **NO Client-Writable Routing Tables:** Client software may request data delivery to a peer ID; they CANNOT alter the routing topology or frequency calibrations.
- **NO Infinite Retries:** All transmission queues must implement exponential backoff with bounded max attempts (default max: 5 attempts).
- **NO Blocking System Calls on Packet Reception:** All transceiver I/O must run on non-blocking asynchronous event loops.

---

## 5. INTER-REPOSITORY COMMUNICATION
- **Upstream Caller:** `sat-os-core-api` (Inbound dispatch commands and telemetry queries).
- **Hardware Coordination:** `sat-os-motion-service` (Sends pointing correction vectors for optical alignment via internal bus).
- **Health Telemetry:** Periodically reported to `sat-os-observability-agent`.

---

## 6. SECURITY & COMPLIANCE
- **Cryptographic Authentication:** All mesh frames sent over ISL MUST be authenticated and encrypted using symmetric session keys managed by the platform HSM/enclave.
- **Zero Raw Memory Leaks:** Transceiver driver buffers must be zeroed upon release to prevent data leakage between different missions/clients.

---

## 7. CHANGE MANAGEMENT
- Protocol buffer contracts in `protos/mesh_service.proto` follow semantic versioning. 
- Breaking message schema updates require a deprecation cycle and synchronized updates with `sat-os-core-api`.
