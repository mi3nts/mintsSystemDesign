
# SharedAirDFW System & Script Guide

_Last Updated: July 2025_

## 1. Purpose
SharedAirDFW (https://www.sharedairdfw.com) is a **real-time air quality monitoring platform** for the Dallas-Fort Worth region. It collects data from 100+ custom-built sensors and provides interactive public visualizations via a web map and REST API.

---

## 2. System Architecture

### **Sensors**
- **Primary (Mothership) Sensors:**
  - Wired or cellular networked.
  - Send data via:
    - **MQTT →** `mqtt.circ.utdallas.edu` (Mosquitto broker).
    - **rsync →** `mintsdata.circ.utdallas.edu` (daily CSV & JSON, stored on Io).
- **LoRaWAN Tertiary Sensors:**
  - Gateways forward data to **ChirpStack (AWS)** → `mqtt.lora.trecis.cloud` → Python scripts process & save to Io.

### **Servers**
- **Mosquitto Broker:**  
  `mosquitto.circ.utdallas.edu` (CNAME: `mqtt.circ.utdallas.edu`).  
  Internal: 10.247.245.206 | External: 129.110.242.249.

- **Data Aggregation:**  
  `mintsdata.circ.utdallas.edu`.  
  Internal: 10.247.245.211 | External: 129.110.46.113.  
  - rsync service (CSV & JSON collection).  
  - PostgreSQL database (backend for API).  
  - Node.js server (serves https://sharedairdfw.com).  
  - NOAA wind data ingestion (cron).

- **Grafana & InfluxDB:**  
  `mdash.circ.utdallas.edu`.  
  Internal: 10.247.245.223 | External: 129.110.46.78.  
  - InfluxDB, Grafana, Node-RED dashboards.

- **MFS Storage Cluster:**  
  `mfsmaster.circ.utdallas.edu`.  
  - 14 TB allocated (8 TB used) for CSV & JSON files.

---

## 3. Website & API
- **Frontend:**  
  - Built with Node.js.  
  - Location:  
    ```
    /mfs/io/circ/www/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map
    ```
  - Live build served from:  
    ```
    /mfs/io/circ/www/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map/dist
    ```

- **Backend:**  
  - REST API at `api.sharedairdfw.com`.  
  - Reads from PostgreSQL database (on `mintsdata`).

---

## 4. Script Locations & How They Work

### **Frontend (Website)**
- **Main script:** `update.sh`
- **Purpose:**  
  1. Pulls the latest GitHub code.  
  2. Installs dependencies via `npm install`.  
  3. Builds the production frontend with `npm run build`.  
  4. Deploys build to `dist/`.  
- **Logs:**  
  ```
  /mfs/io/circ/www/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map/update.log
  ```

### **Backend (Data Ingestion)**
- **PostgreSQL ingestion:**  
  - `mints-sensordata-to-postgres-backend`: loads CSVs into DB.  
  - `mints-wind-data-ingestion`: pulls NOAA wind data.  
- **Location:**  
  ```
  /home/mints/WebApp/AQViz/AQBACKEND-POSTGRES/
  ```

---

## 5. How to Run the Scripts

### **1. Frontend Build (Website)**
**SSH into io-sftp server:**
```bash
ssh <your-username>@io-sftp.circ.utdallas.edu
cd /mfs/io/circ/www/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map
```

**Run deployment manually:**
```bash
bash update.sh
```

**Check logs:**
```bash
cat update.log
```

**If `.git` permission errors occur:**
```bash
sudo chown -R mints:mints .git
bash update.sh
```

---

### **2. Backend Ingestion (Wind & Sensor Data)**
**SSH into mintsdata server:**
```bash
ssh <your-username>@mintsdata.circ.utdallas.edu
cd /home/mints/WebApp/AQViz/AQBACKEND-POSTGRES/
python3 converter.py   # For NOAA wind data
```

---

## 6. Recovery Steps (If Website Goes Down)
1. **SSH into io-sftp:**
   ```bash
   ssh <user>@io-sftp.circ.utdallas.edu
   cd /mfs/io/circ/www/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map
   ```
2. **Fix permissions (if needed):**
   ```bash
   sudo chown -R mints:mints .git
   ```
3. **Rebuild:**
   ```bash
   bash update.sh
   ```
4. **Check logs:**
   ```bash
   cat update.log
   ```

---

## 7. Common Issues
- **Website downtime:** Caused by failed builds or `.git` permissions.  
- **Service fragmentation:** Components are spread across multiple servers.  
- **Firewall updates:** Occasionally block sensor data pipelines.  
- **LoRaWAN reliability:** DNS outages and firmware limits affect data flow.

---

## 8. Quick Reference
- **Frontend (Website):**  
  `/mfs/io/circ/www/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map` (io-sftp server).  
- **Backend (PostgreSQL + APIs):**  
  `/home/mints/WebApp/AQViz/AQBACKEND-POSTGRES/` (mintsdata server).  
- **MQTT Broker:**  
  `mqtt.circ.utdallas.edu` (CNAME for `mosquitto.circ.utdallas.edu`).  
- **InfluxDB + Grafana:**  
  `mdash.circ.utdallas.edu`.  

---
