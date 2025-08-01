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

| Hostname                 |  Internal IP      | Role                                     |
|--------------------------|-------------------|------------------------------------------|
| **virsh.circ**           | 10.247.245.145    | Hypervisor (KVM host)                    |
| **psql.circ**            | 10.247.245.219    | Master PostgreSQL DB                     |
| **mintsdata.circ**       | 10.247.245.211    | Read-only DB, API, web services          |
| **www1.circ**            | 10.182.78.148     | Web host (multi-site)                    |
| **mosquitto/mqtt.circ**  | 10.247.245.206    | MQTT broker (IoT data)                   |
| **io-sftp.circ**         | 10.182.78.143     | Deployment / SFTP                        |
| **borg.circ**            | 10.182.78.141     | Backup storage                           |
| **mdash.circ**           | 10.247.245.223    | Sensor processing + Influx/Grafana/Node-RED |

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
**-Are any other SharedAirDFW services running on this server?**
**-How does this tie into our existing crontab for sensor data processing?**
**-Is there any connection to the IMD server? If so, how is it linked?** 

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

**Questions:**
**-What is Mfs- mooseFS, how it works between these virtual machines**
**-Is there any connection to the IMD server? If so, how is it linked?** 


