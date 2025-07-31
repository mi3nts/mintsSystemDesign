
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
- **LoRaWAN Tertiary Sensors:**
  - Uses LoRaWAN technology to transmit the data to LoRaWAN Gateways 
  - Gateways forward data to **ChirpStack (AWS)** → `mqtt.lora.trecis.cloud`
 


## **Data Conversion**
## SharedAirDFW Support 
The data produced by MINTS Nodes, accessible through [sharedairdfw.com](https://www.sharedairdfw.com/), is archived in a PostgreSQL database. To ensure seamless integration with SharedairDFW, we generate structured .csv files from the two MQTT pipelines. It's important to note that, while not mandatory for SharedairDFW, we employ machine learning-based calibration techniques, utilizing a research-grade climate sensor to enhance the quality of the collected climate data. This calibrated data is also stored in the PostgreSQL database. However, it's worth mentioning that the demand for machine learning-calibrated climate data is currently unnecessary, as sharedairdfw.com does not currently display climate data from MINTS nodes.

### MQTT data from directly connected nodes for sharedairdfw.com as well as LoRaWAN Nodes
The process of calibrating climate data, along with the steps of data cleansing and averaging, is managed by the firmware accessible on [GitHub - mi3nts/mqttLiveV3](https://github.com/mi3nts/mqttLiveV3). Within the IMD system, this repository is located on the mfs mount at /mfs/io/groups/lary/gitHubRepos/mqttLive/firmware.
```
cd /mfs/io/groups/lary/gitHubRepos/mqttLiveV3/firmware
nohup ./runDataReaders.sh  >/dev/null 2>&1  &
```

At this point the Sensor data is properly formatted to be accepted into pstgresql DB. 


### Live data migration into PostgreSQL
[sharedairdfw.com](https://www.sharedairdfw.com/) relies on the structured data stored in CSV files, which are imported into a PostgreSQL database. This data integration process is facilitated by the firmware accessible at [GitHub - mi3nts/mints-sensordata-to-postgres-backend](https://github.com/mi3nts/mints-sensordata-to-postgres-backend). Within the IMD system, you can find this repository on the mfs mount at /mfs/io/groups/lary/mints-sensordata-to-postgres-backend.
```
cd /mfs/io/groups/lary/mints-sensordata-to-postgres-backend
top | grep node
```
At this poin you should see something similar to the following 
```
 1434 mints     20   0  924556  63372  15816 S   2.3  0.1   2:59.65 node                                                      1434 mints     20   0  924556  63372  15816 S   0.3  0.1   2:59.66 node                                                      1434 mints     20   0  924556  63964  15816 S   2.3  0.1   2:59.73 node       
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

Shared Air DFW also contains Wind Data Submitted by NOAH 

- **Data Aggregation:**  
  `mintsdata.circ.utdallas.edu`.  
  Internal: 10.247.245.211 | External: 129.110.46.113.  
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
