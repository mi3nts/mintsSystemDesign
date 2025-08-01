
# Comprehensive Summary of MINTS-AI Meeting  
**Date:** April 27, 2023  

## Purpose of the Meeting
- Conduct a **discovery session** to review the current system for sensing infrastructure.
- **Document existing architecture** and outline next steps for migrating to a **robust, AWS-based infrastructure**.
- Plan for **regular weekly cadence meetings** with key members for progress tracking.

---

## Project Overview
- **Objective:** "Sensing in service of society" using multi-modal data from:  
  - Satellite sensors  
  - City-wide live environmental sensors  
  - Wearable sensors  
  - Robotic teams  
- **Website:** [SharedAirDFW](https://sharedairdfw.com) — live public-facing portal for air quality and related environmental data.  
- **Key features:**  
  - Real-time air quality (particulates, CO₂, gases, meteorological variables).  
  - Acoustic monitoring to detect bird calls for ecological studies.  
  - Community engagement: sensors deployed with neighborhoods and cities (e.g., City of Richardson).  
  - Partnerships with organizations (e.g., Dallas County using the data).  

---

## System Architecture (Current)
1. **LoRaWAN-based Sensor Network:**  
   - Sensors clustered with hubs using wired/cellular connectivity.  
   - Deployed on **AWS** with **ChirpStack** servers for LoRaWAN data management.  

2. **MQTT Data Transmission:**  
   - Data sent from field devices to **AWS**, then via VPN to UTD’s **ARDC** data center.  
   - **Mosquitto MQTT server** (`mqtt.circ.utdallas.edu`).  

3. **File Storage:**  
   - Data written to **MooseFS POSIX storage** (4.6 PB total, 1.5 PB used).  

4. **Web Server:**  
   - **Node.js application** running `sharedairdfw.com`.  
   - Served via **F5 load balancer** and **NGINX** on VM (`mintsdata.circ.utdallas.edu`).  

5. **Visualization:**  
   - **Grafana** dashboards for real-time visualization.  
   - **Node-RED** for zero-code parsing and InfluxDB ingestion.  

---

## Key Issues & Challenges
- **Aging infrastructure** leading to **periodic outages**.  
- **Difficult recovery process** (only a few people know the restart sequence).  
- **Website updates** sometimes break functionality.  
- **Opaque system architecture** — lacks documentation and clear operational flow.  

---

## Goals & Action Items
1. **Document the Current Architecture:**  
   - Create an architecture diagram of all components.  
   - Establish a "Center of Excellence" process for documentation.  

2. **Design AWS Migration Strategy:**  
   - Use **AWS IoT** for managed MQTT.  
   - **Amazon Managed Grafana** for visualization.  
   - Host web application on **EC2** instances.  
   - Ensure **incremental migration** to avoid service downtime.  

3. **Long-term Vision:**  
   - Create a **scalable, low-maintenance infrastructure** to support community and research needs.  
   - **Broader applicability** for other UTD researchers working on IoT and environmental sensing.  

---

**Next Steps:**  
- Schedule **working sessions** for architecture documentation.  
- **Set up AWS IoT endpoint** and test with lab sensors.  
- **Gradual migration** to AWS-hosted infrastructure while keeping production running.  



# Comprehensive Summary of MINTS-AI Meeting  
**Date:** May 11, 2023  

---

## Purpose of the Meeting
- **Weekly progress check** on MINTS-AI AWS migration efforts.  
- **Clarify account access, roles, and responsibilities** for the MINTS-AI project.  
- Plan **next steps** for testing AWS IoT Core and developing architecture documentation.  

---

## Key Discussion Points

### 1. **AWS Account Setup & Access**  
- A **sandbox AWS account** has been created for MINTS-AI.  
- Plan to **create a dedicated production account** once architecture is finalized.  
- Access issues were reported; the **Cloud team will resolve** and ensure **mints-ai team gain access**.  
- **No shared service accounts** for security reasons — individuals will have role-based access.

### 2. **Training & Enablement**  
- **Cloud 101 training session** planned for EC2, S3, storage, networking, and security basics.  
- **Workshop with AWS IoT SME** in progress (immersion day or 3-hour session) to guide pipeline design.  
- Team encouraged to **self-learn AWS IoT Core** using exercises and tutorials.  

### 3. **Architectural Planning**  
- **Discovery Form** to gather current-state system information (AS-IS documentation).  
- Next step: **Prepare architecture diagrams**:  
  - **Current State:** Existing infrastructure (on-prem MQTT, ChirpStack, web apps, dashboards).  
  - **Future State:** Target AWS architecture with IoT Core, Analytics, and supporting services.  

### 4. **Immediate Technical Goals**  
- **Get a sensor (physical or virtual)** sending data to **AWS IoT Core**.  
- **Pull data** from IoT Core to an **EC2 instance** (POSIX file system) for use with the Node.js application.  
- Identify pain points and **define migration roadmap** for infrastructure.  

---

## Action Items
1. **Resolve AWS access issues** 
2. **Fill out Discovery Form** for current infrastructure documentation.  
3. **Schedule Cloud 101 training** for MINTS-AI team.  
4. **Engage AWS IoT SME** for a deep-dive workshop.  
5. **Prepare Current and Future State Architecture Diagrams** within two weeks.  
6. **Set up virtual sensor pipeline**: Send data to AWS IoT Core and retrieve it via EC2.  

---

## Key Decisions
- Migration will be **incremental** with testing in sandbox environments before production.  
- **Weekly Thursday check-ins** to track progress and present updates.  

---

## Next Steps
- Verify AWS access for key team members.  
- Begin **current-state documentation** and architecture diagrams.  
- Schedule **IoT Core pipeline setup** (virtual sensor test).  
- Plan and confirm **training/workshop sessions** for the coming weeks.  



# Comprehensive Summary of MINTS-AI Meeting  
**Date:** May 25, 2023  


## Purpose of the Meeting
- **Discuss architecture planning** for AWS migration of MINTS-AI infrastructure.  
- **Review feedback from AWS IoT training workshop**.  
- **Address storage and cost concerns** for long-term sensor and research data.  
- **Plan for architectural diagrams and a proof-of-concept (POC)** to estimate costs and validate feasibility.  

---

## Key Discussion Points

### 1. **Feedback from Training**
- Team found the **AWS IoT Core training valuable** — especially the simplified device registration and pipeline features.  
- **Custom Grafana plugins** needed for their use case; team prefers to **host Grafana in a containerized environment** with S3 backend for cost efficiency.  

### 2. **Storage & Cost Concerns**
- **Long-term storage cost** is the biggest concern.  
- Current dataset: **~8 TB**, projected growth to **~20 TB within a year** as sensors and remote sensing integrations expand.  
- **Key challenge:** AWS S3 and data access (egress) fees could become prohibitive, especially with iterative ML training and external community queries.  
- **Critical concern:** Ensuring **data persistence** even if funding lapses (avoiding risk of data loss).  

### 3. **Proposed Storage Strategies**
- **Hybrid storage model:**  
  - Keep **primary data in AWS S3** (data lake).  
  - Maintain **local on-premises copies** for high-volume processing (e.g., ML model training).  
- **Cost optimization approaches:**  
  - **Multi-account architecture** to isolate datasets.  
  - **Use of reserved instances** and **savings plans** for compute (up to 70% savings).  
  - **Tiered storage for S3** (move older/less-accessed data to cheaper tiers).  
  - **Use of Fargate/ECS for containers** to reduce infrastructure management costs.  

### 4. **IoT & Data Pipeline Plans**
- **Replace ChirpStack with AWS IoT Core** for sensor data ingestion.  
- Support for **frequent high-resolution data** (some devices reporting every 1–5 seconds).  
- **Leverage AWS SiteWise** for processing high-frequency sensor data at the edge.  
- Use **time-series database (TSDB)** and **S3-based data lake** for warm/cold storage.  
- Investigate **querying strategies** to balance cost vs. performance (frequent vs. archival data access).  

### 5. **ML & Data Analytics Workflows**
- ML model training currently done on **HPC clusters** using Julia and other open-source tools.  
- Consider **using AWS SageMaker** for in-cloud training to reduce data transfer costs.  
- Need to ensure **open, reproducible workflows** for the academic community.  

---

## Action Items
1. **Develop architectural diagrams** for the proposed AWS pipeline (IoT Core, S3, Grafana, SiteWise, ECS/Fargate).  
2. **Run cost estimates** for multiple scenarios:  
   - Full cloud storage and compute.  
   - Hybrid (AWS + on-prem storage).  
3. **Set up a Proof-of-Concept (POC)** with limited sensors to measure actual costs.  
4. **Explore OIT funding support** for long-term cloud costs (discussion with Frank).  
5. **Develop cost monitoring & alerting** to prevent unexpected budget overruns.  
6. **Prepare cost estimates for community data access** (queries from external users).  

---

## Key Decisions
- **Hybrid storage approach** is preferred for cost efficiency and risk management.  
- **Incremental migration**: Begin with a POC using a subset of sensors to estimate real-world AWS costs.  
- **Local syncing of raw data**: Maintain offline copies to reduce repetitive egress fees.  
- **AWS IoT Core will replace ChirpStack** as the primary ingestion platform.  

---

## Next Steps
- **Prepare detailed architecture diagram** (by next week).  
- **Develop POC** with a limited set of sensors in AWS IoT Core and Grafana.  
- **Estimate costs** for 100+ sensors scaling to thousands.  
- **Set up multi-tier storage strategy** (warm vs. cold data).  
- **Schedule follow-up** with AWS experts for architectural review and cost modeling.  



# Comprehensive Summary of MINTS-AI Meeting  
**Date:** June 01, 2023  
---

## Purpose of the Meeting
- **Establish clear path for AWS migration**, including cost modeling and funding strategies.  
- **Clarify current infrastructure size, budget, and growth projections.**  
- **Plan for architectural documentation and proof-of-concept (POC).**  

---

## Key Discussion Points

### 1. **Budget & Funding**
- **Frank has approved funding** for maintaining the current infrastructure through **June 30, 2024** (possibly extendable).  
- Current AWS spend: **~$1,700/month** (includes EC2, storage, and related resources).  
- Goal: **Re-architect system** to either maintain or reduce monthly spend while increasing stability.  
- Need to model **costs for scaling** to 10–100+ new sensor locations.  

### 2. **Data Storage & Growth**
- Current data volume: **~8 TB**, projected to grow to **~20 TB within a year**.  
- Data growth is **linear** with the number of sensors and their reporting frequency.  
- **All historical data** must remain accessible (no purging), as it supports community portals and long-term environmental analysis.  

### 3. **Proposed Storage & Architecture Adjustments**
- **AWS S3** identified as primary storage solution: estimated **~$400/month for 20 TB** (S3 Standard).  
- **Explore cost optimization** using **S3 infrequent access** for older data (Glacier not feasible due to frequent access needs).  
- **Node.js website & Grafana dashboards** currently rely on POSIX and InfluxDB.  
- Plan to **migrate dashboards to managed cloud services** and potentially **containerize** them (e.g., AWS LightSail, ECS).  
- Investigate **automatic data tiering** for cost efficiency.  

### 4. **Proof-of-Concept (POC) Plan**
- **Develop POC** using **a subset of IoT devices** in AWS IoT Core.  
- Leverage **POC credits** (AWS account team assisting) to test architecture and gather **real cost metrics**.  
- Compare **re-architected solution** vs. current hybrid on-prem/AWS setup.  

### 5. **Long-Term Strategy**
- UTD aiming to **invest in new cyberinfrastructure** (on-prem) as part of a proposal with UT system schools.  
- If successful, future data hosting could shift to **UTD-supported infrastructure** while keeping AWS for IoT pipeline and scaling.  

---

## Action Items
1. **Document current micro-architecture**   
2. **Request POC credits** for AWS testing 
3. **Develop POC pipeline** for subset of sensors in AWS.  
4. **Create cost models** for scaling (10–100+ sensors, storage growth).  
5. **Evaluate Node.js & Grafana migration** options (managed services, containerization).  
6. **Assess backup strategies & SLAs** for data in AWS.  

---

## Key Decisions
- **Re-architect for cost efficiency**: Maintain or reduce current spend while improving reliability.  
- **All data will remain online**: No down-sampling or purging, but tiering options for older data will be explored.  
- **Incremental migration**: POC first, then phased scaling.  

---

## Next Steps
- **Deliver architecture diagram** of current system (by next week).  
- **Launch POC** with AWS IoT Core and limited devices.  
- **Run cost estimation models** for storage, compute, and scaling.  
- **Plan for extended AWS IoT workshop** (week of June 12).  


# Comprehensive Summary of MINTS-AI Meeting  
**Date:** June 08, 2023  

---

## Purpose of the Meeting
- **Review current system architecture** for MINTS-AI infrastructure.  
- **Identify cost-saving opportunities** in the AWS sandbox environment.  
- **Plan for architecture diagram creation** and next steps for AWS IoT Core integration.  

---

## Key Discussion Points

### 1. **Current System Architecture**
Chris provided a detailed breakdown of the **current MINTS-AI infrastructure**:  
- **Sensors in the field:**  
  - **Mothership devices** (wired or cellular, AT&T FirstNet) for primary data collection.  
  - **Tertiary LoRaWAN sensors** (clusters of ~10 per mothership).  
- **Data Flow:**  
  - **MQTT pipeline**: Sensors → `mqtt.circ.utdallas.edu` (Mosquitto server on Proxmox).  
  - **LoRaWAN pipeline**: Tertiary sensors → ChirpStack cluster (AWS) → MQTT broker.  
  - **Direct Connect** now used instead of VPN for AWS-to-UTD communications.  
- **On-Premises Infrastructure:**  
  - **Proxmox cluster** hosting:  
    - `mosquitto.circ.utdallas.edu` (MQTT broker).  
    - `mintsdata.circ.utdallas.edu` (Node.js website, rsync services, cron jobs).  
    - `mdash.circ.utdallas.edu` (Grafana dashboards).  
  - **MooseFS cluster (“EO”)** for POSIX storage of sensor data (8–20 TB).  
- **Cron Jobs & Scripts:**  
  - NOAA wind data fetcher.  
  - Website rebuild and Git pull scripts.  
  - Python service to continuously convert MQTT data to CSV for storage.  
- **PostgreSQL database** used for website sensor data.  

### 2. **AWS Sandbox & Cost Reduction**
- Identified **5 EC2 instances** in AWS ChirpStack cluster; at least **2 can be decommissioned** to cut costs.  
- Goal: **Reduce HPC sandbox costs** by optimizing AWS resources.  

### 3. **Next Steps for IoT Core**
- Plan to **test IoT Core with 1–2 devices** on the MINTS AWS account.  

---

## Action Items
1. **Draft visual architecture diagram** 
2. **Chris to review/update** the architecture diagram and ChirpStack configuration.  
3. **Decommission unused EC2 instances** in ChirpStack AWS cluster (Chris).  
4. **Verify permissions** for AWS IoT Core device setup (Sahil & Korki).  
5. **Begin adding 1–2 devices** to AWS IoT Core as a test.  
6. **Share meeting recording** with AWS IoT experts for review.  

---

## Key Decisions
- **Use Direct Connect** (instead of VPN) for AWS-UTD communications.  
- **Reduce AWS EC2 footprint** in ChirpStack cluster for cost savings.  
- **Collaborative approach** for creating architecture diagram (draft by team, refined by Chris).  

---

## Next Steps
- **Architecture diagram draft** by next session.  
- **AWS IoT Core pilot** with 1–2 devices.  
- **Cost optimization**: Decommission unused EC2 instances in AWS.  
- **Schedule extended IoT architecture workshop** with AWS experts.  



# Comprehensive Summary of MINTS-AI Meeting  
**Date:** June 22, 2023  

---

## Purpose of the Meeting
- **Clarify AWS IoT Core configuration** and troubleshoot gateway/device setup.  
- **Discuss cost implications** of message frequency and IoT Core pricing.  
- **Plan next steps** for architecture diagrams and re-architecture options.  

---

## Key Discussion Points

### 1. **Scientific Justification for High-Frequency Data**
- **Goal:** Characterize **small-scale temporal variability** of atmospheric measurements.  
- **Temporal variograms** show that **1-minute resolution is insufficient**; **sub-minute data** needed for representativeness uncertainty calculations.  
- **High-frequency sampling** critical for characterizing mixing barriers and system shocks.  

### 2. **Current Data Flow**
- **LoRaWAN devices** → **AWS ChirpStack cluster** → **Direct Connect** → **ARDC (UTD)** → **MooseFS & S3 (OSN)**.  
- **Directly wired sensors** send data via **MQTT** to `mqtt.circ.utdallas.edu`.  
- **Open Storage Network (OSN)** buckets used for **long-term, large-scale storage** of processed data.  
- **ChirpStack**: Currently runs on **5 EC2 instances** using **I/O-optimized storage**, costing ~$800/month (major cost driver).  

### 3. **AWS IoT Core Integration**
- **Dragino LG16 & LPS8 V2 gateways** being configured for AWS IoT Core.  
- **Troubleshooting:**  
  - Gateway not recognized due to **missing CUPS (Configuration and Update Server) private key**.  
  - **AWS IoT experts** provided updated workshop links and Python-based Lambda parser templates for packet decoding.  
  - **Lambda functions** will **decode Base64-encoded payloads**, republish parsed data to IoT Core topics, and forward to databases.  

### 4. **Cost Modeling & Pricing**
- **AWS IoT Core pricing:**  
  - **$1 per million MQTT messages**.  
  - **$2.30 per million LoRaWAN messages** (higher due to combined services).  
  - **Connection cost:** $0.08 per million minutes of connection.  
- **Message size:** 1 message = **5 KB chunk**; larger payloads split into multiple messages.  
- **Projected message rates:**  
  - **LoRaWAN devices:** 1–2 messages per 10 seconds.  
  - **Current deployment:** ~50 devices; **planned scale:** 150–200 devices.  
  - **Direct-wired sensors:** Higher frequency (1 message/second) but remain on-prem (no AWS charges).  
- **Next step:** AWS team (Ryan & Rob) to **run cost estimates** for scaling scenarios (current + 200 devices).  

### 5. **Re-Architecture Options**
- **Option 1:** **Add resilience** to current architecture (hybrid on-prem + AWS).  
- **Option 2:** **Fully migrate** to AWS services (IoT Core, EFS/S3, managed Node.js).  
- **Option 3:** **Rebuild with on-prem resources** for cost savings and control.  

### 6. **Grafana Hosting**
- Discussion of **AWS-hosted Grafana** vs **self-hosted customized Grafana**.  
- Decision: **Remain self-hosted** for full customization capabilities.  

---

## Action Items
1. **Troubleshoot IoT Core gateway/device pairing**  
2. **Develop architecture diagrams:**  
   - **Current state:** Draft by MINTS team using draw.io.  
   - **Proposed AWS architecture:** To be prepared by AWS team.  
3. **Run detailed cost estimates** for IoT Core vs ChirpStack
4. **Evaluate Lambda-based parsers** for payload decoding and integration with IoT Core pipelines.  
5. **Schedule follow-up session** (Tuesday) for architecture review and cost modeling.  

---

## Key Decisions
- **LoRaWAN devices** will migrate to **AWS IoT Core** for cost efficiency and high availability.  
- **Direct-wired MQTT devices** will **remain on-prem**.  
- **ChirpStack cluster** likely to be **phased out** if IoT Core proves cost-effective.  
- **Self-hosted Grafana** to remain for customization flexibility.  

---

## Next Steps
- **AWS IoT Core gateway fix** (CUPS key + connection validation).  
- **Prepare ballpark cost calculations** for current and scaled deployments.  
- **Complete architecture diagrams** for review in the next session.  
- **Plan migration strategy** for LoRaWAN devices (POC first, then phased rollout).  


# Comprehensive Summary of MINTS-AI Meeting  
**Date:** June 29, 2023  

--- 

## Purpose of the Meeting
- **Review progress on system architecture diagram** for MINTS-AI infrastructure.  
- **Validate AWS IoT Core proof-of-concept (POC)** for sensor data ingestion.  
- **Discuss database and web hosting migration plans** for cost reduction and resiliency.  

---

## Key Discussion Points

### 1. **Architecture Diagram Progress**
- A **draft architecture diagram** was presented by the MINTS team showing:  
  - **Primary & secondary sensors (motherships)** → **MQTT** → `mqtt.circ.utdallas.edu`.  
  - **R-Sync pipeline** from sensors → `mintsdata.circ.utdallas.edu` → **MooseFS storage**.  
  - **Grafana dashboards** and **Node-RED** (pending integration in diagram).  
- Chris confirmed the **data flow and system structure** were correctly represented.  
- **Next steps:** Add remaining components (Node-RED → InfluxDB → Grafana).  

### 2. **AWS IoT Core Proof-of-Concept**
- POC validated **connectivity for one sensor** to IoT Core via Dragino gateway.  
- **Next steps:**  
  - **Expand testing** to multiple sensors.  
  - **Set up IoT rules** to forward processed data to **S3 (Athena)** for analytics.  

### 3. **Cost Estimates**
- **AWS IoT Core ingestion for 150 devices:**  
  - ~518,400 messages per device/month (~0.2 messages/sec).  
  - Estimated **<$1,000/month** (approx. $700–800).  
  - Message size: **<5 KB** → within single-message pricing tier.  
- **Storage costs:** To be calculated separately for **S3** and **Aurora (for PostgreSQL migration)**.  

### 4. **PostgreSQL Database Migration**
- Current **PostgreSQL database** serves **SharedAirDFW** website (sensor maps, wind data, recent values).  
- **Proposed migration:** Move to **AWS Aurora** for improved resilience and management.  
- **Data retention:**  
  - Retain **recent data (7–30 days)** in Aurora for quick access.  
  - Full historical data stored in **Grafana/InﬂuxDB** and **OSN S3 buckets**.  

### 5. **R-Sync & Data Redundancy**
- **R-Sync pipeline** currently ensures **packet recovery** when network outages occur.  
- Needed for **direct-wired sensors**, but **not required for LoRaWAN devices** (no backfill mechanism).  

### 6. **Re-Architecture Goals**
- **Highlight current hosting locations:**  
  - **On-prem:** Grafana, Node.js, InfluxDB, MooseFS storage.  
  - **AWS:** ChirpStack (to be replaced with IoT Core).  
  - **OSN:** Long-term S3-based data storage.  
- **AWS team to propose a fully managed architecture** replacing on-prem components with AWS equivalents.  

### 7. **Resiliency & Public Access**
- **SharedAirDFW website** requires **high availability** to avoid public-facing outages.  
- Goal: Ensure **redundancy and failover** for website and APIs.  

---

## Action Items
1. **Finalize architecture diagram** 
2. **Expand IoT Core POC** to more sensors, with rules to push parsed data to S3 (Athena).  
3. **Run cost estimates** for:  
   - Aurora PostgreSQL migration (including backups & failover).  
   - Long-term S3 storage for high-volume data.  
4. **Determine retention policy** for PostgreSQL (7–30 days for website vs full historical in S3/Grafana).  
5. **Develop AWS-based architecture proposal** covering all components (database, website, dashboards).  

---

## Key Decisions
- **LoRaWAN ingestion will fully migrate to AWS IoT Core** (deprecating ChirpStack).  
- **PostgreSQL database** to be migrated to **AWS Aurora** with reduced data retention.  
- **SharedAirDFW portal** requires high-resiliency setup to support public access.  

---

## Next Steps
- Complete **full system architecture diagram** by early next week.  
- **Review POC cost estimates** and adjust based on message scaling.  
- **Draft AWS-based re-architecture proposal** (AWS team).  


# Comprehensive Summary of MINTS-AI Meeting  
**Date:** July 06, 2023  

---

## Purpose of the Meeting
- **Review AWS IoT-based architecture draft** for MINTS-AI infrastructure.  
- **Discuss Node-RED vs. AWS-native approaches** for data processing.  
- **Plan proof-of-concept (POC) tests** for high-frequency wired sensors.  
- **Evaluate containerization strategy for Node.js website and Grafana.**  

---

## Key Discussion Points

### 1. **Proposed AWS IoT Architecture**
- **Sensors → AWS IoT Core:** Raw payloads from LoRaWAN and wired sensors ingested.  
- **IoT Rules & Lambda Functions:**  
  - Parse payloads into structured formats.  
  - Average **8 ms execution time** per function (low-cost).  
- **Storage:**  
  - Parsed data stored in **S3 data lake** (long-term).  
  - **Aurora (serverless)** for recent (30-day) data serving the website.  
- **Visualization & Web:**  
  - **Grafana** connected to S3 via Athena plugin (self-hosted on EC2 or Fargate for custom plugins).  
  - **SharedAirDFW Node.js website**: Evaluate hosting on **ECS Fargate** vs. EC2 (cost vs. management trade-offs).  

### 2. **Node-RED Discussion**
- **Current Role:**  
  - Subscribes to **two MQTT streams** (LoRaWAN + direct-wired sensors).  
  - **Parses byte arrays** and injects structured data into a time-series DB for Grafana.  
- **AWS-native Alternative:**  
  - Replace Node-RED with **IoT Core rules + Lambda parsing**.  
  - Pros: **Resiliency, reduced DevOps, improved security** (no Node Package Manager risks).  
  - Cons: Node-RED is **low-code and easy to maintain** for the research team (no need for multiple programming languages).  
- **Hybrid Option:**  
  - Keep Node-RED for **wired high-frequency sensors**, move LoRaWAN devices to IoT Core.  
  - Evaluate incremental migration.  

### 3. **High-Frequency Wired Sensors**
- **Cost Concerns:** Wired sensors produce **5–20x more data** than LoRaWAN devices.  
- **Estimates:** LoRaWAN (150 devices) ≈ **$1,000/month**. Wired sensors could add **5–20x** that cost.  
- **Recommendation:** Run **POC tests** to measure actual cost for wired sensor ingestion.  

### 4. **Website & Grafana Containerization**
- **SharedAirDFW Node.js website:**  
  - Plan to **containerize using Docker** and deploy to **ECS Fargate** for cost reduction and automatic scaling.  
- **Grafana:**  
  - Must remain **self-hosted** due to custom plugins.  
  - **Containerized deployment** recommended for resiliency (e.g., Fargate).  

### 5. **Data Lake Strategy**
- **S3 data lake** is the **primary storage** for all parsed sensor data (scalable, low-cost).  
- **Athena** allows querying directly from S3 for Grafana dashboards and research needs.  
- **Benefit:** Easily extract and migrate data (e.g., to on-prem storage) if funding changes.  

---

## Action Items
1. **Run POC tests:** Ingest wired sensors into IoT Core, monitor costs for 20–150 devices.  
2. **Containerize Node.js website:** Prepare Docker Compose setup for ECS Fargate deployment.  
3. **Finalize hybrid architecture plan:** LoRaWAN via IoT Core, wired sensors via Node-RED → S3.  
4. **Test Grafana Athena plugin** for querying S3-stored data.  
5. **Provide architecture files:** AWS team to share **source diagrams (SVG, PDF)** for modification.  

---

## Key Decisions
- **LoRaWAN devices** will migrate to **IoT Core + Lambda pipeline**.  
- **Node-RED** will likely remain (short-term) for wired sensors until costs and feasibility are fully assessed.  
- **Aurora serverless** will replace PostgreSQL for website data (30-day retention).  
- **ECS Fargate** preferred for Node.js hosting (cost-effective and serverless).  

---

## Next Steps
- **POC with high-frequency sensors** (simulate 20–150 devices for one week).  
- **Develop ECS/Fargate deployment** for the SharedAirDFW website.  
- **Update hybrid architecture diagram** (AWS + on-prem + OSN).  
- **Review POC results** to finalize migration plan for wired sensors.  



# Comprehensive Summary of MINTS-AI Meeting  
**Date:** July 20, 2023  

---

## Purpose of the Meeting
- **Review progress on POC pipeline** for AWS IoT Core and S3 integration.  
- **Discuss cost estimates** for scaling LoRaWAN and wired sensors.  
- **Plan next steps for migrating website, Grafana, and InfluxDB to AWS.**  

---

## Key Discussion Points

### 1. **POC Progress: Data Ingestion**
- **Mints** successfully set up a **node sending data to AWS IoT Core**.  
- Data is processed via **Lambda functions** and stored in **S3**.  
- **Two storage formats tested:**  
  - **Raw packets:** Full LoRaWAN payloads.  
  - **Decoded packets:** Parsed sensor readings (node ID, sensor ID, values).  
- **S3 folder structure:** Organized by **year → month → date → hour**.  
- **Current cost:** **~$0.30 (S3)** and **~$0.27 (IoT)** for July (low cost).  

### 2. **Next Steps for Testing**
- **Expand POC:** Add 12 more physical sensors and/or use **AWS IoT emulator** for scaling tests (50–150 devices).  
- **Athena Testing:**  
  - Evaluate **AWS Athena** to query decoded S3 data.  
  - Integrate **Athena with Grafana** for live dashboards.  
- **DynamoDB Option:** Consider storing **recent (real-time) data** in **DynamoDB** for faster queries.  

### 3. **Website & Grafana Migration**
- **SharedAirDFW Node.js website:**  
  - Currently hosted **on-prem (Proxmox)** with POSIX storage.  
  - **Goal:** Migrate to **EC2** or **ECS Fargate** for resiliency and lower management cost.  
- **Grafana & InfluxDB:**  
  - Currently **on-prem in containers**.  
  - Plan to **deploy to ECS cluster** (Fargate) for high availability.  
  - **Managed Grafana** not feasible due to custom plugins.  

### 4. **Cost & Storage Strategy**
- **LoRaWAN data:**  
  - S3 + Athena provides a **low-cost, scalable data lake**.  
- **High-frequency wired sensors:**  
  - Need **Node-RED pipeline** for continuous ingestion.  
  - Costs to be measured after adding more devices.  
- **Node.js Website Storage:**  
  - Current on-prem POSIX store ≈ **5–10 TB**.  
  - **EFS cost is high** (~$3,000/month).  
  - **S3 with S3FS** (POSIX-like mount) considered (~$300/month).  
  - **Compromise:** Keep **1 month of data on EFS** (for website), archive historical data to S3.  

### 5. **Optimization Opportunities**
- **Reserve instances & savings plans** for cost reduction (up to 70%).  
- **Containerization:** Use **Docker + ECS Fargate** to simplify scaling.  
- **Serverless Aurora** for website data (30-day retention).  

---

## Action Items
1. **Test Athena integration:** Query S3 data from Grafana dashboards.  
2. **Run POC scaling:** Simulate 50–150 devices to measure IoT Core + S3 costs.  
3. **Explore DynamoDB:** Test for storing high-frequency data for real-time queries.  
4. **Containerize Grafana & Node.js website:** Deploy to ECS (Fargate).  
5. **Calculate EFS vs. S3FS trade-offs** for hosting Node.js data.  
6. **Get plugin list for Grafana:** AWS to verify compatibility for managed Grafana.  
7. **Gather storage usage:** Determine **exact size of 1-month website data** for cost estimation.  

---

## Key Decisions
- **LoRaWAN pipeline:** Use **AWS IoT Core → Lambda → S3 (Athena)**.  
- **Wired high-frequency sensors:** Keep **Node-RED ingestion pipeline** for now.  
- **Website:** Likely to migrate to **EC2/ECS with POSIX-like storage** (hybrid EFS + S3).  
- **Grafana:** Remains **self-hosted in ECS containers** (custom plugin support).  

---

## Next Steps
- **Run scaled POC** with emulator (150 simulated devices).  
- **Test Grafana-Athena integration** for querying S3.  
- **Migrate Node.js website to AWS EC2/ECS** (POC).  
- **Prepare cost estimates** for full migration (target: by end of July).  
- **Schedule follow-up session** for reviewing storage & cost model.  


# Comprehensive Summary of MINTS-AI Meeting  
**Date:** August 10, 2023  

---

## Purpose of the Meeting
- **Review containerization progress** for the SharedAirDFW website.  
- **Discuss emulator setup** for simulating sensor data traffic.  
- **Plan next steps** for containerizing InfluxDB and Grafana and preparing cost estimates.  

---

## Key Discussion Points

### 1. **Website Containerization (SharedAirDFW)**
- **Website successfully containerized** using **AWS ECS with Fargate**.  
- Hosted on a temporary **POC DNS**: `mints.trecis.io`.  
- **Configuration:** Minimal setup for proof-of-concept; production-ready setup will require:  
  - **SSL certificate** for HTTPS.  
  - **Configuration adjustments** to match on-prem performance requirements.  
- **Next Steps:**  
  - Add SSL certificate (OIT to assist).  
  - Collect on-prem site configuration for resource sizing.  
  - Run the containerized website for **1–2 weeks** to gather accurate **cost estimates**.  
  - Move containerized **Docker files** and code to a **dedicated GitHub repo** for centralized management.  

### 2. **Grafana & InfluxDB Migration**
- **InfluxDB and Grafana** currently run **on-premises in containers**.  
- Plan to **migrate these to ECS (Fargate)** in the same cluster as the website.  
- **Scaling:** Use CloudWatch metrics to determine required compute/memory resources.  
- **Historical data:**  
  - InfluxDB will retain **recent data (~1 month)**.  
  - **Long-term data** will remain archived in **Open Storage Network (OSN)** and AWS S3.  

### 3. **IoT Emulator Setup**
- **Emulator configured** to simulate **30 LoRaWAN sensors** sending messages every **10 seconds**.  
- **Next Steps:**  
  - Run **long-term simulation (1+ week)** to mimic production load.  
  - Use emulator data to refine **cost modeling for AWS IoT Core + S3 pipeline**.  

### 4. **Cost Estimation Plans**
- **Target:** Estimate **end-to-end migration costs** (website, Grafana, InfluxDB, IoT pipeline).  
- **Storage:**  
  - **S3 for LoRaWAN data lake** (low-cost).  
  - **EFS vs. S3FS** for website POSIX-like storage (trade-off between cost & performance).  
- **Compute:** Evaluate **Fargate auto-scaling** to optimize costs based on actual usage.  
- **Licensing:** SSL and reserved instance pricing strategies to be explored.  

### 5. **Collaboration & Documentation**
- Move **all Docker files and container configurations** to a **centralized GitHub repository**.  
- Document ECS cluster setup for **reproducibility** and **future team onboarding**.  
- Plan for **Grafana plugin compatibility** testing in AWS-hosted environment.  

---

## Action Items
1. **Add SSL certificate** and configure HTTPS for containerized website (OIT to assist).  
2. **Run containerized website** for 1–2 weeks to gather cost metrics.  
3. **Migrate InfluxDB & Grafana** to ECS (Fargate) in the same cluster as the website.  
4. **Run emulator simulation** for 1+ week to replicate production load.  
5. **Collect on-prem metrics**:  
   - Website server specs (CPU, RAM).  
   - Database sizes for historical vs. recent data.  
6. **Move Docker files** and website code to **GitHub** for centralized tracking.  
7. **Prepare cost estimates** for the full AWS migration (target by next session).  

---

## Key Decisions
- **Website, Grafana, and InfluxDB** will be **containerized on ECS Fargate**.  
- **1 month of recent data** will remain in InfluxDB for quick access; historical data will remain in **OSN/S3**.  
- **LoRaWAN simulation** to run for 1+ week to refine cost modeling.  
- **GitHub repository** will store all containerization code for version control.  

---

## Next Steps
- Verify **containerized website functionality** with MINTS team.  
- **Test load** (100–200 concurrent users) using Selenium/browser scripts to evaluate ECS scaling.  
- **Prepare draft cost estimates** for full migration.  
- Schedule **next session** to review emulator results and cost projections.  


# Comprehensive Summary of MINTS-AI Meeting  
**Date:** August 31, 2023  
---

## Purpose of the Meeting
- **Review AWS ECS deployment** of website, Node-RED, Grafana, and InfluxDB.  
- **Evaluate AWS IoT Core simulation** for device cost modeling.  
- **Plan migration of PostgreSQL database** to AWS RDS and finalize remaining pipeline integrations.  

---

## Key Discussion Points

### 1. **ECS Deployment Status**
- **Website, Node-RED, Grafana, and InfluxDB** successfully containerized and running on **AWS ECS with Fargate**.  
- **Configuration:** Using minimal resources (**1 CPU, 6 GB memory**) during testing.  
- **Next steps:** Monitor **CloudWatch metrics** to determine scaling requirements.  

### 2. **AWS IoT Core Simulation**
- **Simulator configured** for **100 devices** to estimate monthly costs.  
- **Limitation:** AWS simulator sessions auto-terminate after **15 minutes**.  
- **Cost estimate for 100 devices:** ~**$12/month** (AWS IoT Core only; excludes storage and downstream processing).  
- **Next steps:**  
  - Explore **long-running simulations** (workaround for 15-minute limit).  
  - Integrate simulated data with **Node-RED, InfluxDB, and Grafana** for full pipeline testing.  

### 3. **PostgreSQL Database Migration**
- **Current state:** PostgreSQL database running **on-premises**, supporting SharedAirDFW website.  
- **Size:** ~**25 GB**.  
- **Proposed migration:**  
  - Move to **AWS RDS (PostgreSQL)** for improved reliability.  
  - Use **AWS Database Migration Service (DMS)** or snapshot-based import (simpler for 25 GB size).  
- **On-prem dependency:** Website still relies on campus-hosted PostgreSQL; migration will eliminate this.  

### 4. **Data Flow & Pipeline Integration**
- **Current pipelines:**  
  - **MQTT → CSV → PostgreSQL** for website live data.  
  - **MQTT → Node-RED → InfluxDB → Grafana** for dashboards.  
- **Issues identified:** CSV-based approach is inefficient.  
- **Proposed changes:**  
  - **Direct ingestion from AWS IoT Core** to RDS and InfluxDB (eliminate CSV).  
  - Evaluate **AWS Glue catalog** for SQL-like querying on S3 data.  

### 5. **Website & Grafana Integration**
- **SharedAirDFW website**: Will be updated to point to **RDS PostgreSQL** once migration is complete.  
- **Grafana dashboards**: Hosted on ECS; need to **connect to AWS IoT Core and InfluxDB** for live data.  
- **Stress testing planned**: Use **Selenium/browser scripts** to simulate **100–200 concurrent users**.  

### 6. **Documentation & Architecture**
- **Action:** Prepare **updated AWS architecture diagrams** covering current ECS, IoT Core, RDS, and data pipelines.  
- **Include:** Networking flows, scaling strategies, and component dependencies.  

---

## Action Items
1. **Migrate PostgreSQL database** to AWS RDS using **snapshot or DMS** (OIT database team to assist).  
2. **Replace CSV ingestion pipeline** with **direct IoT Core → RDS/InfluxDB** integration.  
3. **Run stress tests** on the website using Selenium (target 100–200 concurrent users).  
4. **Expand IoT simulation** (longer runtime) for realistic cost estimation.  
5. **Prepare detailed AWS architecture diagram** showing updated environment and dependencies.  
6. **Review ECS configurations** for scaling and security group adjustments.  
7. **Collaborate with database team** for best practices in RDS configuration and migration.  

---

## Key Decisions
- **PostgreSQL will migrate to AWS RDS** for improved reliability.  
- **CSV-based ingestion will be phased out** in favor of direct IoT Core → RDS/InfluxDB pipeline.  
- **Stress testing and cost estimation** will guide ECS scaling decisions.  
- **Architecture documentation** will be developed for visibility and long-term management.  

---

## Next Steps
- Complete **PostgreSQL migration** and reconfigure website connections.  
- **Connect AWS IoT Core data** to InfluxDB and RDS.  
- Run **stress/load tests** on ECS-hosted website and dashboards.  
- Finalize **architecture diagrams** for review at next session.  
- Continue **weekly cadence meetings** for next 6 weeks to monitor progress.  



# Comprehensive Summary of MINTS-AI Meeting  
**Date:** September 6, 2023  
---

## Purpose of the Meeting
- **Assess on-premises database infrastructure** for SharedAirDFW and Grafana.  
- **Plan migration of databases** (PostgreSQL & InfluxDB) to AWS RDS or equivalent managed services.  
- **Identify critical issues** with on-prem storage and database management to prevent service downtime.  

---

## Key Discussion Points

### 1. **Current Database Challenges**
- **On-prem environment:**  
  - Hosted on **Proxmox virtualized servers** (ARDC).  
  - **PostgreSQL (v11.9)** powering **SharedAirDFW** website (~23 GB).  
  - **InfluxDB** powering Grafana dashboards.  
- **Critical issues:**  
  - **Disk space at 100%** on the PostgreSQL VM (root filesystem fully utilized).  
  - Growth rate: ~**200 MB/week**; disk will fill within **3 weeks** unless cleaned or expanded.  
  - **Backup directories (Borg.local)** taking up ~58 GB (uncertain contents, likely backups).  
  - **Hardware support status unknown** — physical host may be out of warranty.  
- **Impact:** When the disk fills, **SharedAirDFW website goes down**, generating support calls.  

### 2. **Immediate Stabilization Plan**
- **Expand disk storage:**  
  - Investigate expansion of Proxmox VM disk (possible downtime).  
  - Consider offloading Borg backups to **MooseFS storage (NFS)** for extra capacity.  
- **Clean up unnecessary files:** Determine which Borg backups can be deleted or relocated.  
- **Set up monitoring:** Use **Zabbix** to alert on low disk space.  

### 3. **AWS Migration Strategy**
- **Goal:** Move databases to **AWS managed services** for improved reliability and scalability.  
- **Proposed steps:**  
  - **Backup databases:** Use `pg_dump` for PostgreSQL and equivalent export for InfluxDB.  
  - **Migrate PostgreSQL** to **AWS RDS (latest supported version)**.  
  - **Evaluate InfluxDB alternatives**: Either migrate to **managed InfluxDB** or switch Grafana backend to **PostgreSQL/MySQL**.  
  - Integrate databases with **existing AWS ECS-hosted website** and **IoT Core pipelines**.  

### 4. **Infrastructure Risks & Considerations**
- **Single points of failure:**  
  - No proactive upgrades or patching on database VMs.  
  - Unclear backup frequency & off-site backup policy.  
  - Potential downtime required for disk expansion.  
- **Sensor scaling:**  
  - Current system supports **~15 campus sensors**; plans to add **~100 more** soon.  
  - Expect **significant growth** in data volume (doubling every few months).  

### 5. **Responsibilities & Coordination**
- **On-Prem Team:** Steven Goss & Susmita to work with Chris Simmons on **disk expansion & cleanup**.  
- **AWS Migration Team:** Greg & Simon to set up **AWS RDS environments** for testing.  
- **Mints:** Provide database dumps and assist with integration testing.  
- **OIT:** Validate **hypervisor capacity** and physical server warranty status.  

---

## Action Items
1. **Expand VM storage** on Proxmox.  
2. **Investigate & clean Borg.local** backups 
3. **Backup PostgreSQL & InfluxDB** using `pg_dump` and equivalent tools.  
4. **Migrate PostgreSQL to AWS RDS** 
5. **Evaluate Grafana backend options** (InfluxDB vs. PostgreSQL).  
6. **Add Zabbix monitoring** for database servers.  
7. **Document on-prem infrastructure** and update architecture diagrams.  

---

## Key Decisions
- **SharedAirDFW PostgreSQL database** is the **top priority** for migration.  
- **Grafana InfluxDB** migration is secondary but still important for reliability.  
- **Disk expansion** and **monitoring setup** are urgent to prevent outages.  
- **AWS RDS** will be used for PostgreSQL migration; Grafana database solution TBD.  

---

## Next Steps
- **Perform disk expansion** and clean-up within the week.  
- **Backup and migrate PostgreSQL** to RDS (proof-of-concept environment).  
- **Determine best backend for Grafana** and plan migration.  
- **Update system architecture diagrams** to reflect AWS migration path.  
- Schedule **follow-up meeting** to review migration progress and infrastructure stability.  




# Comprehensive Summary of MINTS-AI Meeting  
**Date:** September 7, 2023  

---

## Purpose of the Meeting
- **Review progress on AWS integration** for sensor data ingestion and S3 storage.  
- **Discuss database stability issues** and long-term hosting strategy.  
- **Plan documentation and architecture updates** for current and AWS environments.  

---

## Key Discussion Points

### 1. **Sensor Data Ingestion Updates**
- **Mints reformatted high-frequency sensor data** for **direct ingestion into S3**:  
  - Created **structured JSON files** with timestamps (microseconds since Unix epoch).  
  - Organized data by **node ID → sensor ID** hierarchy in S3 buckets.  
  - **Parameters included:** BME280 (environmental data), GPS strings, IPS7100 (battery), CO₂ sensor.  
- **Proposed improvement:** Further divide data by **day/hour folders** to simplify queries.  
- **Implementation:** MQTT script embedded within devices to **push data directly to S3**.  
- **Credentials discussion:** Currently using **shared credentials** for devices; discussed separating credentials per device for better security.  

### 2. **AWS Secrets Management**
- Chris recommended using **AWS Secrets Manager**:  
  - Store IoT credentials securely.  
  - Retrieve credentials programmatically from Python scripts.  
  - Removes need for hardcoding secrets in Git repos.  

### 3. **On-Prem Database Stabilization**
- **PostgreSQL server (SharedAirDFW):**  
  - Cleared **Borg.local backups** to free up **~63 GB space**.  
  - Estimated remaining capacity: **1–2 years** at current growth (~200 MB/week).  
  - No immediate disk expansion needed but will revisit long-term solution.  
- **InfluxDB:** Still powering Grafana dashboards; migration plan under discussion.  
- **Next steps:** Determine **Proxmox hypervisor capacity** and consider **disk expansion** if needed.  

### 4. **Access & Permissions**
- **Chris** gained access to **psql.circ.utdallas.edu** for further investigation.  
- **Group access issue:** Users lack write permissions to Mints’s directories on **Mdash** (Grafana/Node-RED containers).  
- Plan: **Create user group with write permissions** for research team to update dashboards and repos without bottlenecks.  

### 5. **Long-Term Database Strategy**
- **Three options under review:**  
  - **Continue on-prem hosting** (expand storage and manage locally).  
  - **Migrate to AWS RDS** (PostgreSQL for SharedAirDFW).  
  - **Hybrid approach**: Keep query-heavy datasets on-prem, offload long-term storage to AWS.  
- **AWS Cost Estimate:** Chris estimated **$100–200/month** for a 500 GB AWS RDS database.  
- **Goal:** Prototype AWS database to estimate cost and performance with live workloads.  

### 6. **Documentation & Architecture**
- **Action:** Document current environment (hardware, software, responsibilities).  
- **Develop AWS architecture diagrams** for managed service migration (ECS, RDS, S3).  
- **Include:** Scaling plans for 10 → 100+ sensors and cost projections.  

---

## Action Items
1. **Implement AWS Secrets Manager** for IoT device credentials.  
2. **Restructure S3 storage** to include **day/hour partitioning** for sensor data.  
3. **Create user group** for write access to Mdash directories.  
4. **Backup PostgreSQL & InfluxDB** for migration testing.  
5. **Prototype AWS RDS** instance for SharedAirDFW database.  
6. **Document on-prem & AWS environments** (hardware, software, owners).  
7. **Update architecture diagrams** for current and AWS-hosted environments.  

---

## Key Decisions
- **Secrets Manager** will replace static IoT credentials in Git.  
- **Prototyping AWS RDS** for PostgreSQL is a priority for cost evaluation.  
- **Group access** approach preferred over individual repo copies for Grafana/Node-RED management.  
- **Hybrid hosting** remains an option depending on cost and performance benchmarks.  

---

## Next Steps
- **Implement Secrets Manager** for devices and test credential retrieval.  
- **Run AWS RDS prototype** and measure performance vs on-prem database.  
- **Prepare updated architecture diagrams** for AWS-managed environment.  
- **Schedule follow-up session** to review database migration progress and cost modeling.  




# Comprehensive Summary of MINTS-AI Meeting  
**Date:** September 14, 2023  

---

## Purpose of the Meeting
- **Review PostgreSQL migration progress** to AWS.  
- **Investigate ingestion workflows** for updating the SharedAirDFW PostgreSQL database.  
- **Discuss cost monitoring and cloud credits** for AWS infrastructure.  
- **Finalize updates to AWS ECS containers** and ensure system connectivity.  

---

## Key Discussion Points

### 1. **PostgreSQL Migration Progress**
- **Steven & Susmita** successfully created a **PostgreSQL dump** of the SharedAirDFW database.  
- **Next step:** Deploy the database to **AWS RDS** for use by the SharedAirDFW container hosted in AWS.  
- **Outstanding question:** Unclear **cron job/process** that updates the PostgreSQL database with CSV data.  
  - Investigated **GaiKon’s account on EOSFTP**, web server, and PostgreSQL server; could not locate cron jobs responsible for ingestion.  
  - **Action:** Contact **GaiKon** to clarify how data is injected into PostgreSQL (cron jobs, scripts, or external API).  

### 2. **Data Ingestion & API Discovery**
- Found an API endpoint: `api.sharedair.dfw.com/info` that provides some data.  
- **Unclear** if this API is responsible for database writes or just data retrieval.  
- **DB connector scripts** discovered in the **SharedAirDFW GitHub repository** with basic data insertion queries.  
- **Next steps:** Investigate whether devices or external scripts directly insert data into PostgreSQL.  

### 3. **ECS Containers & Connectivity**
- **Node-RED** had issues writing to **InfluxDB** on ECS; fixes pushed to GitHub by **John**.  
- **InfluxDB & Node-RED** successfully deployed to **AWS ECS** using existing container images.  
- **SharedAirDFW website**: Needs to be updated to connect to the new AWS-hosted PostgreSQL database.  

### 4. **Architecture & Documentation**
- **Susmita** is consolidating a list of all on-prem and AWS servers, including their responsibilities and backup strategies.  
- **Steven Goss** will assist in documenting **backup workflows** and determining **on-prem hardware warranty status**.  
- **AWS diagram updates:** Ongoing as infrastructure changes are made (ECS containers, databases, networking).  

### 5. **Cost Monitoring & AWS Credits**
- AWS team requested a **cost threshold** for setting up **alerts**.  
- **No threshold set yet**, but project has **AWS credits** assigned; usage will be tracked closely.  
- **Plan:** Once PostgreSQL migration is live, use real-world cost data to refine projections.  

### 6. **InfluxDB & Grafana Migration**
- **InfluxDB** (Grafana backend) remains on-prem but is planned for **ECS container migration**.  
- **Grafana** will run on **EC2 or containerized ECS instance** (hosted Grafana not feasible due to plugin dependencies).  

---

## Action Items
1. **Contact GaiKon** to clarify data ingestion process for PostgreSQL.  
2. **Deploy PostgreSQL dump to AWS RDS** and reconfigure the SharedAirDFW website.  
3. **Investigate DB connector scripts** for insertion processes and dependencies.  
4. **Document on-prem & AWS infrastructure** (servers, responsibilities, backups).  
5. **Set up AWS cost alerts** after determining thresholds.  
6. **Update architecture diagrams** to reflect current AWS deployments.  
7. **Plan migration for InfluxDB & Grafana** to AWS ECS/EC2.  

---

## Key Decisions
- **PostgreSQL migration to AWS RDS** is the top priority for improving reliability.  
- **Contacting GaiKon** is necessary to clarify unknown ingestion workflows.  
- **Cost monitoring** will be implemented after setting thresholds.  
- **Grafana & InfluxDB** will be migrated after PostgreSQL stabilization.  

---

## Next Steps
- **Confirm data ingestion workflow** for PostgreSQL.  
- **Deploy and configure PostgreSQL on AWS RDS** for SharedAirDFW.  
- **Test ECS-hosted website connectivity** to AWS RDS.  
- **Update system architecture documentation** with recent changes.  
- **Schedule follow-up** to review migration progress and cost monitoring setup.  


# Comprehensive Summary of MINTS-AI Meeting  
**Date:** September 21, 2023  

---

## Purpose of the Meeting
- **Review PostgreSQL migration** to AWS Aurora (RDS).  
- **Plan ingestion workflow migration** to AWS-hosted PostgreSQL.  
- **Discuss replacing CSV ingestion with Node-RED** for improved flexibility.  
- **Address database credentials and access for migration.**  

---

## Key Discussion Points

### 1. **PostgreSQL Migration Update**
- **DBA team imported PostgreSQL database** into **AWS Aurora (RDS)** using **PG dump from September 12th** provided
- **Next steps:**  
  - Determine whether to **update with a fresh backup** or proceed with the current dump.  
  - Connect the **SharedAirDFW website** to the AWS-hosted database.  

### 2. **Data Ingestion Workflow**
- **Current ingestion method:**  
  - **CSV files** generated from MQTT pipelines.  
  - **Python scripts + cron jobs** (likely using SQLAlchemy) ingest CSV data into PostgreSQL.  
  - Additional scripts fetch **NOAA wind data every six hours**.  
- **Challenge:** Workflows are **scattered across on-prem servers** and difficult to maintain.  

### 3. **Proposed Migration: Node-RED**
- **Mints proposed replacing CSV ingestion with Node-RED**:  
  - Directly ingest data into PostgreSQL.  
  - Easier to manage and more scalable for future growth.  
- **John & Chris:** Support the proposal but raised **concerns about development time** and resource availability.  
- **Consensus:** Continue using **CSV ingestion for now** while planning a **long-term Node-RED migration**.  

### 4. **Action Items for Ingestion Migration**
- **Identify all Python ingestion scripts** currently used.  
- **Review cron jobs** under **GaiKon’s account** (IMD server) for ingestion pipelines.  
- **Document NOAA data fetching workflows** for migration to AWS.  

### 5. **Database Credentials & Access**
- **Credentials for the on-prem PostgreSQL database are missing**.  
- **Action:** Investigate GaiKon’s user space and scripts to retrieve credentials.  
- If credentials cannot be recovered, **create new users** with proper access for ingestion.  

### 6. **Architecture & Containerization**
- **Node-RED & website containers** are running in **AWS ECS**.  
- Plan to **containerize the ingestion workflows** for better portability.  
- **Node-RED API server** will be made **public-facing**, while **PostgreSQL remains on a private network** for security.  

---

## Action Items
1. **Decide on using current or fresh PostgreSQL dump** for AWS RDS.  
2. **Retrieve PostgreSQL credentials** from legacy environment or create new users.  
3. **Document ingestion workflows** (CSV pipelines, NOAA wind data scripts, cron jobs).  
4. **Plan Node-RED migration** for direct ingestion (long-term).  
5. **Reconfigure SharedAirDFW website** to connect to AWS RDS.  
6. **Prepare containerized ingestion workflows** for future deployment.  
7. **Set up a working session** to review ingestion scripts and workflows.  

---

## Key Decisions
- **Current CSV ingestion will remain** in use for now to minimize disruption.  
- **Node-RED ingestion** is a long-term goal for scalability and easier maintenance.  
- **AWS Aurora (RDS)** is the chosen platform for PostgreSQL hosting.  
- **Working session** will clarify all ingestion pipeline dependencies.  

---

## Next Steps
- **Investigate ingestion workflows** and clarify missing cron job details.  
- **Update AWS RDS** with a fresh backup if needed.  
- **Reconfigure SharedAirDFW website** for AWS-hosted PostgreSQL.  
- **Plan Node-RED migration** as part of long-term architecture modernization.  





# Comprehensive Summary of MINTS-AI Meeting  
**Date:** September 25, 2023  
---

## Purpose of the Meeting
- **Clarify PostgreSQL ingestion workflows** for SharedAirDFW.  
- **Document current scripts and dependencies** for future AWS migration.  
- **Discuss eliminating CSV intermediate step** for direct data ingestion.  
- **Review updates on containerized services in AWS ECS.**  

---

## Key Discussion Points

### 1. **Current Data Ingestion Workflow**
- **Two Python scripts** handle data ingestion:  
  - Script 1: **Reads MQTT pipeline** (directly connected and LoRaWAN nodes) → **creates CSV**.  
  - Script 2: **Reads CSV** → **inserts data into PostgreSQL**.  
- **Wind data ingestion:** Separate cron job/script located in the `mints_data-ingestion` GitHub repository.  
- **Averaging:** Current scripts average PM sensor data (1-second → 30-second intervals).  
- **Limitation:** No automatic restart on reboot; scripts must be manually restarted when servers go down.  

### 2. **Proposed Changes**
- **Eliminate CSV step**: Directly ingest data into PostgreSQL (likely via Node-RED).  
- **Streamline ingestion:** Consolidate Python scripts and cron jobs into a **containerized workflow** for AWS ECS.  
- **Reassess database schema:** Determine if **table headers and structure need modification** for direct MQTT ingestion.  
- **Maintain GPS fallback:** Scripts must handle missing GPS data (using YAML file defaults).  

### 3. **AWS Migration Progress**
- **SharedAirDFW website, Node-RED, InfluxDB, and Grafana** are successfully containerized and running on **AWS ECS**.  
- **PostgreSQL database** migrated to **AWS Aurora (RDS)**; **credentials recovered** for the `mints` database user.  
- Next step: **Connect containerized website and workflows to AWS RDS**.  

### 4. **Documentation & Discovery**
- **Susmita** leading efforts to document:  
  - **Servers and VMs** in ARDC (locations, roles, patch history, warranties).  
  - **Scripts, cron jobs, and GitHub repositories** used for ingestion.  
  - **Current architecture diagram** (on-prem + AWS).  
- **Mints** to provide a document listing all **repositories and running scripts** by Thursday.  

### 5. **On-Premises Infrastructure Updates**
- **Servers and VMs:** All hosted in **ARDC**; sensors distributed across Dallas.  
- **Power maintenance:** Scheduled **every Friday** for the next several weeks; requires **manual script restarts** after outages.  
- **Server administration:** Access shared between Steven, Chris, and GaiKon.  

### 6. **Migration Coordination**
- **Final migration to AWS** will require:  
  - Shutting down the on-prem PostgreSQL database.  
  - Obtaining a fresh **PG dump**.  
  - Importing into AWS RDS (~2-hour process).  
  - Restarting applications after migration.  
- **Collaboration:** Cloud team to coordinate with **DBA team** for final cutover.  

---

## Action Items
1. **Document all Python ingestion scripts**.  
2. **Create a process map** of current ingestion workflows.  
3. **Eliminate CSV ingestion step**: Design direct MQTT → PostgreSQL pipeline.  
4. **Containerize ingestion workflows** for AWS ECS.  
5. **Prepare for final migration**: Plan shutdown, fresh PG dump, and import into RDS.  
6. **Continue documentation**: Servers, warranties, patch history, and network dependencies.  
7. **Coordinate with AWS cloud team** for final migration planning.  

---

## Key Decisions
- **Node-RED or Python (containerized)** will replace the CSV-based ingestion pipeline.  
- **AWS RDS** will serve as the primary PostgreSQL database.  
- **Fresh PG dump** required before final cutover.  
- **Documentation** of current ingestion processes is a priority for migration readiness.  

---

## Next Steps
- **Complete repository and script inventory** by Thursday.  
- **Develop containerized ingestion solution** (Python/Node-RED).  
- **Prepare migration plan** for final PostgreSQL cutover.  
- **Review updated documentation and architecture diagrams** at the next session.  

# Comprehensive Summary of MINTS-AI Meeting  
**Date:** October 5, 2023  

---

## Purpose of the Meeting
- **Review AWS deployment status** and current monthly costs.  
- **Verify live data integration** for the SharedAirDFW containerized website.  
- **Plan for integrating AWS Glue crawlers and Parquet format** for S3-stored IoT data.  
- **Discuss on-prem compliance documentation and identify gaps.**  

---

## Key Discussion Points

### 1. **AWS Deployment & Costs**
- **Most resources have been migrated to AWS**; only a few unknown on-prem components remain.  
- **Current costs:** ~$200–300/month; projected costs could rise to **$500/month** as more resources are added.  
- **Main cost drivers:** Containerized dashboards and database services.  

### 2. **SharedAirDFW Website & Database Integration**
- **Containerized website** is live on AWS but **still pulling data from the on-prem database** (via API at `api.sharedairdfw.com`).  
- **Action required:** Reconfigure API endpoints to point to the **AWS RDS database**.  
- **Raised concern:** Current AWS-hosted website is **not receiving live sensor data**; only static database copy is displayed.  
- **Plan:** Redirect one or two sensors to **send data directly to AWS S3** for live updates.  

### 3. **Data Pipeline Improvements**
- **IoT Core → S3 Pipeline:** Currently stores data in **JSON** format.  
- **Proposal:** Convert JSON to **Parquet format** for:  
  - Faster query performance.  
  - Lower storage costs.  
- **AWS Glue Crawler:** Suggested to **catalog S3 data** for querying via Athena or pushing into **AWS RDS**.  
- **Follow-up:** **Workshop scheduled for October 11 (9 AM CST)** to review S3 data, test crawlers, and plan website integration.  

### 4. **Compliance Documentation & On-Prem Gaps**
- **Susmita shared compliance documentation** covering:  
  - **Monitoring & Operations** (Zabbix, Splunk, etc.).  
  - **Data movement & dependencies**.  
  - **Backup & disaster recovery planning**.  
  - **SLAs and resources for migration.**  
- **Key issues identified:**  
  - Lack of proactive alerts for outages (e.g., power failures).  
  - **Backup & patching workflows** need improvement.  
  - **Disaster recovery strategy** still incomplete.  

### 5. **Next Steps for Cloud Migration**
- **Kishore & Susmita:** Develop **architecture diagram** showing current AWS and on-prem components.  
- **Define monitoring strategy:** Determine alerts for website downtime, container scaling, and infrastructure health.  
- **Plan observability:** Attend **AWS observability workshop** for implementing best practices.  

---

## Action Items
1. **Reconfigure SharedAirDFW API** to point to AWS RDS.  
2. **Redirect one or two sensors** to send data directly to AWS S3.  
3. **Explore Parquet format conversion** for S3-stored data.  
4. **Run AWS Glue crawler** on S3 data for cataloging.  
5. **Finalize compliance document** with feedback from Dr. Larry and team.  
6. **Create architecture diagram** (AWS + on-prem + data flows).  
7. **Define backup & disaster recovery strategies** with clear ownership.  

---

## Key Decisions
- **AWS RDS** will replace on-prem database as the primary data source.  
- **Parquet + Glue crawler** approach will be explored for cost-effective data management.  
- **Workshop on Oct 11** to review and refine data ingestion and cataloging strategies.  
- **Compliance and architecture documentation** are priorities for next phase.  

---

## Next Steps
- **Host working session (Oct 11)** to evaluate S3 data and AWS Glue integration.  
- **Reconfigure website** for live data ingestion from AWS-hosted pipelines.  
- **Update architecture diagrams** to reflect new data flows and components.  
- **Develop proactive monitoring strategy** for website and database uptime.  



# Comprehensive Summary of MINTS-AI Meeting  
**Date:** October 11, 2023  

---

## Purpose of the Meeting
- **Explore options for migrating live sensor data** into AWS-hosted databases.  
- **Evaluate DynamoDB vs. PostgreSQL (RDS)** for IoT sensor ingestion.  
- **Review AWS Glue integration** for data transformation.  
- **Plan migration strategy for on-prem databases.**  

---

## Key Discussion Points

### 1. **Current Data Pipeline**
- **Live data** from IoT Core (MQTT) stored in **AWS S3** as JSON.  
- **Python script** processes raw MQTT data:  
  - **Averages data over 30-second intervals.**  
  - Combines particulate matter (IPS7100) and environmental (BME280) data.  
  - Writes formatted rows to PostgreSQL (on-prem).  
- **SharedAirDFW website:** Only **reads PM2.5 data** from PostgreSQL (historical artifact).  
- **Proposed simplification:** Only ingest PM2.5 + GPS into AWS database (ignore other environmental fields).  

### 2. **Database Migration & Structure**
- **Current PostgreSQL (on-prem)**: ~25 GB size, updated by Python scripts and CSV uploads.  
- **Existing AWS RDS (PostgreSQL):** Outdated dump already hosted in AWS.  
- **Options discussed:**  
  - Use **AWS Database Migration Service (DMS)** for continuous sync from on-prem PostgreSQL to AWS RDS.  
  - **Reformat data** for a **NoSQL database (DynamoDB)** to handle flexible columns and simplify ingestion.  
- **Consensus:** Prefer **DynamoDB** for new IoT data (flexibility, no schema updates needed).  

### 3. **AWS Glue for Data Transformation**
- **AWS Glue Studio** can process S3 JSON data:  
  - Perform **ETL (Extract, Transform, Load)** operations.  
  - **Reformat JSON** into a database-ready structure.  
  - **Optionally convert to Parquet** for storage and querying efficiency.  
- **Next Step:** Use Glue to **transform S3 JSON → DynamoDB/PostgreSQL** with automated workflows.  

### 4. **Lambda Functions & Automation**
- **Proposal:** Move the **Python processing script** (averaging, formatting) to an **AWS Lambda function**:  
  - Triggered by **new S3 uploads** or scheduled intervals.  
  - **Outputs formatted rows** directly to DynamoDB or RDS.  
- **GPS Fallback:** Need to integrate YAML-based static GPS mapping for devices that lose signal.  

### 5. **Migration Strategy & Costs**
- **Historical data:** Can be migrated to **DynamoDB** or **remain in RDS** for archival purposes.  
- **Live data:** New MQTT pipeline will **directly populate DynamoDB**.  
- **Cost Considerations:** No cost for incoming IoT data; egress traffic between AWS accounts may add costs.  
- **Size:** ~25 GB PostgreSQL database is manageable for migration.  

### 6. **Operational Needs**
- **Command-line access:** Mints requested ability to **run sensor management commands** (e.g., toggling sensors) in AWS.  
  - **Solution:** Use **ECS container access** or **AWS Cloud9 IDE** for secure interactive management.  
- **Website connectivity:** Must **update SharedAirDFW API** to read from the new AWS-hosted database.  

---

## Action Items
1. **Prototype DynamoDB ingestion** for live IoT data using IoT Core + Lambda.  
2. **Use Glue Studio** to transform S3 JSON into a DynamoDB-ready format.  
3. **Set up DMS** for syncing on-prem PostgreSQL → AWS DynamoDB/RDS.  
4. **Integrate GPS fallback** using YAML mapping in the Lambda workflow.  
5. **Enable command-line access** (ECS or Cloud9) for sensor management.  
6. **Reconfigure SharedAirDFW website API** to point to AWS-hosted database.  

---

## Key Decisions
- **DynamoDB** selected for new IoT data ingestion.  
- **Glue + Lambda** will replace on-prem Python scripts for data transformation and averaging.  
- **PostgreSQL RDS** may remain as a secondary database for historical data.  
- **Cloud-based command-line access** (ECS/Cloud9) will replace on-prem management scripts.  

---

## Next Steps
- **Build DynamoDB pipeline** (IoT Core → Lambda → DynamoDB).  
- **Set up Glue ETL workflow** for S3 → DynamoDB.  
- **Plan final migration** for historical PostgreSQL data.  
- **Update website API** to connect to AWS DynamoDB/RDS.  




# Comprehensive Summary of MINTS-AI Meeting  
**Date:** October 12, 2023  

---

## Purpose of the Meeting
- **Review AWS Glue usage for POC** and plan integration into the data pipeline.  
- **Investigate SharedAirDFW API location and configuration** for database updates.  
- **Plan migration of cron jobs and services** from legacy IMD server to AWS or other infrastructure.  
- **Improve monitoring and documentation** for high-reliability operations.  

---

## Key Discussion Points

### 1. **IMD Server & Cron Jobs**
- **Dr. Simmons confirmed** the IMD server is **old and out of warranty** (in service since 2019).  
- **Legacy cron jobs** running on IMD are **critical to data flow** but can be **migrated off-server** in the next few weeks.  
- **Action:** Work with Mints to **identify all cron jobs** and migrate them to more reliable infrastructure.  

### 2. **SharedAirDFW API & Database**
- **API investigation:**  
  - SharedAirDFW website **pings its own public API endpoint**, which connects to the **PostgreSQL database**.  
  - API likely **hosted on `mintsdata.circ.utdallas.edu`**.  
  - Need to **locate and document** the repository and service deployment details.  
- **Plan:**  
  - Steven to **find API service on the server** and confirm its GitHub repository.  
  - Update Susmita’s **confluence documentation** with repo locations and service details.  

### 3. **AWS DynamoDB Integration**
- **Objective:** Replace on-prem PostgreSQL ingestion pipeline with **AWS DynamoDB**.  
- **Next steps:**  
  - Read all **MQTT data streams** into DynamoDB using **IoT rules**.  
  - Configure **SharedAirDFW website** to read from DynamoDB instead of PostgreSQL.  
  - Perform **one round of testing** once database connection changes are complete.  

### 4. **AWS Glue for POC**
- **AWS Glue** will be used to **automate data ingestion and transformation** from IoT Core/S3 into DynamoDB.  
- Plan to **convert JSON → structured data** for efficient database queries.  
- Testing and **configuration tweaking** still needed for Mints workflows.  

### 5. **Monitoring & Reliability**
- **Steven** is setting up **Zabbix monitoring** for IMD server mounts and other key services.  
- **Goal:** Receive **immediate alerts** when mounts fail or pipelines break, reducing downtime.  

### 6. **Collaboration & Documentation**
- **Documentation efforts:**  
  - Susmita maintaining a **Confluence page** listing repositories, server locations, and services.  
  - Steven to **update with new API/service mappings** after server review.  
- **Repository consolidation:** Team will **map all GitHub repos** to their corresponding servers/services.  

---

## Action Items
1. **Identify and migrate cron jobs** from IMD server to a supported environment.  
2. **Locate SharedAirDFW API service** and document its repository and deployment details.  
3. **Update SharedAirDFW website** to pull data from **AWS DynamoDB**.  
4. **Use AWS Glue** to automate ingestion/transformation of IoT Core data.  
5. **Set up Zabbix alerts** for IMD server mount failures.  
6. **Update Confluence documentation** with new repo mappings and server details.  

---

## Key Decisions
- **IMD server services** will be migrated off to AWS or other stable environments.  
- **DynamoDB** will replace PostgreSQL for live data ingestion.  
- **AWS Glue** will be used for data processing and transformation.  
- **SharedAirDFW API** will be updated to connect to AWS-hosted databases.  

---

## Next Steps
- **Work session Monday:** Steven & Dr. Simmons to review IMD server services.  
- **Reconfigure SharedAirDFW API** for DynamoDB backend.  
- **Continue testing AWS Glue pipeline** for data transformation.  
- **Document all cron jobs, repos, and service locations** in Confluence.  


# Comprehensive Summary of MINTS-AI Meeting  
**Date:** October 19, 2023  

---

## Purpose of the Meeting
- **Review API deployment status** for the SharedAirDFW API in AWS.  
- **Discuss VPC configuration needs** for API Gateway and Lambda.  
- **Plan database monitoring and usage analysis** for PostgreSQL.  
- **Update on Zabbix monitoring deployment and repository mapping efforts.**  

---

## Key Discussion Points

### 1. **API Deployment in AWS**
- **API Deployment:** Working on deploying **SharedAirDFW API** (PostgreSQL-backed) to AWS.  
- **Repository identified:** Correct GitHub repository located for deployment.  
- **Configuration issue:**  
  - Initially attempted to use **shared services VPC** but **cannot create private API endpoints** in a shared VPC.  
  - **Solution:** Create a **dedicated VPC** for the MINTS project with private subnets and migrate Lambda/API Gateway into it.  
  - Temporary workaround: Use **public API endpoints** with encrypted traffic to speed up deployment while security/performance are reviewed.  

### 2. **Performance & Security Concerns**
- **Public API implications:**  
  - Traffic would go over the internet (encrypted).  
  - Adds an extra network hop, potentially increasing latency.  
- **Security context:** The SharedAirDFW project serves public data but could have **community-sensitive implications** (e.g., illegal dumping reporting).  
- **Decision:** Proceed with public endpoints short-term but evaluate private endpoints for final production deployment.  

### 3. **Database API Usage & Load**
- **Current frequency:** SharedAirDFW API updates data every **30 seconds**.  
- **Estimated Lambda invocation:** Could be triggered **once per second** for some sensors.  
- **Action:** Gather **accurate PostgreSQL query and API invocation metrics** using **CloudWatch or Grafana** to refine performance and cost estimates.  

### 4. **Zabbix Monitoring & Repository Mapping**
- **Zabbix:** Monitoring being configured across MINTS servers.  
  - Two servers having firewall issues, expected resolution by Friday.  
- **Repository Mapping:**  
  - Ongoing effort to **map all repositories and running services** on each machine.  
  - Goal: Document locations and dependencies in Confluence for easier management.  

### 5. **Collaboration & Documentation**
- **Current State Discovery:**  
  - HPC and Cloud Services teams will **work with Mints** to collect data on PostgreSQL usage patterns.  
  - **Action item:** Draft a set of discovery questions to assess API/database usage for documentation and optimization.  

---

## Action Items
1. **Create dedicated VPC** for MINTS with private subnets for Lambda/API Gateway.  
2. **Proceed with public API deployment** temporarily to maintain progress.  
3. **Gather PostgreSQL usage metrics** via CloudWatch, Grafana, or other monitoring tools.  
4. **Resolve firewall issues** blocking Zabbix deployment on two machines.  
5. **Continue repository mapping** for all services and document in Confluence.  
6. **Draft discovery questions** for PostgreSQL usage analysis (HPC/Cloud teams + Mints).  

---

## Key Decisions
- **Public API endpoints** will be used temporarily for AWS deployment.  
- **Dedicated VPC** will be created for long-term secure deployment.  
- **Database monitoring** will be prioritized to inform API design and resource allocation.  

---

## Next Steps
- **Deploy API Gateway + Lambda** in AWS using public endpoints.  
- **Start designing dedicated VPC** for MINTS private services.  
- **Implement PostgreSQL monitoring** for accurate API/database usage metrics.  
- **Finalize repository mapping** and document all dependencies.  
- **Follow up** in next session with API deployment and database usage findings.  





# Comprehensive Summary of MINTS-AI Meeting  
**Date:** October 26, 2023  

---

## Purpose of the Meeting
- **Review progress on AWS migration** for SharedAirDFW data pipeline.  
- **Confirm sensor data ingestion into AWS** from direct-connected and LoRaWAN devices.  
- **Evaluate costs for current AWS deployment** and project production-level expenses.  
- **Discuss VPC and transit gateway configuration for connectivity.**  

---

## Key Discussion Points

### 1. **Sensor Data Ingestion**
- **update:**  
  - **Direct-connected sensors:** Successfully publishing MQTT packets into AWS.  
  - **Data format:** Includes **datetime, PM data, and GPS data** required for SharedAirDFW.  
  - **Next steps:**  
    - Write code to include **LoRaWAN node data** into AWS.  
    - Insert all sensor data into **DynamoDB** as the final database target.  

### 2. **Database Migration**
- **Decision from prior meetings:** Use **DynamoDB** instead of RDS for new IoT data ingestion.  
- Plan to **migrate existing data** from RDS (PostgreSQL) into DynamoDB.  
- **Pending:** VPC configuration for proper database deployment.  

### 3. **VPC & Connectivity**
- **Current issue:** Cannot create private endpoints in a **shared VPC**.  
- **Action:**  
  - Create **dedicated VPC** for the MINTS project.  
  - **Evaluate transit gateway needs** based on on-prem connectivity requirements.  
  - **Short-term plan:** Configure new VPC and test AWS-on-prem connectivity.  

### 4. **AWS Cost Estimates**
- **October AWS costs:** ~**$150** for current deployment.  
- **Multiplicative factor for production:** Primarily dependent on **IoT Core traffic** and **storage growth**.  
- **Container costs:** Expected to remain stable unless **traffic scaling** requires additional compute.  
- Plan to use **CloudWatch** to monitor **container CPU and memory utilization** to plan scaling.  

### 5. **Monitoring & Documentation**
- **Zabbix monitoring:** Fully implemented across systems.  
  - **PostgreSQL VM:** Still has a **firewall issue** but actively collecting metrics.  
  - **Mount points monitored** with added alert notes for issue resolution.  
- **Documentation:**  
  - **Component diagram:** Being created in **draw.io** and integrated into Confluence by Susmita.  
  - **Reference documents:** Using architecture references provided by Mints.  

---

## Action Items
1. **Write ingestion code** for LoRaWAN nodes into AWS DynamoDB.  
2. **Migrate existing RDS data** into DynamoDB.  
3. **Configure dedicated VPC** for MINTS with connectivity to on-prem if needed.  
4. **Monitor container utilization** with CloudWatch to estimate scaling needs.  
5. **Complete component diagram** and update Confluence documentation.  
6. **Resolve firewall issue** for PostgreSQL VM in Zabbix monitoring.  

---

## Key Decisions
- **DynamoDB** confirmed as the target database for IoT data.  
- **Dedicated VPC** will be created for final deployment.  
- **Cost monitoring** will continue to refine production-level budget projections.  

---

## Next Steps
- **Implement LoRaWAN data ingestion** into AWS.  
- **Complete VPC configuration** and evaluate need for transit gateway.  
- **Estimate full production costs** based on IoT Core scaling.  
- **Review component diagram** in the next session for finalization.  




# Comprehensive Summary of MINTS-AI Meeting  
**Date:** November 2, 2023  

---

## Purpose of the Meeting
- **Review progress on VPC setup** for connecting containers to DynamoDB.  
- **Finalize sensor data ingestion into AWS DynamoDB.**  
- **Plan updates to SharedAirDFW website** to use DynamoDB instead of PostgreSQL.  
- **Continue work on architecture documentation and closing project milestones.**  

---

## Key Discussion Points

### 1. **VPC Setup & Connectivity**
- **Current issue:** Shared VPC does not support the **private endpoints** needed for container-DynamoDB connectivity.  
- **Action:**  
  - Networking team requested **IP ranges and network architecture documentation** for creating a **dedicated VPC**.  
  - Document prepared by Nevaton and delivered to networking group; awaiting feedback and scheduling.  
- **Goal:** Once the dedicated VPC is created, containers can securely connect to DynamoDB without relying on public endpoints.  

### 2. **Sensor Data Ingestion**
- **Mints update:**  
  - All **direct-connected sensors** successfully publishing to **AWS DynamoDB** through IoT Core.  
  - **Data stored in two DynamoDB tables:** One for PM2.5 data, one for GPS data.  
  - **Partition key:** Combination of **node ID + datetime**, allowing sorting/filtering by sensor and timestamp.  
  - **Next step:** Add **LoRaWAN sensor data** to the same DynamoDB schema.  

### 3. **SharedAirDFW Website Updates**
- **Current state:** Website still pointing to **on-prem PostgreSQL**.  
- **Planned change:** Point the website container to **AWS DynamoDB**.  
- **Action:** Pavana to coordinate with Mints and the DBA team for **container updates** and database connection migration.  
- **Challenges:**  
  - Website scripts were written by a past student, requiring review of **SQL logic** and **GPS fallback handling**.  
  - Need a **working session** between Mints and the DBA team to plan migration.  

### 4. **Architecture Diagram & Documentation**
- **Status:** Architecture diagram is still under review.  
- **Susmita shared the latest draft** with Chris for feedback.  
- **Plan:** Finalize diagram by **next week** for inclusion in Confluence.  

### 5. **Project Closure Planning**
- **Chris emphasized** the need to set an **end date** for the migration effort and plan **transition to operational mode**.  
- **Action:** Next Thursday’s meeting will focus on **aligning milestones** and finalizing **closure steps**.  

---

## Action Items
1. **Create dedicated VPC** for DynamoDB/container connectivity.  
2. **Add LoRaWAN sensor data** to DynamoDB.  
3. **Coordinate working session** with Mints and DBA team to update SharedAirDFW container.  
4. **Finalize architecture diagram** for Confluence.  
5. **Plan project closure:** Set milestones and timeline for finalizing the migration.  

---

## Key Decisions
- **Dedicated VPC** required for secure container-to-DynamoDB communication.  
- **DynamoDB** confirmed as the target database for both direct-connected and LoRaWAN sensor data.  
- **SharedAirDFW website** will migrate from PostgreSQL to DynamoDB.  
- **Next meeting:** Finalize project milestones and set an end date for migration efforts.  

---

## Next Steps
- **Complete LoRaWAN ingestion** into DynamoDB.  
- **Reconfigure website container** for DynamoDB integration.  
- **Finalize architecture documentation** by next week.  
- **Prepare project closure plan** for discussion in the next meeting.  


# Comprehensive Summary of MINTS-AI Meetings  

## Meeting 1: November 6, 2023  

### Purpose:  
- **Clarify database architecture** for SharedAirDFW migration.  
- **Resolve confusion over database strategy:** PostgreSQL RDS vs. DynamoDB.  
- **Align networking/VPC requirements** for container-database connections.  

### Key Points:  
- **Current State:**  
  - **On-prem PostgreSQL** (production).  
  - **AWS RDS (PostgreSQL)** (migrated but not live).  
  - **DynamoDB** discussed as a potential alternative for unstructured data.  
- **Confusion over database selection:**  
  - DynamoDB was mentioned due to **unstructured sensor data** handling benefits.  
  - However, **current networking diagrams** only reference **PostgreSQL**, causing misalignment.  
- **Action:** Need to **finalize architecture**: decide on **PostgreSQL vs. DynamoDB**.  
- **Next Steps:**  
  - Schedule a **working session** with **mints,AWS team and DBA team** to finalize database selection.  
  - **Update architecture diagrams** (Neviton’s diagram currently missing DynamoDB).  
  - **Dedicated VPC** setup still pending (tied to final database decision).  

### Decisions & Action Items:  
1. **Working session** to align database strategy.  
2. **Clarify database selection** (PostgreSQL RDS vs DynamoDB).  
3. **Update diagrams** to reflect correct database and networking boundaries.  
4. **Schedule follow-up** with all stakeholders for final architecture agreement.  

---

## Meeting 2: November 9, 2023  

### Purpose:  
- **Finalize decision on database selection** for SharedAirDFW migration.  
- **Plan live data ingestion** for AWS environment.  
- **Outline steps for updating website containers** to connect to the new database.  

### Key Points:  
- **Database Decision:**  
  - **RDS PostgreSQL (Aurora)** chosen as the primary database.  
  - **DynamoDB** no longer required (to avoid rewriting ingestion/application logic).  
- **Data Ingestion:**  
  - **Two existing databases:**  
    - PostgreSQL RDS (AWS).  
    - DynamoDB (AWS).  
  - **Action:** Focus on **PostgreSQL RDS** and discontinue DynamoDB pipeline.  
  - **Live data:** Currently **not being pushed** to AWS RDS.  
  - **Solution:** Set up a **VM (EC2 T2 micro)** to run **existing Python ingestion scripts and cron jobs**.  
- **Website Updates:**  
  - SharedAirDFW website **still pointing to on-prem database**.  
  - **Action:** Reconfigure website container to **connect to AWS RDS**.  
- **Other considerations:**  
  - Use **AWS IoT Analytics** for real-time pipeline management.  
  - Python ingestion process to run on a **dedicated EC2 instance** (Ubuntu).  
  - **DBA team & consultants** will assist with container-to-database connection.  

### Decisions & Action Items:  
1. **PostgreSQL RDS** confirmed as the final database (DynamoDB removed).  
2. **Spin up EC2 T2 micro** for running Python ingestion scripts (Chris Simmons to set up).  
3. **SSH keys from Mints** to be added for EC2 access.  
4. **Reconfigure SharedAirDFW container** to point to AWS RDS.  
5. **DBA team & consultants** to support integration.  
6. **Project closure planning:** Align milestones and finalize deliverables in next sessions.  





