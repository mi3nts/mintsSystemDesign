
# MINTS System Design

![MINTS Diagram](https://github.com/mi3nts/mintsSystemDesign/blob/main/res/mintsSystemDesign.png?raw=true)

---

## 🖥️ IMD (Dell R710)

### 🔹 Hostname
- **FQDN**: `mintsdata.circ.utdallas.edu`

### 🔹 Operating System
- **OS**: CentOS Linux 7

### 🔹 Hardware Specs
- **CPU**: 8 cores (16 threads)  
- **RAM**: 46 GB  
- **Storage**: 5 TB  

### 🔹 Network Info
- Server IP: `10.247.238.16` Local IP inside UTD network                            
- Host IP  : `10.247.245.211` Internal IP bound to 

### 🔹 Access
- **User**: `MINTS User`  
- **Method**: SSH with key-based authentication

### 🔹 Software
- **npm**: v6.14.5

---

## 📊 MDASH

### 🔹 Hostname
- **FQDN**: `mdash.circ.utdallas.edu`

### 🔹 Services
- InfluxDB, Grafana, Node-RED
- OSN syncs supported

### 🔹 Network Info
- Server IP: `10.247.238.16` Local IP inside UTD network                            

### 🔹 Hardware Specs
- **OS**: Debian GNU/Linux 11  
- **CPU**: 4 cores  
- **RAM**: 8 GB  
- **Disk**: 1 TB  

- **OIT Support**: Stephen Goss

---

## 🌐 www1.circ.utdallas.edu

### 🔹 Purpose
Hosts websites:
- sharedairdfw.com  
- utdmint.info  
- python.davidlary.info
- ganymededocs.circ.utdallas.edu  
- mintsdata.utdallas.edu  
- davidlary.info  
- cisnerosres.utdallas.edu  

### 🔹 Network Info
- Server IP: `10.182.78.148` Local IP inside UTD network        

### 🔹 Hardware Specs
- **CPU**: 1 core  
- **RAM**: 16 GB  
- **Disk**: < 30 GB  
- **Externally Accessible**  
- **OIT Support**: Stephen Goss

---

## 🧪 mosquitto.circ.utdallas.edu / mqtt.circ.utdallas.edu

### 🔹 Network Info
- Server IP: `10.247.245.206` Local IP inside UTD network      

### 🔹 Specs
- **CPU**: 1 core  
- **RAM**: 2 GB  
- **Disk**: 30 GB  
- Requires: `mqtt.circ.utdallas.edu.crt` SSL certificate in `/etc/pki/mosquitto/certs/`  
- **OIT Support**: Stephen Goss

---

## 📁 mintsdata.circ.utdallas.edu
- Shared Air DFW API
- https://mintsdata.circ.utdallas.edu/info
  
### 🔹 Network
- Server IP: `10.247.245.211` Local IP inside UTD network
  
  
---

## 🐘 psql.circ.utdallas.edu

### 🔹 Specs
- **OS**: Debian GNU/Linux 10  
- **PostgreSQL**: v11.9  
- **CPU**: 2 cores  
- **RAM**: 8.5 GB  
- **Disk**: 100 GB  

### 🔹 Network
- Server IP: `10.247.245.219` Local IP inside UTD network
  
---

## 🔄 io-sftp.circ.utdallas.edu

### 🔹 Specs
- **CPU**: 2 cores  
- **RAM**: 8.5 GB  
- **Disk**: 5 GB  
- **Mounted**: `/mfs/io/circ/www/mints`

### 🔹 Function
Auto-updates SharedAirDFW map from GitHub:
```bash
cd /mfs/io/circ/www/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map && bash update.sh >> update.log
```

Comments - I can access this through UTD Credentials 


### 🔹 Network
- Server IP: `10.247.245.208` Local IP inside UTD network
  

---

## 💾 borg.circ.utdallas.edu

- **Likely used for backups**
- Server IP: `10.182.78.141` Local IP inside UTD network

---

## 🖧 virsh.circ.utdallas.edu

### 🔹 Specs
- **CPU**: 48 cores  
- **RAM**: 385 GB  
- **Disk**: 90 TB  
- **Server**: Intel storage server (PSSC Labs)

### 🔹 Network
- Server IP: `10.247.245.145` Local IP inside UTD network


### 🔹 Hosts the following VMs:
- www1.circ.utdallas.edu  
- mosquitto.circ.utdallas.edu / mqtt.circ.utdallas.edu  
- mintsdata.circ.utdallas.edu  
- psql.circ.utdallas.edu  
- io-sftp.circ.utdallas.edu  
- mdash.circ.utdallas.edu  

---

## ☁️ Amazon Services

### 🔹 MINTS - PROD (ChirpStack)
- **Hostname**: `lora-large-1.trecis.cloud`  
- **IP Address**: `34.204.203.10`  

### 🔹 MINTS - PROD (S3/OSN Syncs)
- **OSN Endpoint**: `https://141.142.142.3/`
