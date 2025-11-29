# 💻 Proxy Comparative Analysis

This repository documents an experimental lab setup designed to compare the performance, configuration complexity, and feature sets of three popular proxy and load-balancing technologies: **HAProxy**, **Nginx**, and **Caddy**.

### 🎯 Lab Goal

The primary goal of this experiment is to deploy each proxy in an identical virtualized environment to test and document to see which one is better overall :

1.  **Load Balancing** 
2.  **Resilience** Failover testing when one backend server goes offline.
3.  **Configuration** Comparing the ease, verbosity, and maintainability of setup for each technology.
4.  **Functionality and features** 

---

### 🏛️ Architecture and Setup

The lab is hosted on a **virtualized environment** ( VMware ) using interconnected Virtual Machines (VMs) on an isolated internal network.

| Role | OS | Software Tested | IP |
| :--- | :--- | :--- | :--- |
| **Backend 1** | Linux (Ubuntu) | **Apache** (Content: "Server A") | 192.168.100.10 |
| **Backend 2** | Linux (Ubuntu) | **Apache** (Content: "Server B") | 192.168.100.11 |
| **Proxy 1** | Linux (Ubuntu) | **Nginx** | 192.168.100.12 |
| **Proxy 2** | Linux (Ubuntu) | **HAProxy** | 192.168.100.13 |
| **Proxy 3** | Linux (Ubuntu) | **Caddy** | 192.168.100.14 |
| **Session Cache** | Linux (Ubuntu) | **Redis** | 192.168.100.15 |
| **Persistent Data 1** | Linux (Ubuntu) | **MySQL Master / NFS Share (primary)** | 192.168.100.16 |
| **Persistent Data 2** | Linux (Ubuntu) | **MySQL Slave / NFS Mirror** | 192.168.100.17 |
