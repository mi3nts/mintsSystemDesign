# CIRC Services for MINTS 

 
## virsh.circ.utdallas.edu (VIRSH Machine)

VM?: No – This is a physical host.
OS: CentOS Linux 8.
Service: virsh 4.5.0 – This indicates it’s running KVM (Kernel-based Virtual Machine), used for hosting VMs. (It’s not Proxmox, but conceptually similar — Proxmox also uses KVM under the hood.)
Purpose:
Serves as the hypervisor host for many CIRC virtual machines.

Likely runs:
- www1.circ.utdallas.edu (web host)
- mosquitto/mqtt.circ.utdallas.edu (MQTT broker)
- mintsdata.circ.utdallas.edu (PostgreSQL subscriber & API host)
- psql.circ.utdallas.edu (PostgreSQL master)
- io-sftp.circ.utdallas.edu (code deployment server)
- mdash.circ.utdallas.edu (sensor data processing)

- Storage: 90 TB.
- Hardware: Intel Storage server purchased from PSSC Labs.
- Access: csim, Steven, Gi, Stephen.
- CPU / RAM: 48 CPUs, 385 GB RAM.
- Externally Accessible?: No (internal only).


### mintsdata.circ.utdallas.edu

#### Overview
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


### www1.circ.utdallas.edu – Summary
#### Overview
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





