
# ChirpStack & LoRaWAN Infrastructure – Roles, Storage, and Capacity Planning

This document summarizes the most likely roles, storage allocations, and data volume calculations for the ChirpStack & LoRaWAN deployment.

---

## **Instance Roles**

### **1. lora small**
- **Type:** t3a.small (2 vCPU, 2 GiB RAM)  
- **Role:** **Testing/Staging Server** for ChirpStack components.  
- **Estimated Storage:** ~20–30 GB (minimal, for OS + test environment).

### **2. lora large 1**
- **Type:** t3a.large (2 vCPU, 8 GiB RAM)  
- **Role:** **ChirpStack Network Server (NS):** Handles LoRaWAN MAC layer, device session management, ADR, downlink scheduling, and MAC commands.  
- **Estimated Storage:** ~50 GB (OS + ChirpStack + logs).

### **3. lora large 2**
- **Type:** t3a.large (2 vCPU, 8 GiB RAM)  
- **Role:** **ChirpStack Application Server (AS):** Decrypts application payloads, provides APIs, and publishes to the EMQX MQTT broker.  
- **Estimated Storage:** ~50 GB (OS + ChirpStack + logs).

### **4. lora medium 1**
- **Type:** t3a.medium (2 vCPU, 4 GiB RAM)  
- **Role:** **PostgreSQL Database:** Stores device/session data, join information, and application metadata.  
- **Estimated Storage:** ~50–100 GB.

### **5. lora medium 2**
- **Type:** t3a.medium (2 vCPU, 4 GiB RAM)  
- **Role:** **EMQX MQTT Broker:** Handles uplink/downlink messaging between Network Server, Application Server, and integrations.  
- **Estimated Storage:** ~20–30 GB.

### **6. lora medium 3**
- **Type:** t3a.medium (2 vCPU, 4 GiB RAM)  
- **Role:** **Redis/Memcached:** Provides caching for device sessions and queue processing.  
- **Estimated Storage:** ~20–30 GB.

---

## **Payload Volume Calculation**

**Assumptions:**  
- **Payload:** 100 bytes  
- **Interval:** 30 seconds (2 messages/minute)  
- **Nodes:** 250  
- **Gateways:** 50 (minimal additional metadata overhead)  

### **1. Per node**
- Messages per day:  
  \( 2 	imes 60 	imes 24 = 2,880 \)
- Data per day per node:  
  \( 2,880 	imes 100 pprox 288,000\ \mathrm{bytes} \ (pprox 0.274\ \mathrm{MB}) \)

### **2. For 250 nodes**
\( 0.274\ \mathrm{MB} 	imes 250 pprox 68.5\ \mathrm{MB/day} \)

### **3. For 30 days**
\( 68.5\ \mathrm{MB/day} 	imes 30 pprox 2.05\ \mathrm{GB/month} \)

### **4. Including gateway metadata**
Adding ~20–30 bytes overhead per message:  
\( pprox 90\ \mathrm{MB/day} \ (pprox 2.7\ \mathrm{GB/month}) \)

### **5. Logs & metadata**
Adding 10× overhead for PostgreSQL indexing, Redis caching, and logs:  
\( pprox 30–40\ \mathrm{GB/month} \)

### **6. Long-term planning**
For **6 months of retention:**  
\( 30–40\ \mathrm{GB/month} 	imes 6 pprox 200–250\ \mathrm{GB} \)

---

## **Storage Recommendations**

- **PostgreSQL:** 100–150 GB (scalable EBS volume for device metadata & indexing).  
- **Network Server:** 50 GB (OS, ChirpStack, and logs).  
- **Application Server:** 50 GB (OS, ChirpStack, and logs).  
- **EMQX Broker:** 30 GB (OS and transient logs).  
- **Redis/Memcached:** 30 GB (OS and ephemeral data).  
- **Testing/Staging Server:** 20–30 GB.

**Total recommended provisioned storage:** ~300–400 GB across all instances.

---

**Note:**  
Since **payloads are not permanently stored**, storage requirements are modest. The largest consumers are **PostgreSQL (session data + indexes)** and **ChirpStack logs**.
