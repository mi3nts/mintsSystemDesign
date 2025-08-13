# **MINTS System Design**

## **Overall Architecture**
![MINTS Diagram](https://github.com/mi3nts/mintsSystemDesign/blob/main/res/mintsSystemDesignV3.png?raw=true)

---

# **MINTS Systems – Current vs. Recommended Architecture**

| System / Component | Current vCPU | Current RAM | Current Storage | Current Notes | Recommended vCPU | Recommended RAM | Recommended Storage | Recommended Notes |
|--------------------|--------------|-------------|-----------------|---------------|------------------|-----------------|--------------------|-------------------|
| **CIRC – virsh.circ (Hypervisor)** | 48 | 385 GB | 90 TB | Primary KVM host for all VMs | 48 | 385 GB | 90 TB | Retain as primary; adjust VM allocations |
| **CIRC – psql.circ (Master DB)** | 2 | 16 GB | 100 GB | PostgreSQL 11.9 master | 8 | 32 GB | 500 GB | Upgrade to v15+, increase RAM & disk |
| **CIRC – mintsdata.circ (API + Replica)** | 2 | 16 GB | 100 GB | Read-only DB, API, web services | 8 | 32 GB | 500 GB | Merge API + dashboards, containerize |
| **CIRC – www1.circ (Web Host)** | 2 | 16 GB | <30 GB | Hosts multiple sites | 8 | 32 GB | 200 GB | Consolidate MINTS apps, migrate to Ubuntu + Nginx |
| **CIRC – mosquitto/mqtt.circ (MQTT)** | 1 | 2 GB | 30 GB | Mosquitto 1.6.15 | 4 | 8 GB | 100 GB | Upgrade Mosquitto, enable TLS/mTLS |
| **CIRC – io-sftp.circ (Deployment)** | 2 | 8.5 GB | 5 GB | Deployment server for web updates | 4 | 16 GB | 50 GB | Add CI/CD tooling |
| **CIRC – borg.circ (Backup)** | N/A | N/A | 50 TB | Backup storage | N/A | N/A | 50 TB | Expand capacity, automate snapshots & off-site replication |
| **CIRC – mdash.circ (Processing)** | 16 | 64 GB | 1 TB + 14 TB MFS | InfluxDB, Grafana, Node-RED | 20 | 64 GB | 2 TB + 50 TB MFS | Add Prometheus, container orchestration |
| **LoRaWAN – lora small (Test/Staging)** | 2 | 2 GB | Small | Testing ChirpStack configs | 2 | 4 GB | 50 GB | Local staging environment |
| **LoRaWAN – lora large 1 (NS)** | 2 | 8 GB | Small | ChirpStack Network Server | 8 | 16 GB | 100 GB | MAC layer, ADR, sessions |
| **LoRaWAN – lora large 2 (AS)** | 2 | 8 GB | Small | ChirpStack Application Server | 8 | 16 GB | 100 GB | Payload decryption, API, MQTT output |
| **LoRaWAN – lora medium 1 (PostgreSQL)** | 2 | 4 GB | Small | ChirpStack DB | 8 | 16 GB | 600 GB | Store device/session data & metadata |
| **LoRaWAN – lora medium 2 (MQTT Broker)** | 2 | 4 GB | Small | EMQX MQTT Broker | 4 | 8 GB | 100 GB | High-volume MQTT messaging |
| **LoRaWAN – lora medium 3 (Redis/Memcached)** | 2 | 4 GB | Small | Caching layer | 4 | 8 GB | 50 GB | Performance caching |
| **MintsWiki – MediaWiki Server** | 1 | 1 GB | 10–20 GB | MediaWiki documentation server | 4 | 8 GB | 100 GB | Wiki, DB, uploads |
| **IMD – mintsdata.utdallas.edu** | 8 (16 threads) | 46 GB | 150 GB local + 14 TB MFS | CentOS 7, Dell R710 | 16 | 64 GB | 10 TB NVMe + 50 TB MFS | Ubuntu 22.04, enhanced ingestion & DB |
| **IMD – Backup NAS** | N/A | N/A | None | No dedicated backup | N/A | N/A | 40 TB | Daily/weekly backups of IMD + MooseFS |
| **MooseFS – Distributed Storage** | N/A | N/A | 14 TB (73% used) | Shared code/data for IMD, mdash, others | N/A | N/A | 50 TB | Expand capacity, quotas, monitoring, backups |

---

## **Detailed Reference Documents**
- **VIRSH Services:** [circServicesVIRSH.md](https://github.com/mi3nts/mintsSystemDesign/blob/main/circServicesVIRSH.md)  
- **LoRaWAN AWS Deployment:** [awsec2LNS.md](https://github.com/mi3nts/mintsSystemDesign/blob/main/awsec2LNS.md)  
- **Internal Mints Data:** [circServcesIMD.md](https://github.com/mi3nts/mintsSystemDesign/blob/main/circServcesIMD.md)  
- **MooseFS Storage:** [circServicesMFS.md](https://github.com/mi3nts/mintsSystemDesign/blob/main/circServicesMFS.md)  
- **Wiki AWS Deployment:** [awsec2Wiki.md](https://github.com/mi3nts/mintsSystemDesign/blob/main/awsec2Wiki.md)
