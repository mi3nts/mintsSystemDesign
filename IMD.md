
## IMD dependancies for MINTS 
Data from MINTS nodes is acquired through three distinct data pipelines: 
- RSYCED data from directly connected nodes
- MQTT data from directly connected nodes
- MQTT data from LoRaWAN nodes


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




