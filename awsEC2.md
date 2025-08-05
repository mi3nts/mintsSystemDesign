
# ChirpStack & LoRaWAN Infrastructure – Likely Roles

This document summarizes the most likely roles of each AWS EC2 instance in the ChirpStack & LoRaWAN deployment.

---

## **1. lora small**
- **ID:** i‑07a7399c5dbb6f979  
- **Type:** t3a.small (2 vCPU, 2 GiB RAM)  
- **Most Likely Role:**  
  - **Testing/Staging Server** for ChirpStack components.  
  - Could host a **test environment** for development or pilot device integrations before pushing to production.

---

## **2. lora large 1**
- **ID:** i‑023f0d2e73691b079  
- **Type:** t3a.large (2 vCPU, 8 GiB RAM)  
- **Most Likely Role:**  
  - **ChirpStack Network Server (NS):**  
    - Handles **LoRaWAN MAC layer**, **device session management**, **ADR**, **downlink scheduling**, and **MAC commands**.  
    - Communicates with the Gateway Bridge and Application Server.  

---

## **3. lora large 2**
- **ID:** i‑0c1cbeba834cf2246  
- **Type:** t3a.large (2 vCPU, 8 GiB RAM)  
- **Most Likely Role:**  
  - **ChirpStack Application Server (AS):**  
    - Decrypts application payloads using **AppSKey**.  
    - Exposes **REST and gRPC APIs** for device data.  
    - Publishes messages to the **EMQX MQTT broker** for downstream integrations.  
    - Handles **multi-tenant management** and device profiles.

---

## **4. lora medium 1**
- **ID:** i‑0c88a21ff5cf26794  
- **Type:** t3a.medium (2 vCPU, 4 GiB RAM)  
- **Most Likely Role:**  
  - **PostgreSQL Database:**  
    - Stores **device/session data**, **join information**, and **application metadata**.  
    - Essential for ChirpStack’s persistent storage.

---

## **5. lora medium 2**
- **ID:** i‑05228f1849fbf0ac3  
- **Type:** t3a.medium (2 vCPU, 4 GiB RAM)  
- **Most Likely Role:**  
  - **EMQX MQTT Broker:**  
    - Handles **uplink and downlink messaging** between Network Server and Application Server.  
    - Provides MQTT endpoints for external applications consuming sensor data.

---

## **6. lora medium 3**
- **ID:** i‑03b2c17738b728b06  
- **Type:** t3a.medium (2 vCPU, 4 GiB RAM)  
- **Most Likely Role:**  
  - **Redis/Memcached Instance:**  
    - Used for **caching device session data**.  
    - Helps **speed up queue processing** and improves performance of the Network Server.  

---

## **Overall Architecture (Likely Flow)**
1. **Gateways** connect via **ChirpStack Gateway Bridge** (running on Network Server).  
2. **Network Server (lora large 1)** processes LoRaWAN MAC-layer functions and communicates with the Application Server.  
3. **Application Server (lora large 2)** decrypts payloads and publishes data to the **EMQX MQTT broker**.  
4. **PostgreSQL (lora medium 1)** stores all device and session data.  
5. **Redis/Memcached (lora medium 3)** accelerates lookups and queue management for active sessions.  
6. **EMQX (lora medium 2)** handles all MQTT traffic for integrations.

---
