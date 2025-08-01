
# SharedAirDFW System & Script Guide

## Purpose
SharedAirDFW (https://www.sharedairdfw.com) is a **real-time air quality monitoring platform** for the Dallas-Fort Worth region. It collects data from 100+ custom-built sensors and provides interactive public visualizations via a web map and REST API.

---

## System Architecture

### **Sensors**
- **Primary (Mothership) Sensors:**
  - Wired or cellular networked.
  - Send data via:
    - **MQTT →** `mqtt.circ.utdallas.edu` (Mosquitto broker).
- **LoRaWAN Tertiary Sensors:**
  - Uses LoRaWAN technology to transmit the data to LoRaWAN Gateways 
  - Gateways forward data to **ChirpStack (AWS)** → `mqtt.lora.trecis.cloud`

 ---

## **Data Conversion**

### Sensor data Ingestion
The data produced by MINTS Nodes, accessible through [sharedairdfw.com](https://www.sharedairdfw.com/), is archived in a PostgreSQL database. To ensure seamless integration with SharedairDFW, we generate structured .csv files from the two MQTT pipelines. It's important to note that, while not mandatory for SharedairDFW, we employ machine learning-based calibration techniques, utilizing a research-grade climate sensor to enhance the quality of the collected climate data. This calibrated data is also stored in the PostgreSQL database. However, it's worth mentioning that the demand for machine learning-calibrated climate data is currently unnecessary, as sharedairdfw.com does not currently display climate data from MINTS nodes.

#### MQTT data from directly connected nodes for sharedairdfw.com as well as LoRaWAN Nodes
The process of calibrating climate data, along with the steps of data cleansing and averaging, is managed by the firmware accessible on [GitHub - mi3nts/mqttLiveV3](https://github.com/mi3nts/mqttLiveV3). Within the IMD system, this repository is located on the mfs mount at /mfs/io/groups/lary/gitHubRepos/mqttLive/firmware.
Log In to IMD mintsdata.utdallas.edu
```
cd /mfs/io/groups/lary/gitHubRepos/mqttLiveV3/firmware
nohup ./runDataReaders.sh  >/dev/null 2>&1  &
```
At this point the Sensor data is properly formatted to be accepted into pstgresql DB. 

#### Live sensor data migration into PostgreSQL
[sharedairdfw.com](https://www.sharedairdfw.com/) relies on the structured data stored in CSV files, which are imported into a PostgreSQL database. This data integration process is facilitated by the firmware accessible at [GitHub - mi3nts/mints-sensordata-to-postgres-backend](https://github.com/mi3nts/mints-sensordata-to-postgres-backend). Within the IMD system, you can find this repository on the mfs mount at /mfs/io/groups/lary/mints-sensordata-to-postgres-backend.

Log In to IMD mintsdata.utdallas.edu
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

---

### Wind Data Ingestion

#### Purpose
Fetch **NOAA GFS (Global Forecast System)** wind forecast data every 6 hours, convert it into JSON format, and **store it in the SharedAirDFW PostgreSQL database** for display on the website and API.

#### How It Works
1. **Downloads** GFS forecast files from NOAA servers (`nomads.ncep.noaa.gov`).
2. **Converts GRIB2 → JSON** using `grib2json`:
   - U-component (east–west wind).
   - V-component (north–south wind).
3. **Combines** both components into `wind_data.json`.
4. **Adds metadata** (forecast time, recorded time).
5. **Inserts/updates** the `wind_data` table in PostgreSQL.
6. **Deletes old data** with `deleteOld.py`.

### One-Time Setup
```bash
ssh <your-username>@mintsdata.circ.utdallas.edu
cd /home/mints/WebApp/AQViz/AQBACKEND-POSTGRES/mints-wind-data-ingestion
python3 -m pip install requests psycopg2
cp pypsqlcon-template.py pypsqlcon.py  # Configure DB credentials
```
**Q: Why can’t I use the IMD server, and where is the process currently running? Is it been hosted on io-sftp?**

#### Manual Run
```bash
cd /home/mints/WebApp/AQViz/AQBACKEND-POSTGRES/mints-wind-data-ingestion
python3 converter.py
python3 deleteOld.py
```

#### Automating (Cron)
Edit crontab (`crontab -e`):
```
0 */6 * * * cd /home/mints/WebApp/AQViz/AQBACKEND-POSTGRES/mints-wind-data-ingestion && /usr/bin/python3 converter.py >> wind_ingestion.log 2>&1
0 */6 * * * cd /home/mints/WebApp/AQViz/AQBACKEND-POSTGRES/mints-wind-data-ingestion && /usr/bin/python3 deleteOld.py >> wind_ingestion.log 2>&1
```

---

## Rest API 

### Purpose
Provide a **REST API** combining:
- **MINTS sensor data** (calibrated air quality readings).
- **NOAA wind data** (from `mints-wind-data-ingestion`).

This powers **[https://sharedairdfw.com](https://sharedairdfw.com)** and other clients.

### How It Works
1. Runs an **Express.js server** on **port 3000**.
2. **Connects to PostgreSQL** to query wind & sensor data.
3. Serves **REST API endpoints** for:
   - NOAA wind data.
   - Sensor data (latest, by sensor, in ranges, averaged).
   - Sensor metadata (IDs, names, locations).
4. Supports **JSON responses** (and some HTML for debugging).
5. **CORS-enabled** for frontend access.

### One-Time Setup
```bash
ssh <your-username>@mintsdata.circ.utdallas.edu
cd /home/mints/WebApp/AQViz/AQBACKEND-POSTGRES/mints-noaa-api
npm install
```

**Questions:**
**- How exactly is this process currently being run?**
**- Where is it running (e.g., which server or environment)?** 

### Run the API
```bash
node index.js
```
API is now live at:
```
http://mintsdata.circ.utdallas.edu:3000
```

In production, it's proxied via Nginx as:
```
https://api.sharedairdfw.com
```

### Key API Routes

#### **Wind Data**
- `/wind_data/latest` → Latest forecast.
- `/wind_data/:recorded_time` → Forecast at a specific time.
- `/wind_data` → All forecasts.

#### **Sensor Data**
- `/sensors/list` → List of sensors.
- `/sensors/:sensor_id/latest` → Latest data for one sensor.
- `/latest/all` → Latest data for all sensors.
- `/data/:sensor_id/:start_date/:end_date` → Data for one sensor in a range.
- `/data/average/:type/:sensor_id/:start_date/:end_date/:interval?` → Averaged readings.

**Datetime format:** `YYYY-MM-DDTHH:MM:SS.00Z`

**Questions:**
**Not sure if this is correct** 

### Production Management
Usually run as a **systemd service** or via **pm2**:
```bash
pm2 start index.js --name "mints-noaa-api"
pm2 status
pm2 logs mints-noaa-api
```


## Front End - SharedAirDFW (`sharedairdfw_map`)

The **SharedAirDFW frontend** is the public-facing **interactive map and dashboard** available at [https://sharedairdfw.com](https://sharedairdfw.com).  
It is a **Single Page Application (SPA)** built using **Vue.js** and served through **Nginx** at the University of Texas at Dallas.

---

### **Purpose**
- Provide a **real-time, interactive map** of air quality data across Dallas-Fort Worth.
- Display data from:
  - **Calibrated sensor readings** (via `mints-noaa-api`).
  - **NOAA wind forecasts** (via `mints-wind-data-ingestion`).
- Serve as a **public portal** for SharedAirDFW’s datasets.

---

### **Key Technologies**
- **Vue.js**: For building the frontend SPA.
- **Nginx**: Serves the compiled static files from `/dist`.
- **Node.js API**: Consumes data from `mints-noaa-api` (proxied by Nginx).
- **Cron & Bash (`update.sh`)**: Automates updates by pulling the latest GitHub code and rebuilding the site.

---

### **Repository**
- GitHub: [mi3nts/sharedairdfw_map](https://github.com/mi3nts/sharedairdfw_map)
- Path on server:
  ```
  /home/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map
  ```

---

### **Running in Development Mode**
For local testing:
```bash
git clone https://github.com/mi3nts/sharedairdfw_map.git
cd sharedairdfw_map
npm install
npm run serve
```
- Runs a development server at `http://localhost:8080`.

---

### **Building for Production**
To build a deployable version:
```bash
npm run build
```
- Generates static files in `dist/`.

Deploy to Nginx:
```bash
rm -rf /home/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map/dist/*
cp -r dist/* /home/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map/dist/
```

---

### **Automated Updates**
**Questions:**
**- How exactly is this process currently being run?**
**- Where is it running (e.g., which server or environment)?** 
**- How does www1.circ come into the fold** 

A **cron job** on `io-sftp.circ.utdallas.edu` running on geikhans account:
- Pulls the latest changes from GitHub.
- Runs `npm run build`.
- Updates the `/dist` directory.
- Logs are stored at:
  ```
  /home/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map/update.log
  ```

Cron command:
```bash
cd /mfs/io/circ/www/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map && bash update.sh >> update.log
```

---

### **Nginx**
**Questions:**
**Not sure of how this works** 

The frontend is served by **Nginx**:
- Root directory:
  ```
  /home/mints/WebApp/AQViz/AQFRONTEND/sharedairdfw_map/dist
  ```
- To restart after changes:
```bash
sudo systemctl restart nginx
```

---

### **Data Flow**
1. **Sensors** collect air quality data → stored in PostgreSQL (via `mints-sensordata-to-postgres-backend`).
2. **Wind data** fetched via `mints-wind-data-ingestion` → stored in PostgreSQL.
3. **mints-noaa-api** exposes sensor & wind data through REST endpoints.
4. **sharedairdfw_map** fetches from these endpoints and displays data on the public map.

---

### **Common Troubleshooting**
- **Frontend not updating?**
  - Check `update.log` for cron errors.
  - Ensure `.git` permissions allow the `mints` user to pull updates.
- **API data not loading?**
  - Confirm `mints-noaa-api` is running and accessible.
- **Website down?**
  - Check Nginx status and F5 load balancer routing.

---



