# Data Server Configuration Guide

This documentation covers the setup of two data servers (Data-A and Data-B).

* **Part 1:** MySQL Master-Slave Replication (Current)
* **Part 2:** NFS Server Configuration (To Follow)

---

## Part 1: MySQL Replication Setup

### Prerequisites

* **Data-A:** Acts as the MySQL Master (IP assumed: `192.168.100.16`)
* **Data-B:** Acts as the MySQL Slave

---

## Step 1: Install Software

Run the following commands on **BOTH** Data-A and Data-B.

Update repositories and install the necessary packages for both MySQL and the future NFS setup.

```bash
sudo apt update
sudo apt install mysql-server nfs-kernel-server -y
```

### Network Configuration

For the purpose of this lab/setup, disable the firewall to prevent connection issues.

```bash
sudo ufw disable
```

---

## Step 2: Configure MySQL Master (Data-A)

We need to allow remote connections and enable binary logging so the Slave can track changes.

### 1. Edit Configuration

Open the MySQL configuration file:

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Find and modify the following lines to match:

```ini
# Allow remote connection
bind-address            = 0.0.0.0

# Unique ID for Master
server-id               = 1

# Enable replication log
log_bin                 = /var/log/mysql/mysql-bin.log

# Only replicate our specific database
binlog_do_db            = my_lab_db
```

Save the file (`Ctrl+O`, `Enter`) and exit (`Ctrl+X`).

### 2. Restart MySQL

Apply the changes:

```bash
sudo systemctl restart mysql
```

### 3. Set up Users & Database

Log into the MySQL shell:

```bash
sudo mysql
```

Run the following SQL commands:

```sql
-- 1. Create the Application Database
CREATE DATABASE my_lab_db;

-- 2. Create the Replication User (The Slave uses this to log in)
CREATE USER 'replicator'@'%' IDENTIFIED WITH mysql_native_password BY 'secret_password';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'%';

-- 3. Create the App User (The Web Servers use this)
CREATE USER 'app_user'@'%' IDENTIFIED WITH mysql_native_password BY 'app_password';
GRANT ALL PRIVILEGES ON my_lab_db.* TO 'app_user'@'%';

-- 4. Apply changes
FLUSH PRIVILEGES;
```

### 4. Lock and Get Status

**Crucial Step:** We need the coordinate position of the data so the Slave knows exactly where to start reading.

Run these commands inside the MySQL shell:

```sql
USE my_lab_db;
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
```

📝 **NOTE:** Write down the values for `File` (e.g., `mysql-bin.000001`) and `Position` (e.g., `1234`). You will need these for Step 3.

---

## Step 3: Configure MySQL Slave (Data-B)

### 1. Edit Configuration

Open the MySQL configuration file:

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Find and modify the following lines:

```ini
bind-address            = 0.0.0.0

# MUST be different from Master (Master is 1)
server-id               = 2 

# Note: Do not add binlog_do_db here
```

Save and restart MySQL:

```bash
sudo systemctl restart mysql
```

### 2. Link to Master

Log into the MySQL shell:

```bash
sudo mysql
```

Configure the slave to look at the master. Replace the `MASTER_LOG_FILE` and `MASTER_LOG_POS` with the data you recorded in Step 2.4.

```sql
CHANGE MASTER TO
MASTER_HOST='192.168.100.16',
MASTER_USER='replicator',
MASTER_PASSWORD='secret_password',
MASTER_LOG_FILE='mysql-bin.000001',  -- REPLACE with your file name
MASTER_LOG_POS=1234;                 -- REPLACE with your position number

START SLAVE;
```

---

## Step 4: Verification

To verify the replication is working, run the following inside the MySQL shell on Data-B:

```sql
SHOW SLAVE STATUS\G
```

Look for these two specific lines near the top of the output. Both must say **Yes**:

* `Slave_IO_Running: Yes` (Connected to Master)
* `Slave_SQL_Running: Yes` (Applying changes)

✅ **If both are YES, your database replication is active.**

---

## Next Steps

Part 2 (NFS Server Configuration) will be added later.
