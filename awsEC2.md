
# ChirpStack & LoRaWAN Local Deployment Upgrade Plan

This document provides the specifications, architecture, and capacity planning to replicate and upgrade the current ChirpStack & LoRaWAN setup for **on-premises (local) deployment**.

---

## Objectives
- Deploy a **self-contained ChirpStack stack** (Network Server, Application Server, Gateway Bridge) on local infrastructure.
- Host **PostgreSQL**, **EMQX MQTT broker**, and **Redis** locally.
- Ensure the system can support **250 LoRaWAN nodes** and **50 gateways** with minimal latency.
- Provide **scalable storage** for 6+ months of operational data (session metadata, logs).

---

## Recommended Local Server Specifications

| Component                | vCPU | RAM  | Storage (EBS/Local SSD) | Notes                                |
|--------------------------|------|------|--------------------------|--------------------------------------|
| **ChirpStack Network Server**     | 4    | 8 GB  | 50 GB                   | Handles MAC layer, ADR, sessions    |
| **ChirpStack Application Server** | 4    | 8 GB  | 50 GB                   | Decrypts payloads, API, MQTT output |
| **PostgreSQL Database**           | 4    | 8 GB  | 150 GB                  | Stores device/session data & metadata |
| **EMQX MQTT Broker**              | 2    | 4 GB  | 30 GB                   | Handles MQTT messaging              |
| **Redis/Memcached**               | 2    | 4 GB  | 30 GB                   | Session caching for performance     |
| **Testing/Staging Server**        | 2    | 2 GB  | 20 GB                   | Optional – for development          |

**Total:** ~18–20 vCPUs, 34–36 GB RAM, 300–400 GB storage.

---

## Payload & Storage Planning

**Payload assumptions:**  
- **50 bytes per packet**  
- **1 packet every 15 seconds**  
- **250 nodes**  

**Daily volume per node:**  
- 4 messages/min × 60 × 24 = **5,760 messages/day**  
- 5,760 × 50 bytes ≈ **288 KB/day**  

**For 250 nodes:**  
- ≈ **68.5 MB/day**  
- ≈ **2–3 GB/month** (raw payload)  

**With logs, indexes, metadata:**  
- ≈ **40–50 GB/month**  

**6-month retention:**  
- ≈ **240–300 GB** total.

---

## Recommended Local Architecture

1. **Gateways** connect to **ChirpStack Gateway Bridge** running on the Network Server.  
2. **ChirpStack Network Server** handles LoRaWAN MAC layer, ADR, device sessions.  
3. **ChirpStack Application Server** decrypts payloads and pushes data to **EMQX MQTT broker**.  
4. **PostgreSQL** stores device sessions, join info, and metadata.  
5. **Redis** provides caching for session lookups and accelerates performance.  
6. **Optional staging server** for testing upgrades before production rollout.

---

## Upgrade Considerations
- **High Availability:** Consider using **two local servers** for Network/Application Server redundancy.  
- **Backups:** Regular PostgreSQL backups to external storage or cloud.  
- **Scalability:** Choose hardware that allows easy CPU/RAM upgrades for >250 nodes.  
- **Security:** Use TLS for MQTT, role-based DB access, and firewall rules for external access.  
- **Monitoring:** Integrate Prometheus/Grafana for local performance dashboards.  

---

## Next Steps
1. **Procure hardware:** Minimum 8-core CPU, 32 GB RAM, 500 GB SSD storage.  
2. **Install OS:** Ubuntu Server 22.04 LTS (or similar).  
3. **Deploy services:**  
   - PostgreSQL  
   - Redis  
   - EMQX MQTT broker  
   - ChirpStack stack (Gateway Bridge, Network Server, Application Server)  
4. **Configure:**  
   - LoRaWAN regions, device profiles, and gateways.  
   - Secure MQTT with TLS and authentication.  
   - Set up DB backup routines.  
5. **Test:** Validate connectivity with 1–2 gateways and a small device set before full migration.

---
