
# MintsWiki (MediaWiki) Local Deployment Upgrade Plan

This document summarizes the **current cloud infrastructure** for the MintsWiki server and proposes an **on‑premises upgrade** to improve performance, reliability, and scalability.

---

## 1. Current Cloud Infrastructure

| Name         | Type      | vCPU | RAM  | Purpose                    |
|--------------|-----------|------|------|---------------------------|
| **MintsWiki**| t3.micro  | 1    | 1 GB | **MediaWiki server** – Hosts project documentation using MediaWiki, serving wiki pages, images, and files. |

**Current load:**  
- **Platform:** MediaWiki (exact version can be checked at `Special:Version`).  
- **Traffic:** Light to moderate (internal documentation).  
- **Storage:** Minimal (~10–20 GB including images and database).  
- **Limitations:** Low vCPU and RAM restrict performance under concurrent access; limited storage for long‑term growth.

---

## 2. Target Upgrade Requirements
- **Performance:** Improve responsiveness for multiple concurrent editors and readers.  
- **Storage:** Provide room for growth (uploaded files, images, page history).  
- **Security:** Ensure HTTPS, regular backups, and proper access controls.  
- **Resilience:** Reduce downtime and provide backup/restore processes.  

---

## 3. Proposed Local Infrastructure (Upgraded)

| Component          | vCPU | RAM  | Storage | Notes                                |
|--------------------|------|------|---------|--------------------------------------|
| **MediaWiki Server** | 4    | 8 GB | 100 GB  | Hosts MediaWiki, database, and uploaded files. |

**Total:** **4 vCPUs, 8 GB RAM, 100 GB SSD storage.**

---

## 4. Software Stack
- **OS:** Ubuntu Server 22.04 LTS  
- **Web Server:** Apache2 (recommended for MediaWiki)  
- **Database:** MariaDB (or MySQL)  
- **PHP:** v8.1+ (required for modern MediaWiki versions)  
- **MediaWiki:** Same version as current cloud instance (upgrade optional).  
- **HTTPS:** Let’s Encrypt for SSL.  
- **Backups:** Automated daily database dumps and weekly off‑site file backups.  

---

## 5. Migration Plan
1. **Backup current wiki:**  
   - Export MariaDB/MySQL database (`mysqldump`).  
   - Copy `/images/` directory (uploaded files).  
   - Backup `LocalSettings.php` (configuration and extensions).  
2. **Provision local server:**  
   - Install Ubuntu Server, Apache2, MariaDB, PHP, and MediaWiki.  
3. **Restore MediaWiki:**  
   - Import database dump.  
   - Copy `/images/` and `LocalSettings.php` to the new server.  
4. **Test:**  
   - Verify page rendering, templates, extensions, and images.  
   - Check user authentication and permissions.  
5. **Enable HTTPS:**  
   - Configure Let’s Encrypt SSL certificates.  
6. **Set up backups:**  
   - Daily DB dumps.  
   - Weekly off‑site file backups.  

---

## 6. Key Improvements
- **Performance:** 4× increase in vCPUs and 8× RAM.  
- **Storage:** From ~20 GB to 100 GB, allowing long‑term growth.  
- **Security:** HTTPS, role‑based access, and automated backups.  
- **Resilience:** Local control with documented backup/restore processes.

---
