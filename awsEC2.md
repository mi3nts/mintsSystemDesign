
# ChirpStack & LoRaWAN Infrastructure – Roles, Storage, and Capacity Planning

This document summarizes the most likely roles, CPU/RAM specifications, storage allocations, and data volume calculations for the ChirpStack & LoRaWAN deployment.

---

## Instance Roles, Specs & Storage

| Instance         | Type        | vCPU | RAM  | Role                          | Estimated Storage |
|------------------|-------------|------|------|--------------------------------|-------------------|
| **lora small**   | t3a.small   | 2    | 2 GB | Testing/Staging Server        | 20–30 GB          |
| **lora large 1** | t3a.large   | 2    | 8 GB | ChirpStack Network Server     | 50 GB             |
| **lora large 2** | t3a.large   | 2    | 8 GB | ChirpStack Application Server | 50 GB             |
| **lora medium 1**| t3a.medium  | 2    | 4 GB | PostgreSQL Database          | 50–100 GB         |
| **lora medium 2**| t3a.medium  | 2    | 4 GB | EMQX MQTT Broker             | 20–30 GB          |
| **lora medium 3**| t3a.medium  | 2    | 4 GB | Redis/Memcached              | 20–30 GB          |

**Total recommended provisioned storage:** ~300–400 GB across all instances.

---

## Payload Volume Calculation

**Assumptions:**  
- Payload size: **50 bytes**  
- Interval: **15 seconds** (4 messages per minute)  
- Nodes: **250**  
- Gateways: **50** (minimal overhead)

### Per Node
- Messages per day:  
  `4 × 60 × 24 = 5,760 messages/day`
- Data per day per node:  
  `5,760 × 50 bytes ≈ 288,000 bytes (~0.274 MB)`

### For 250 Nodes
- `0.274 MB × 250 ≈ 68.5 MB/day`

### For 30 Days
- `68.5 MB/day × 30 ≈ 2.05 GB/month`

### Including Gateway Metadata
- Adding ~20–30 bytes per message:  
  `≈ 135 MB/day (≈ 4 GB/month)`

### Logs & Metadata
- Adding 10× overhead for PostgreSQL indexing, Redis caching, and logs:  
  `≈ 40–50 GB/month`

### Long-Term Planning
- For **6 months of retention**:  
  `40–50 GB/month × 6 ≈ 240–300 GB`

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
