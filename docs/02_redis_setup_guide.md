# Redis Session Server Configuration Guide

This documentation covers the setup of Redis as a session/cache server.

**Server:** VM 6 (Redis)  
**IP Address:** 192.168.100.15

---

## Overview

Redis will serve as the centralized session storage for your web application, allowing session data to be shared across multiple web servers. This setup is much simpler than the database configuration.

---

## Step 1: Install Redis

Update the package repositories and install Redis server:

```bash
sudo apt update
sudo apt install redis-server -y
```

---

## Step 2: Configure Remote Access

By default, Redis only accepts connections from localhost for security. We need to configure it to accept connections from our internal network.

### Edit Configuration File

Open the Redis configuration file:

```bash
sudo nano /etc/redis/redis.conf
```

### Modify the Bind Address

Find the line that starts with `bind`:

```
bind 127.0.0.1 ::1
```

Change it to:

```
bind 0.0.0.0
```

This allows Redis to listen on all network interfaces.

**Save and Exit:** Press `Ctrl+O`, `Enter`, then `Ctrl+X`

---

## Step 3: Apply Changes

Restart the Redis service to load the new configuration:

```bash
sudo systemctl restart redis-server
```

---

## Step 4: Verify Configuration

Check if Redis is actively listening on the network interface.

```bash
sudo ss -tuln | grep 6379
```

### Expected Output

✅ **Success:** You should see `0.0.0.0:6379` in the output  
❌ **Failure:** If you see `127.0.0.1:6379`, the configuration didn't save correctly. Go back to Step 2.

---

## Verification Complete

Once you see `0.0.0.0:6379` in the output, your Redis session server is ready! 🎉

---

## Security Notes

**For Production Environments:**

This configuration allows unrestricted access from any IP address. For production deployments, consider:

1. **Bind to specific IPs:** Instead of `0.0.0.0`, use specific internal IPs
2. **Enable authentication:** Set a password using `requirepass` in redis.conf
3. **Use firewall rules:** Restrict access to Redis port (6379) to only your web servers
4. **Enable TLS/SSL:** Encrypt connections between clients and Redis

For this lab environment, the current configuration is sufficient.

---

## Next Steps

The Redis server is now ready to accept session data from your web servers. The next phase will cover configuring your web application servers to use this Redis instance for session storage.