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

The lab is hosted on a **virtualized environment** (VirtualBox, VMware) using five interconnected Virtual Machines (VMs) on an isolated internal network.

| Role | OS | Software Tested |
| :--- | :--- | :--- |
| **Proxy 1** | Linux (Ubuntu) | **HAProxy** |
| **Proxy 2** | Linux (Ubuntu) | **Nginx** |
| **Proxy 3** | Linux (Ubuntu) | **Caddy** |
| **Backend 1** | Linux (Ubuntu) | **Apache** (Content: "Server A") |
| **Backend 2** | Linux (Ubuntu) | **Apache** (Content: "Server B") |
