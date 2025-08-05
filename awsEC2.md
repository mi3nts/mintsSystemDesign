
# ChirpStack & LoRaWAN Local Deployment Upgrade Plan

This document summarizes the **current cloud infrastructure** for the ChirpStack & LoRaWAN setup and proposes an **on‑premises upgrade** to support **500 nodes** and **50 gateways**.

---

## 1. Current Cloud Infrastructure

| Name            | Type       | vCPU | RAM  | Purpose                         |
|-----------------|-----------|------|------|---------------------------------|
| **lora small**  | t3a.small  | 2    | 2 GB | **Test/Staging server** – used for testing new ChirpStack configs and updates before moving to production. |
| **lora large 1**| t3a.large  | 2    | 8 GB | **ChirpStack Network Server (NS)** – manages LoRaWAN MAC layer, ADR, device sessions, and downlink scheduling. |
| **lora large 2**| t3a.large  | 2    | 8 GB | **ChirpStack Application Server (AS)** – decrypts payloads, provides APIs, and publishes to MQTT. |
| **lora medium 1**| t3a.medium | 2    | 4 GB | **PostgreSQL Database** – stores device/session data, join info, and ChirpStack metadata. |
| **lora medium 2**| t3a.medium | 2    | 4 GB | **EMQX MQTT Broker** – handles uplink/downlink messaging between ChirpStack components and external applications. |
| **lora medium 3**| t3a.medium | 2    | 4 GB | **Redis/Memcached** – caches session data and accelerates ChirpStack performance. |

**Current load:**  
- **Nodes:** ~150  
- **Gateways:** ~30  
- **Traffic:** 1 packet (100 bytes) every 15 s → ~576 kB per node per day  
- **Storage needs:** ~25–30 GB/month (including logs and indexes)  

---

## 2. Target Load After Upgrade
- **Nodes:** 500  
- **Gateways:** 50  
- **Traffic:** 1 packet (100 bytes) every 10 s → 8,640 packets/day per node  

**Estimated storage:**  
- **Raw payloads:** ≈ 13 GB/month  
- **With logs, indexes, and metadata:** ≈ 120–140 GB/month  
- **6‑month retention:** ≈ 700–800 GB  

---

## 3. Proposed Local Infrastructure (Upgraded)

| Component                     | vCPU | RAM  | Storage | Notes                                |
|------------------------------|------|------|---------|--------------------------------------|
| **ChirpStack Network Server**| 8    | 16 GB | 100 GB  | Handles MAC layer, ADR, and sessions |
| **ChirpStack Application Server** | 8 | 16 GB | 100 GB  | Decrypts payloads, API, MQTT output  |
| **PostgreSQL Database**      | 8    | 16 GB | 600 GB  | Stores device/session data & metadata |
| **EMQX MQTT Broker**         | 4    | 8 GB  | 100 GB  | Handles high-volume MQTT messaging   |
| **Redis/Memcached**          | 4    | 8 GB  | 50 GB   | Caching for performance              |
| **Testing/Staging Server**   | 2    | 4 GB  | 50 GB   | Optional development environment     |

**Total:** **32 vCPUs, 64 GB RAM, ~1 TB SSD storage.**

---

## 4. Key Improvements
- **Capacity Increase:** From **150 → 500 nodes** (over 3× scaling).  
- **Consolidation:** Fewer, more powerful servers replacing 6 smaller EC2 instances.  
- **Performance:** More vCPUs and RAM for ChirpStack, PostgreSQL, and MQTT under higher load.  
- **Resilience:** Ability to separate database and MQTT broker for fault isolation.  
- **Future‑proofing:** Scalable storage and compute to support growth beyond 500 nodes.

---

## 5. Recommendations
- Use **NVMe SSDs** for PostgreSQL for better write performance.  
- Separate **database** and **MQTT broker** onto dedicated hardware or VMs for reliability.  
- Enable **daily database backups** and weekly off‑site replication.  
- Use **TLS for MQTT** and enforce strict firewall rules.  
- Integrate **Prometheus/Grafana** for monitoring system performance.

---
