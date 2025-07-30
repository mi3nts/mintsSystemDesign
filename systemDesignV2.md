# CIRC Serices


## VIRSH Machine 

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

### www1.circ.utdallas.edu – Summary
#### Overview
VM?: Yes (it’s a virtual machine).
OS: CentOS Linux 8.
Relevant Software: nginx (used as a web server and reverse proxy).
Backups: /var/www is mounted through IO (likely backed up by IO storage system).
Storage: Less than 30 GB on this VM.

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
