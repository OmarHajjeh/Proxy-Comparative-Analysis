# Stateless Web Servers Configuration Guide

This documentation covers the setup of stateless web servers that use centralized session storage and database connections.

**Servers:**
* **Web-A:** 192.168.100.10
* **Web-B:** 192.168.100.11

---

## Overview

These web servers are designed to be **stateless** - they don't store any data locally. Instead, they:
* Store sessions in **Redis** (192.168.100.15)
* Access database from **Data-A/Data-B** (MySQL cluster)
* Access shared files from **NFS server** (Data-A)

This architecture allows for true load balancing and high availability.

---

## Step 1: Install Software

Run these commands on **BOTH Web-A and Web-B**.

Install Apache web server, PHP, and the necessary connectors for Redis and MySQL:

```bash
sudo apt update
sudo apt install apache2 php libapache2-mod-php php-mysql php-redis nfs-common -y
```

**Packages Installed:**
* `apache2` - Web server
* `php` - PHP language runtime
* `libapache2-mod-php` - Apache PHP module
* `php-mysql` - MySQL database connector
* `php-redis` - Redis connector
* `nfs-common` - NFS client utilities (for shared storage)

---

## Step 2: Configure PHP to Use Redis for Sessions

By default, PHP saves login sessions to the local hard drive, which breaks load balancing. We need to configure PHP to save sessions to our centralized Redis server instead.

Run these steps on **BOTH Web-A and Web-B**.

### 1. Open PHP Configuration File

```bash
sudo nano /etc/php/*/apache2/php.ini
```

**Note:** The `*` wildcard matches your PHP version (e.g., 8.1, 8.2). Use `Tab` to auto-complete if needed.

### 2. Change Session Handler

Press `Ctrl+W` to search for `session.save_handler`.

Change from:
```ini
session.save_handler = files
```

To:
```ini
session.save_handler = redis
```

### 3. Set Session Save Path

Press `Ctrl+W` to search for `session.save_path`.

Uncomment the line (remove `;` at the beginning) and set it to your Redis server IP:

```ini
session.save_path = "tcp://192.168.100.15:6379"
```

**If you set a password on Redis**, use this format instead:
```ini
session.save_path = "tcp://192.168.100.15:6379?auth=your_password"
```

### 4. Save and Restart Apache

**Save the file:** Press `Ctrl+O`, `Enter`, then `Ctrl+X`

Restart Apache to apply changes:

```bash
sudo systemctl restart apache2
```

---

## Step 3: Create a Stateless Test Page

Now we'll create a test page to prove that sessions are stored in Redis, not locally on the web server.

### Create Test File on Web-A

Run this on **Web-A ONLY**:

```bash
sudo nano /var/www/html/test_session.php
```

Paste the following PHP code:

```php
<?php
session_start();
$count = isset($_SESSION['count']) ? $_SESSION['count'] : 1;
echo "<h1>Stateless Test</h1>";
echo "<h2>Served By: " . gethostname() . " (" . $_SERVER['SERVER_ADDR'] . ")</h2>";
echo "<h3>You have visited this page: " . $count . " times.</h3>";
$_SESSION['count'] = $count + 1;
?>
```

**Save and Exit:** Press `Ctrl+O`, `Enter`, then `Ctrl+X`

---

## Step 4: Verification

Test that the session counter is working correctly.

### Test Web-A

1. Open your browser and visit: `http://192.168.100.10/test_session.php`
2. Refresh the page multiple times
3. The counter should increment: 1, 2, 3, 4...

### What This Proves

The counter incrementing shows that:
* PHP is successfully connecting to Redis
* Session data is being stored and retrieved correctly
* Your web server is properly configured

---

## Understanding the Test Page

The test page does the following:

1. **`session_start()`** - Initiates or resumes a session
2. **Retrieves counter** - Gets the current visit count from the session (stored in Redis)
3. **Displays info** - Shows which server is responding and the current count
4. **Increments counter** - Increases the count and saves it back to Redis

The key point: **The session data lives in Redis**, not on the web server's local disk. This means any web server in your cluster can access the same session data.

---

## Next Steps

The next phase will cover setting up Nginx as a load balancer to distribute traffic between Web-A and Web-B. Once configured, you'll be able to see session persistence working across multiple servers.