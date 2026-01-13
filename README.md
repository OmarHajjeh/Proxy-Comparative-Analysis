# 💻 Proxy Comparative Analysis & High-Availability Lab

This repository documents a experimental lab setup designed to compare the performance, configuration, and resilience of three popular open source proxy/load-balancing technologies: **Nginx**, **HAProxy**, and **Caddy**.

The architecture simulates a modern, **Stateless 3-Tier Web Application** to ensure data persistence and high availability.

---

### 🎯 Lab Goals
The primary objective is to evaluate how each proxy handles traffic in a redundant environment:
1.  **Load Balancing:** Distributing traffic across multiple backend servers.
2.  **Stateless Session Management:** Using Redis to maintain user logins across different web servers.
3.  **Data Resilience:** Implementing MySQL Master-Slave replication and shared NFS storage.
4.  **Automatic Failover:** Testing system behavior when primary nodes go offline.

---

### 🏛️ Architecture & Topology
The lab is hosted on **VMware** using an isolated **Host-Only** network. All VMs use static IPs to ensure stable communication between layers.

| Role | VM Name | OS | Software | Static IP |
| :--- | :--- | :--- | :--- | :--- |
| **Proxy 1** | `Proxy-HA` | Ubuntu | **HAProxy** | `192.168.100.10` |
| **Proxy 2** | `Proxy-NGX` | Ubuntu | **Nginx** | `192.168.100.11` |
| **Proxy 3** | `Proxy-Cdy` | Ubuntu | **Caddy** | `192.168.100.12` |
| **Web Server 1** | `Web-A` | Ubuntu | **Apache + PHP** | `192.168.100.13` |
| **Web Server 2** | `Web-B` | Ubuntu | **Apache + PHP** | `192.168.100.14` |
| **Session Cache**| `Redis` | Ubuntu | **Redis-Server** | `192.168.100.15` |
| **Data Primary** | `Data-A` | Ubuntu | **MySQL (Master) + NFS** | `192.168.100.16` |
| **Data Mirror** | `Data-B` | Ubuntu | **MySQL (Slave) + NFS** | `192.168.100.17` |

---

### 🛠️ Technical Implementation

#### 1. The Gateway Layer (Proxies)
* **Nginx Configuration:** Implemented an `upstream` block to balance traffic between `.13` and `.14`.
* **Header Forwarding:** Configured to pass the client's real IP (`X-Real-IP`) to the backends for accurate logging.

#### 2. The Application Layer (Stateless Web Servers)
* **PHP-Redis Integration:** Modified `php.ini` to store session data in the central Redis VM (`.15`). This prevents users from being logged out when the load balancer switches their server.
* **NFS Mounting:** Both Web-A and Web-B mount a shared folder from the Data-A server to `/var/www/html/uploads`, ensuring images uploaded to one server are visible on the other.

#### 3. The Data Layer (Persistent Storage)
* **MySQL Replication:** Configured Master-Slave replication. All data written to `Data-A` is mirrored in real-time to `Data-B`.
* **Redundancy:** Tested "Offline Sync" by shutting down the Slave, adding data to the Master, and verifying the Slave caught up automatically upon reboot.
