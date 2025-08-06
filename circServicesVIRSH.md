# CIRC Services for MINTS

## virsh.circ.utdallas.edu (VIRSH Machine)
- **VM?**: **No** – This is a **physical host**.
- **OS**: CentOS Linux 8  
- **Service**: **virsh 4.5.0** (KVM hypervisor; hosts all VMs)  
- **Purpose**:  
  - Serves as the **hypervisor host** for many CIRC virtual machines:  
    - `www1.circ.utdallas.edu` (web host)  
    - `mosquitto/mqtt.circ.utdallas.edu` (MQTT broker)  
    - `mintsdata.circ.utdallas.edu` (PostgreSQL subscriber & API host)  
    - `psql.circ.utdallas.edu` (PostgreSQL master)  
    - `io-sftp.circ.utdallas.edu` (deployment server)  
    - `mdash.circ.utdallas.edu` (sensor processing)  
- **Storage**: 90 TB  
- **Hardware**: Intel Storage server purchased from PSSC Labs  
- **CPU / RAM**: 48 CPUs, 385 GB RAM  
- **Externally Accessible?**: No  
- **Internal IP**: `10.247.245.145`  

### Quick Reference Table

### Quick Reference Table (with CPU, RAM, and Storage)

| Hostname               | Internal IP      | Role                                     | vCPU | RAM    | Storage  |
|------------------------|------------------|------------------------------------------|------|--------|----------|
| **virsh.circ**         | 10.247.245.145   | Hypervisor (KVM host)                    | 48   | 385 GB | 90 TB    |
| **psql.circ**          | 10.247.245.219   | Master PostgreSQL DB                     | 2    | 16 GB  | 100 GB   |
| **mintsdata.circ**     | 10.247.245.211   | Read-only DB, API, web services          | 2    | 16 GB  | 100 GB   |
| **www1.circ**          | 10.182.78.148    | Web host (multi-site)                    | 2    | 16 GB  | <30 GB   |
| **mosquitto/mqtt.circ**| 10.247.245.206   | MQTT broker (IoT data)                   | 1    | 2 GB   | 30 GB    |
| **io-sftp.circ**       | 10.182.78.143    | Deployment / SFTP                        | 2    | 8.5 GB | 5 GB     |
| **borg.circ**          | 10.182.78.141    | Backup storage                           | N/A  | N/A    | 50 TB    |
| **mdash.circ**         | 10.247.245.223   | Sensor processing + Influx/Grafana/Node-RED | 16 | 64 GB  | 1 TB + 14 TB (MFS) |


---

## mintsdata.circ.utdallas.edu
- **VM?**: Yes  
- **Purpose**:  
  - **Read-only PostgreSQL subscriber** for the MINTS project  
  - Hosts **public APIs & dashboards** (e.g., SharedAirDFW)  
  - **Web services** (Node.js + nginx proxy)  
  - Pulls **data updates from psql.circ.utdallas.edu** (master DB)  
  - **Code updates** automated via cron job on `io-sftp`  
- **Storage**: Part of IO-mounted web storage  
- **Externally Accessible?**: Yes (via `https://api.sharedairdfw.com`)  
- **Internal IP**: `10.247.245.211`  

**Questions:**
 - **Are any other SharedAirDFW services running on this server?**
 - **How does this tie into our existing crontab for sensor data processing?**
 - **Is there any connection to the IMD server? If so, how is it linked?** 

---

## www1.circ.utdallas.edu
- **VM?**: Yes  
- **OS**: CentOS Linux 8  
- **Service**: nginx (web server & reverse proxy)  
- **Purpose**: Hosts multiple websites:  
  - `sharedairdfw.com`  
  - `utdmint.info`  
  - `python.davidlary.info`  
  - `ganymededocs.circ.utdallas.edu`  
  - `mintsdata.utdallas.edu`  
  - `davidlary.info`  
  - `cisnerosres.utdallas.edu`  
- **Backups**: `/var/www` is mounted via IO (likely backed up at storage level)  
- **Storage**: <30 GB  
- **CPU / RAM**: 2 CPUs, 16 GB RAM  
- **Internal IP**: `10.182.78.148`  
**Questions:**
**Will we get ownership of this, if so what services should I be running here. How do i manage sharedairdfw and other mints websites here** 

---

## mosquitto.circ.utdallas.edu / mqtt.circ.utdallas.edu
- **VM?**: Yes  
- **OS**: CentOS Linux 8  
- **Service**: Mosquitto 1.6.15 (MQTT broker)  
- **Purpose**: Handles **IoT sensor data** publishing/subscriptions for the MINTS ecosystem  
- **Special Notes**:  
  - Requires valid SSL cert: `/etc/pki/mosquitto/certs/mqtt.circ.utdallas.edu.crt`  
- **Storage**: 30 GB  
- **CPU / RAM**: 1 CPU, 2 GB RAM  
- **Internal IP**: `10.247.245.206`  

---

## psql.circ.utdallas.edu
- **VM?**: Yes  
- **OS**: Debian GNU/Linux 10  
- **Service**: PostgreSQL 11.9 (**primary master database**)  
- **Purpose**:  
  - **Main PostgreSQL DB** for the MINTS ecosystem  
  - Replicated by `mintsdata.circ.utdallas.edu` (read-only)  
  - **Backups** stored via `borg.circ.utdallas.edu`  
- **Storage**: 100 GB  
- **CPU / RAM**: 2 CPUs, 16 GB RAM  
- **Externally Accessible?**: No  
- **Internal IP**: `10.247.245.219`  

---

## io-sftp.circ.utdallas.edu
- **VM?**: Yes  
- **OS**: CentOS Linux 8  
- **Service**: SFTP + automation  
- **Purpose**:  
  - **Deployment server** for updating the SharedAirDFW frontend from GitHub  
  - Runs **cron job**:  
    ```bash
    cd /mfs/io/circ/www/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map && \
    bash update.sh >> /mfs/io/circ/www/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map/update.log
    ```  
- **Storage**: 5 GB  
- **CPU / RAM**: 2 CPUs, 8.5 GB RAM
- **Internal IP**: `10.182.78.143`

---

## borg.circ.utdallas.edu
- **VM?**: Yes  
- **Service**: Borg Backup  
- **Purpose**:  
  - **Central backup server**  
  - Stores deduplicated, compressed backups for CIRC systems (**especially `psql`**)  
- **Access**: Controlled by CIRC admins (via Ansible)  
- **Internal IP**: `10.182.78.141`
---

## mdash.circ.utdallas.edu
- **VM?**: Yes  
- **OS**: Debian GNU/Linux 11  
- **Purpose**:  
  - Ingests raw sensor data and processes it for the **Open Storage Network (OSN)**.  
  - Hosts **Podman containers** for:  
    - **InfluxDB** (time-series database for sensor data)  
    - **Grafana** (data visualization dashboards)  
    - **Node-RED** (data processing workflows and automation)  
  - Supports **automated reporting** via **Quarto**.  

### Network Information
- **Internal IP**: `10.247.245.223`  
- **External/Public IP**: `129.110.247.1`  
- **Subnet**: `10.247.245.128/25`  
- **Default Gateway**: `10.247.245.129`  

### Storage
- **Local Volumes (LVM):**
  - `/home`: **1 TB** total, **390 GB used** (41%) → likely container data and user files  
- **Network-mounted storage:**  
  - `/mfs/io/groups/lary`: **14 TB total**, **10 TB used** (73%)  
---


# Proposed Upgrade Plan for CIRC Services

### Objectives
- **Consolidate**: Reduce VM sprawl by grouping related services (e.g., web, API, and processing) where feasible.
- **Scale**: Increase CPU/RAM to support higher traffic and heavier analytics workloads (e.g., InfluxDB, Grafana, Quarto reporting).
- **Modernize**: Upgrade all CentOS 8 systems (EOL) to **Ubuntu Server 22.04 LTS** for long-term support.
- **Secure & Resilient**: Improve backup coverage and disaster recovery with automated snapshots and off-site replication.
- **Streamline**: Establish clear ownership of SharedAirDFW services and unify management of all MINTS web services under a single VM or containerized cluster.

### Recommended Upgraded Architecture

| Component                  | vCPU | RAM   | Storage | Notes                                   |
|---------------------------|------|-------|---------|----------------------------------------|
| **virsh.circ (Hypervisor)** | 48   | 385 GB | 90 TB   | Remains primary KVM host, expand VM allocations. |
| **psql.circ (Master DB)**   | 8    | 32 GB  | 500 GB  | Upgrade PostgreSQL to v15+, increase RAM & disk for larger datasets. |
| **mintsdata.circ (API + Read Replica)** | 8 | 32 GB | 500 GB | Merge API hosting + dashboards here; containerize for easier deployments. |
| **www1.circ (Web Host)**    | 8    | 32 GB  | 200 GB  | Consolidate all MINTS-related web apps; migrate to Ubuntu + Nginx. |
| **mosquitto.circ (MQTT)**   | 4    | 8 GB   | 100 GB  | Upgrade Mosquitto to latest LTS, enhance TLS handling. |
| **mdash.circ (Data Processing & Monitoring)** | 20 | 64 GB | 2 TB   | Expand for InfluxDB (sensor data), Grafana (dashboards), Node‑RED (pipelines), *Prometheus* (infrastructure monitoring). |
| **io-sftp.circ (Deployment)** | 4 | 16 GB | 50 GB   | Continue GitHub → Production pipelines, add CI/CD tooling. |
| **borg.circ (Backup)**      | N/A  | N/A    | 50 TB   | Expand storage, automate full-system snapshots and off-site replication. |

### Key Improvements
- **OS Modernization**: Migrate CentOS 8 VMs to **Ubuntu Server 22.04 LTS**.
- **Database Scaling**: Increase PostgreSQL storage and RAM for high-frequency writes (sensor data).
- **Containerization**: Use **Podman or Docker Swarm** for Grafana, Node-RED, Quarto, APIs → simplifies deployments and updates.
- **Unified Web Management**: Consolidate SharedAirDFW, MINTS dashboards, and related web apps on a single containerized web host.
- **Improved Backups**: Expand `borg.circ` with **50 TB storage** and automate **daily snapshots + weekly off-site replication**.
- **Enhanced Monitoring**: Centralized **Prometheus + Grafana** stack for all VMs with alerting.

### Next Steps
1. **Assess VM Consolidation**: Identify which services can be containerized or merged (e.g., web + API hosting).  
2. **Hardware Resource Allocation**: Reallocate vCPUs/RAM from hypervisor to upgraded VMs.  
3. **Develop Migration Plan**: Stage-wise migration to Ubuntu, ensuring no downtime for SharedAirDFW and MINTS services.  
4. **Implement Unified Backups**: Standardize backups using Borg + off-site sync (cloud or secondary datacenter).  
5. **Establish Ownership**: Define service responsibilities (e.g., SharedAirDFW content vs. CIRC ops).  



**Questions:**
- **What is Mfs - mooseFS, how it works between these virtual machines**
- **Is there any connection to the IMD server? If so, how is it linked?**
- **Not all folders within mfs not accessible via IMD or MDASH**
- **Can we manage all mints services using one VM** 
