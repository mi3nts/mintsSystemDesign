
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

## Closing Remarks
- **Partnership between Research & OIT:** Dr. Larry emphasized collaboration for building robust infrastructure that can benefit **multiple researchers** at UTD.  
- **Importance of reliability:** Needed for **community trust** and **city/county partnerships**.  
- **Recognition of efforts:** Appreciation expressed for Chris and team for keeping the current system running despite limitations.  

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
- Access issues for Dr. Larry were reported; the **Cloud team will resolve** and ensure **Lakitha and John gain access**.  
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






