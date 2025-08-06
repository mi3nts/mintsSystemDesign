# Internal Mints Data (IMD) Local Deployment Upgrade Plan

This document summarizes the **current infrastructure** for the IMD (Internal Mints Data) system and proposes an **on‑premises upgrade** to enhance performance, reliability, and scalability for data acquisition and processing.

---

## 1. Current Infrastructure

| Hostname                    | OS                    | Hardware   | vCPU (Threads) | RAM   | Storage (Local) | Network Storage       | Purpose                              |
|-----------------------------|----------------------|------------|----------------|-------|-----------------|----------------------|--------------------------------------|
| **mintsdata.utdallas.edu**  | CentOS Linux 7 (Core) | Dell R710  | 8 (16 threads) | 46 GB | 150 GB (root+home) | 14 TB MooseFS (`/mfs/io/groups/lary`, 10 TB used) | Handles MINTS node data ingestion (RSYNC & MQTT), processing, and PostgreSQL integration for SharedAirDFW. |

**Key Characteristics:**  
- **Node.js version:** npm 6.14.5 (may be higher)  
- **Backups:** None currently implemented  
- **Load Balancer:** F5 (redirects `mintsdata:2222` → `imd:22` for SSH)  
- **Status:** No backup system, not integrated with automated failover  

---

## 2. IMD Dependencies for MINTS

IMD acquires and processes data through three main pipelines:

### **a. RSYNC Data from Directly Connected Nodes**  
- Automated rsync via cronjobs ensures data synchronization after reboots without manual intervention.  
- Specific SSH keys are established for secure connections.  

### **b. MQTT Data from Directly Connected Nodes & LoRaWAN Nodes**  
- Data acquisition scripts: [mqttSubscribersV2](https://github.com/mi3nts/mqttSubscribersV2)  
- Data calibration: [mqttLiveV3](https://github.com/mi3nts/mqttLiveV3)  
- Particulate matter corrections: [mqttPMCorrections](https://github.com/mi3nts/mqttPMCorrections)  

### **c. PostgreSQL Integration for SharedAirDFW**  
- CSV outputs from MQTT pipelines are ingested into PostgreSQL using [mints-sensordata-to-postgres-backend](https://github.com/mi3nts/mints-sensordata-to-postgres-backend)  
- Provides data for [SharedAirDFW](https://www.sharedairdfw.com/)  

---

## 3. Identified Issues
- **No backup system:** Local and MooseFS data are not currently backed up  
- **Limited processing power:** Dell R710 hardware with 8 vCPUs may bottleneck as pipelines grow  
- **Potential SSH outages:** Recent issues highlight the need for resilient remote management  
- **Legacy OS:** CentOS 7 nearing end-of-life; security and support concerns  
- **Limited shared storage:** Current MooseFS capacity (14 TB) is reaching 73% utilization  

---

## 4. Proposed Upgraded Infrastructure

| Component               | vCPU | RAM   | Storage (Local) | Network Storage        | Notes                                |
|------------------------|------|-------|-----------------|------------------------|--------------------------------------|
| **IMD Server**         | 16   | 64 GB | **10 TB NVMe**  | **50 TB MooseFS (expanded)** | Enhanced ingestion, processing, PostgreSQL, and shared storage |
| **Backup NAS (New)**   | N/A  | N/A   | N/A             | **40 TB**             | Dedicated storage for daily/weekly backups of IMD + MooseFS data |

**Total:** **16 vCPUs, 64 GB RAM, 10 TB primary local + 50 TB MooseFS + 40 TB backup NAS**

---

## 5. Software Stack
- **OS:** Ubuntu Server 22.04 LTS (replaces CentOS 7)  
- **Node.js:** Latest LTS version for compatibility with existing scripts  
- **Database:** PostgreSQL (for SharedAirDFW integration)  
- **Backup System:** Automated rsync/ZFS snapshots to dedicated NAS  
- **Monitoring:** Prometheus + Grafana for live system and service health dashboards  

---

## 6. MooseFS in IMD
IMD leverages **MooseFS (MFS)** as its **primary distributed file system** for bulk sensor data storage.  
- **Mounted at:** `/mfs/io/groups/lary`  
- **Current capacity:** 14 TB (10 TB used, 73% utilization)  
- **Proposed capacity:** **Expand to 50 TB** to accommodate growth for at least 5–7 years.  
- **Purpose:** Provides **shared, redundant storage** accessible by multiple CIRC systems (IMD, mdash, etc.).  
- **Key Benefit:** Ensures data is **fault-tolerant** and accessible across the cluster without duplicating datasets locally.  

---

## 7. Migration Plan
1. **Backup Current Data:**  
   - Full backup of all local data (rsync to external NAS)  
   - Backup all repositories and scripts (`/mfs/io/groups/lary/gitHubRepos`)  

2. **Provision New Hardware:**  
   - Deploy Ubuntu Server 22.04 LTS  
   - Configure RAID/NVMe for 10 TB primary storage  

3. **Reinstall Services:**  
   - Re-deploy rsync and MQTT pipelines  
   - Reinstall Node.js-based ingestion scripts  
   - Reconnect PostgreSQL to SharedAirDFW  

4. **Set Up Automated Backups:**  
   - Daily rsync to 40 TB NAS  
   - Weekly off-site/cloud backups  

5. **Testing:**  
   - Validate ingestion from all pipelines (RSYNC, MQTT, LoRaWAN)  
   - Verify PostgreSQL connectivity with SharedAirDFW  

---

## 8. Key Improvements
- **Modernized OS:** Ubuntu 22.04 ensures long-term support  
- **Performance:** Doubled CPU threads and increased RAM for faster processing  
- **Expanded Local Storage:** **10 TB NVMe** for high-speed ingestion & processing  
- **Expanded Shared Storage:** **MooseFS expanded from 14 TB → 50 TB** for long-term growth  
- **Data Resilience:** Daily NAS backups + weekly off-site backups  
- **Monitoring:** Prometheus + Grafana integration for system visibility  
- **Future-proofing:** Scalable architecture for growing data pipelines  

---

# Key Running Processes (How to Manage Them)

### RSYCED data from directly connected nodes
The rsync scripts run automatically on the nodes via cronjobs, ensuring that in the event of an IMD system reboot, data synchronization is automatic. As a result, there is no need for any manual intervention on IMD's part after an unexpected reboot. We have established specific SSH keys to facilitate these connections.


### MQTT Data 

As an additional layer of data backup, we store data from the two MQTT pipelines. The firmware required for this task is available at [mqttSubscribersV2](https://github.com/mi3nts/mqttSubscribersV2). Within the IMD system, you can locate the repository on the mfs mount at /mfs/io/groups/lary/gitHubRepos/mqttSubscribersV2/firmware.

```
cd /mfs/io/groups/lary/gitHubRepos/mqttSubscribersV2/firmware
nohup ./runDataReaders.sh > /dev/null 2>&1 &
```

## SharedAirDFW Support 
The data produced by MINTS Nodes, accessible through [sharedairdfw.com](https://www.sharedairdfw.com/), is archived in a PostgreSQL database. To ensure seamless integration with SharedairDFW, we generate structured .csv files from the two MQTT pipelines. It's important to note that, while not mandatory for SharedairDFW, we employ machine learning-based calibration techniques, utilizing a research-grade climate sensor to enhance the quality of the collected climate data. This calibrated data is also stored in the PostgreSQL database. However, it's worth mentioning that the demand for machine learning-calibrated climate data is currently unnecessary, as sharedairdfw.com does not currently display climate data from MINTS nodes.

### MQTT data from directly connected nodes for sharedairdfw.com as well as LoRaWAN Nodes
The process of calibrating climate data, along with the steps of data cleansing and averaging, is managed by the firmware accessible on [GitHub - mi3nts/mqttLiveV3](https://github.com/mi3nts/mqttLiveV3). Within the IMD system, this repository is located on the mfs mount at /mfs/io/groups/lary/gitHubRepos/mqttLive/firmware.
```
cd /mfs/io/groups/lary/gitHubRepos/mqttLiveV3/firmware
nohup ./runDataReaders.sh  >/dev/null 2>&1  &
```

### Live data migration into PostgreSQL
[sharedairdfw.com](https://www.sharedairdfw.com/) relies on the structured data stored in CSV files, which are imported into a PostgreSQL database. This data integration process is facilitated by the firmware accessible at [GitHub - mi3nts/mints-sensordata-to-postgres-backend](https://github.com/mi3nts/mints-sensordata-to-postgres-backend). Within the IMD system, you can find this repository on the mfs mount at /mfs/io/groups/lary/mints-sensordata-to-postgres-backend.
```
cd /mfs/io/groups/lary/mints-sensordata-to-postgres-backend
top | grep node
```
At this poin you should see something similar to the following 
```
 1434 mints     20   0  924556  63372  15816 S   2.3  0.1   2:59.65 node                                                                                                                                                                         
 1434 mints     20   0  924556  63372  15816 S   0.3  0.1   2:59.66 node                                                                                                                                                                         
 1434 mints     20   0  924556  63964  15816 S   2.3  0.1   2:59.73 node       
```

Make sure the user is mints (second column) and the process ID (first column) is the same for all the rows. Afterwards kill the relavant process ID ( **1434 for this particular case** ).
```
kill 1434
```

The check in any Node JS processors are active.

```
top | grep node
```

If no processors are active, do 
```
./headlessStart.sh
```

**Please ensure that multiple instances of these codes are not executed simultaneously.**

### MINTS Particulate Matter Corrections 
Mints offer particulate matter corrections for data that may be impacted by precipitation forming around particulate matter. The necessary firmware for this task can be found at mqttPMCorrections. Within the IMD system, the repository is located on the mfs mount at /mfs/io/groups/lary/gitHubRepos/mqttPMCorrections/firmware.

```
cd /mfs/io/groups/lary/gitHubRepos/mqttPMCorrections/firmware
nohup ./runCorrections.sh  >/dev/null 2>&1  &
```
