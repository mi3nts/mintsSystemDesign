# CIRC Services for MINTS 

 
## virsh.circ.utdallas.edu (VIRSH Machine)

VM?: No – This is a physical host.
OS: CentOS Linux 8.
Service: virsh 4.5.0 – This indicates it’s running KVM (Kernel-based Virtual Machine), used for hosting VMs. (It’s not Proxmox, but conceptually similar — Proxmox also uses KVM under the hood.)
Purpose:
Serves as the hypervisor host for many CIRC virtual machines.
- Storage: 90 TB.
- Hardware: Intel Storage server purchased from PSSC Labs.
- Access: csim, Steven, Gi, Stephen.
- CPU / RAM: 48 CPUs, 385 GB RAM.
- Externally Accessible?: No (internal only).


Likely runs:
- www1.circ.utdallas.edu (web host)
- mosquitto/mqtt.circ.utdallas.edu (MQTT broker)
- mintsdata.circ.utdallas.edu (PostgreSQL subscriber & API host)
- psql.circ.utdallas.edu (PostgreSQL master)
- io-sftp.circ.utdallas.edu (code deployment server)
- mdash.circ.utdallas.edu (sensor data processing)


internal IP: 10.247.245.145

## mintsdata.circ.utdallas.edu
### Overview
VM?: Yes (it’s a virtual machine).
Purpose: Serves as a read-only PostgreSQL subscriber database for the MINTS (Multimodal Intelligent Sensing) project.
Hosts web services (e.g., SharedAirDFW) built in Node.js, proxied by nginx.
Provides public-facing APIs and dashboards for air quality and sensor data.
Primary Functions:
- Data Storage: Mirrors the master database at psql.circ.utdallas.edu.
- Web Hosting: Runs Node.js apps for APIs and dashboards (HTTPS on 443 → proxied to Node.js 3000).
- Automation: Uses cron jobs (on io-sftp.circ.utdallas.edu) to pull updates from GitHub and deploy them to the website.
- mintsdata.circ.utdallas.edu is the public-facing web and database server for MINTS/SharedAirDFW.
- It hosts Node.js apps, serves APIs, and proxies through nginx, using data replicated from the master PostgreSQL at psql.circ.utdallas.edu.
- Automation for code updates is handled by a cron job on io-sftp.circ.utdallas.edu, 

mintsdata.circ.utdallas.edu is the public-facing web and database server for MINTS/SharedAirDFW.

It hosts Node.js apps, serves APIs, and proxies through nginx, using data replicated from the master PostgreSQL at psql.circ.utdallas.edu.

Automation for code updates is handled by a cron job on io-sftp.circ.utdallas.edu, which needs a long-term solution (service account for deployments).

Internal IP: 10.247.245.211


## www1.circ.utdallas.edu – Summary
### Overview
- VM?: Yes (it’s a virtual machine).
- OS: CentOS Linux 8.
- Relevant Software: nginx (used as a web server and reverse proxy).
- Backups: /var/www is mounted through IO (likely backed up by IO storage system).
- Storage: Less than 30 GB on this VM.

This server hosts multiple websites:
- sharedairdfw.com
- utdmint.info
- python.davidlary.info
- ganymededocs.circ.utdallas.edu
- mintsdata.utdallas.edu
- davidlary.info
- cisnerosres.utdallas.edu

Who has access: CIRC admins, managed via ansible.
- CPUs: 2.
- RAM: 16 GB.
- Externally Accessible?: Yes (it’s public-facing for hosting those websites).

internal IP: 10.182.78.148

## mosquitto.circ.utdallas.edu 
- hostname: mosquitto.circ.utdallas.edu
- cname: mqtt.circ.utdallas.edu
- VM?: Yes
- OS: CentOS Linux 8
- Service: Mosquitto 1.6.15 (MQTT broker)
- Storage: 30 GB disk

Special Notes:
- Requires a valid SSL certificate (mqtt.circ.utdallas.edu.crt) located at:
```/etc/pki/mosquitto/certs/```
- CPU / RAM: 1 CPU, 2 GB RAM
- Externally Accessible?: No (internal-only).

- Purpose:
This VM runs the MQTT broker for the MINTS/CIRC ecosystem.
It handles IoT sensor data publishing and subscriptions for other systems (like mintsdata and external devices), but is not public-facing.

Internal IP: 10.247.245.206

## psql.circ.utdallas.edu
- VM?: Yes
- OS: Debian GNU/Linux 10
- Service: PostgreSQL 11.9 (main master database server)
- Storage: 100 GB disk
- CPU / RAM: 2 CPUs, 16 GB RAM
- Externally Accessible?: No (internal-only)

### Purpose:
This is the primary (master) PostgreSQL database server for the MINTS ecosystem.
Other servers like mintsdata.circ.utdallas.edu act as read-only subscribers, replicating data from here.
It’s backed up regularly with borg, making it the most resilient database in this environment.

Internal IP: 10.247.245.219

## io-sftp.circ.utdallas.edu
- VM?: Yes
- OS: CentOS Linux 8
- Service: Primarily SFTP & automation (no major application services)
- Purpose: Deployment server for updating the SharedAirDFW frontend from GitHub.

Runs a cron job:
```
cd /mfs/io/circ/www/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map && \
bash update.sh >> /mfs/io/circ/www/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map/update.log 
```

This pulls code updates and deploys them automatically to the live site.
Storage: 5 GB disk.
CPU / RAM: 2 CPUs, 8.5 GB RAM.
Externally Accessible?: No (internal only).

In Simple Terms: io-sftp is a deployment automation box. It keeps SharedAirDFW updated from GitHub by running a cron-based update script. It’s small (5 GB) and has no backups or monitoring — but is critical for keeping the frontend in sync with code changes.

## borg.circ.utdallas.edu – Summary
VM?: Yes
Purpose: Backup server using Borg
Service: Borg Backup (deduplicated, compressed backups for other CIRC systems like psql.circ.utdallas.edu)
Maintenance: None scheduled (manual updates)
Backups: This is the backup host itself — it stores archives for other servers
Access: Controlled by CIRC Admins; adding users requires running Ansible

Other Notes:
It does not run application services — it’s purely for backup storage.

Used by servers like psql.circ.utdallas.edu, which explicitly states its database dumps are backed up to this host.

In Simple Terms:
borg.circ.utdallas.edu is the central backup server for the CIRC ecosystem. It stores deduplicated backups of critical systems (especially databases) using the Borg backup tool. It doesn’t host websites or apps — its sole job is to keep copies of important data safe.




----------
# CIRC Services for MINTS

## virsh.circ.utdallas.edu
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
- **Access**: csim, Steven, Gi, Stephen  
- **Externally Accessible?**: No  
- **Internal IP**: `10.247.245.145`  

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
- **Externally Accessible?**: Yes  
- **Internal IP**: `10.182.78.148`  

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
- **Externally Accessible?**: No  
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
- **Externally Accessible?**: No  

---

## borg.circ.utdallas.edu
- **VM?**: Yes  
- **Service**: Borg Backup  
- **Purpose**:  
  - **Central backup server**  
  - Stores deduplicated, compressed backups for CIRC systems (**especially `psql`**)  
- **Access**: Controlled by CIRC admins (via Ansible)  
- **Externally Accessible?**: No  

---

## Quick Reference Table

| Hostname                  | Internal IP       | Role                              | Backups       | Public? |
|--------------------------|-------------------|-----------------------------------|---------------|---------|
| **virsh.circ**           | 10.247.245.145    | Hypervisor (KVM host)            | No            | No      |
| **mintsdata.circ**       | 10.247.245.211    | Read-only DB, API, web services  | No            | Yes     |
| **www1.circ**            | 10.182.78.148     | Web host (multi-site)           | IO-mounted    | Yes     |
| **mosquitto/mqtt.circ**  | 10.247.245.206    | MQTT broker (IoT data)          | No            | No      |
| **psql.circ**            | 10.247.245.219    | Master PostgreSQL DB            | Borg backups  | No      |
| **io-sftp.circ**         | *Internal only*   | Deployment / SFTP               | No            | No      |
| **borg.circ**            | *Internal only*   | Backup storage                  | Self-backup   | No      |

---

## Infrastructure Diagram


# CIRC Services for MINTS

## virsh.circ.utdallas.edu
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
- **Access**: csim, Steven, Gi, Stephen  
- **Externally Accessible?**: No  
- **Internal IP**: `10.247.245.145`  

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
- **Externally Accessible?**: Yes  
- **Internal IP**: `10.182.78.148`  

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
- **Externally Accessible?**: No  
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
- **Externally Accessible?**: No  

---

## borg.circ.utdallas.edu
- **VM?**: Yes  
- **Service**: Borg Backup  
- **Purpose**:  
  - **Central backup server**  
  - Stores deduplicated, compressed backups for CIRC systems (**especially `psql`**)  
- **Access**: Controlled by CIRC admins (via Ansible)  
- **Externally Accessible?**: No  

---


## mdash.circ.utdallas.edu

- **VM?**: Yes  
- **OS**: Debian GNU/Linux 11  

### **Purpose**  
- Ingests raw sensor data and processes it for the **Open Storage Network (OSN)**.  
- Hosts **Podman containers** for:  
  - **InfluxDB** (time-series database for sensor data)  
  - **Grafana** (data visualization dashboards)  
  - **Node-RED** (data processing workflows and automation)  
- Supports **automated reporting** via **Quarto** (rendering and previewing automated reports).  

---

### **Network Information**
- **Internal IP**: `10.247.245.223` (on interface `enp1s0`)  
- **External/Public IP**: `129.110.247.1`  
- **Subnet**: `10.247.245.128/25`  
- **Default Gateway**: `10.247.245.129`  

---

### **Storage (from `df -h`)**
- **Local Volumes (LVM):**
  - `/home`: **1 TB** total, **390 GB used** (41%) → likely container data and user files  
- **Network-mounted storage:**  
  - `/mfs/io/groups/lary`: **14 TB total**, **10 TB used** (73%)  

---



## Quick Reference Table

| Hostname                  | Internal IP       | Role                            | Backups       | Public? |
|--------------------------|-------------------|----------------------------------|---------------|---------|
| **virsh.circ**           | 10.247.245.145    | Hypervisor (KVM host)            | No            | No      |
| **psql.circ**            | 10.247.245.219    | Master PostgreSQL DB             | Borg backups  | No      |
| **mintsdata.circ**       | 10.247.245.211    | Read-only DB, API, web services  | No            | Yes     |
| **www1.circ**            | 10.182.78.148     | Web host (multi-site)            | IO-mounted    | Yes     |
| **mosquitto/mqtt.circ**  | 10.247.245.206    | MQTT broker (IoT data)           | No            | No      |
| **io-sftp.circ**         | *Internal only*   | Deployment / SFTP                | No            | No      |
| **borg.circ**            | *Internal only*   | Backup storage                   | Self-backup   | No      |



---

### External Access:
- **SharedAirDFW API & dashboards** → `mintsdata.circ` (via nginx proxy)  
- **Websites** (`sharedairdfw.com`, etc.) → `www1.circ`  
- **MQTT (IoT)** → `mosquitto/mqtt.circ` (internal TLS)  




