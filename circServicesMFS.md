# MooseFS (MFS) – Distributed Storage for MINTS/CIRC

MooseFS (**MFS**) is a **distributed, fault-tolerant file system** used across CIRC and MINTS infrastructure to provide **large-scale, shared storage** for sensor data, research outputs, and processing pipelines.

---

### Key Details
- **Master Node:** `mfsmaster.circ.utdallas.edu:9421`  
- **Mount Points:**  
  - **IMD:** `/mfs/io/groups/lary`  
  - **mdash:** `/mfs/io/groups/lary`  
  - **Other CIRC VMs:** Mounted as needed for shared code/data access  
- **Total Storage:** **14 TB** (current)  
- **Current Usage:** **~10 TB** (**73% used**)  

---

### Primary Use
- **Central code repository:**  
  - Houses **all GitHub repositories** for MINTS services (e.g., `mqttSubscribersV2`, `mqttLiveV3`, `mqttPMCorrections`, `mints-sensordata-to-postgres-backend`).  
  - Enables **consistent deployment** across IMD and mdash from a single source.  
- **Shared data storage:**  
  - Central repository for **raw sensor data**.  
  - **Processed data staging** for SharedAirDFW and other pipelines.  
- **Machine learning pipelines:**  
  - Storage for **model training data, calibration scripts, and output artifacts**.  

---

### Current Challenges
- **High utilization:** Already at **73%** → nearing critical thresholds.  
- **Shared access:** IMD, mdash, and other VMs rely on this pool simultaneously.  
- **No quota enforcement:** Risk of one pipeline consuming disproportionate space.  

---

### Proposed Upgrade
- **Expand capacity:** Increase MooseFS to **50 TB** to support scaling pipelines and long-term data retention.  
- **Implement quotas:** Enforce per-project or per-service quotas to ensure fair allocation.  
- **Enhanced monitoring:** Integrate **Prometheus** for **real-time usage tracking and alerting**.  
- **Backup integration:** Regular **snapshots to borg.circ** and **off-site replication** for disaster recovery.  
