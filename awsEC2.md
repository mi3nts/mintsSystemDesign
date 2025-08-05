
# ChirpStack & LoRaWAN Infrastructure – Roles, Storage, and Capacity Planning

This document summarizes the most likely roles, storage allocations, and data volume calculations for the ChirpStack & LoRaWAN deployment.

---

## Instance Roles & Storage

| Instance        | Type        | Role                          | Estimated Storage |
|-----------------|------------|--------------------------------|-------------------|
| **lora small**  | t3a.small  | Testing/Staging Server        | 20–30 GB          |
| **lora large 1**| t3a.large  | ChirpStack Network Server     | 50 GB             |
| **lora large 2**| t3a.large  | ChirpStack Application Server | 50 GB             |
| **lora medium 1**| t3a.medium | PostgreSQL Database          | 50–100 GB         |
| **lora medium 2**| t3a.medium | EMQX MQTT Broker             | 20–30 GB          |
| **lora medium 3**| t3a.medium | Redis/Memcached              | 20–30 GB          |

**Total recommended provisioned storage:** ~300–400 GB across all instances.

---

## Payload Volume Calculation

**Assumptions:**  
- Payload size: **100 bytes**  
- Interval: **30 seconds** (2 messages per minute)  
- Nodes: **250**  
- Gateways: **50** (minimal overhead)

### Per Node
- Messages per day:  
  `2 × 60 × 24 = 2,880 messages/day`
- Data per day per node:  
  `2,880 × 100 bytes ≈ 288,000 bytes (~0.274 MB)`

### For 250 Nodes
- `0.274 MB × 250 ≈ 68.5 MB/day`

### For 30 Days
- `68.5 MB/day × 30 ≈ 2.05 GB/month`

### Including Gateway Metadata
- Adding ~20–30 bytes per message:  
  `≈ 90 MB/day (≈ 2.7 GB/month)`

### Logs & Metadata
- Adding 10× overhead for PostgreSQL indexing, Redis caching, and logs:  
  `≈ 30–40 GB/month`

### Long-Term Planning
- For **6 months of retention**:  
  `30–40 GB/month × 6 ≈ 200–250 GB`

---

## Storage Recommendations

- **PostgreSQL:** 100–150 GB (scalable EBS volume for device metadata & indexing)  
- **Network Server:** 50 GB (OS, ChirpStack, and logs)  
- **Application Server:** 50 GB (OS, ChirpStack, and logs)  
- **EMQX Broker:** 30 GB (OS and transient logs)  
- **Redis/Memcached:** 30 GB (OS and ephemeral data)  
- **Testing/Staging Server:** 20–30 GB  

---

**Note:**  
Since **payloads are not permanently stored**, storage requirements are modest. The largest consumers are **PostgreSQL (session data + indexes)** and **ChirpStack logs**.
