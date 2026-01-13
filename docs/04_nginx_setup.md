
## Part 1: Configure Nginx Load Balancer

**Target:** VM 3 (`192.168.100.12`)

### Step 1: Install Nginx

Run the following commands to install the web server:

```bash
sudo apt update
sudo apt install nginx -y
```
### Step 2: Configure the Load Balancer

Create a specific configuration file for the load balancing logic.

1. **Create the file:**
```bash
sudo nano /etc/nginx/conf.d/load-balancer.conf
```




> **Note:** If `conf.d` does not exist, use `/etc/nginx/sites-available/`.


2. **Paste the configuration:**
```nginx
# 1. Define the group of backend servers
upstream my_backend_servers {
    # Default algorithm is Round Robin
    server 192.168.100.10;  # Apache A
    server 192.168.100.11;  # Apache B
}

# 2. Define the Proxy Server listener
server {
    listen 80;

    location / {
        # Pass traffic to the upstream group
        proxy_pass http://my_backend_servers;

        # Forward client's real IP (Best practice for logging)
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

```


3. **Disable the Default Site:**
This prevents conflicts with the default "Welcome" page.
```bash
sudo rm /etc/nginx/sites-enabled/default

```


4. **Test and Restart:**
```bash
sudo nginx -t   # Should return "syntax is ok"
sudo systemctl restart nginx

```



### Step 3: Verification

To verify stateless sessions:

1. Navigate to `http://192.168.100.12/test_session.php` in your browser.
2. Refresh repeatedly.
3. **Success Criteria:**
* **Served By:** Switches between `.10` and `.11`.
* **Count:** Increases continuously (1, 2, 3...) regardless of the server switch (proving Redis/Session store is working).



---

## Part 2: Application Data & Storage (NFS)

**Target:** Web-A (`.10`) and Web-B (`.11`)

We must ensure both web servers share the same "Uploads" folder via NFS.

### Step 1: Mount Shared Storage

Run on **BOTH** Web Servers:

1. **Create local directory:**
```bash
sudo mkdir -p /var/www/html/uploads

```


2. **Mount the NFS share:**
```bash
sudo mount 192.168.100.16:/var/nfs/shared_data /var/www/html/uploads

```


3. **Make it permanent (Optional):**
Add the following line to `/etc/fstab`:
```text
192.168.100.16:/var/nfs/shared_data /var/www/html/uploads nfs defaults 0 0

```


4. **Set Permissions:**
Give Apache (`www-data`) permission to write to the mount.
```bash
sudo chown -R www-data:www-data /var/www/html/uploads

```



---

## Part 3: Database Connection

**Target:** Web-A (`.10`) and Web-B (`.11`)

### Step 1: Create Database Config

Create `db_config.php` on Web-A and copy it to Web-B.

```bash
sudo nano /var/www/html/db_config.php

```

**File Content:**

```php
<?php
// Centralized credentials
$servername = "192.168.100.16"; // Currently Pointing to Master
$username = "app_user";
$password = "app_password";
$dbname = "my_lab_db";

$conn = new mysqli($servername, $username, $password, $dbname);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
?>

```

### Step 2: Initialize Database Table

**Target:** Data-A (`.16`) ONLY

Run the following SQL commands to create the structure:

```bash
sudo mysql

```

```sql
USE my_lab_db;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    photo_path VARCHAR(255)
);

-- Verify creation
SHOW TABLES;

```

---

## Part 4: The Profile Application

**Target:** Web-A (`.10`) and Web-B (`.11`)

Create the application script that tests SQL and NFS simultaneously.

```bash
sudo nano /var/www/html/profile.php

```

**File Content:**

```php
<?php
include 'db_config.php';
session_start(); // Verifies session persistence

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $name = $_POST['username'];
    
    // Handle File Upload (Save to NFS)
    $target_dir = "uploads/";
    $target_file = $target_dir . basename($_FILES["profile_pic"]["name"]);
    move_uploaded_file($_FILES["profile_pic"]["tmp_name"], $target_file);

    // Handle Database Insert (Save to SQL)
    $sql = "INSERT INTO users (name, photo_path) VALUES ('$name', '$target_file')";
    
    // Post-Redirect-Get (PRG) Pattern
    if ($conn->query($sql) === TRUE) {
        header("Location: profile.php");
        exit(); 
    } else {
        echo "Error: " . $conn->error;
    }
}

// Retrieve Data
$result = $conn->query("SELECT * FROM users ORDER BY id DESC");
?>

<h2>User Profile System</h2>
<h4>Served By: <?php echo gethostname(); ?> (IP: <?php echo $_SERVER['SERVER_ADDR']; ?>)</h4>

<form method="post" enctype="multipart/form-data">
    Name: <input type="text" name="username" required>
    Photo: <input type="file" name="profile_pic" required>
    <input type="submit" value="Save Profile">
</form>

<hr>
<h3>Registered Users:</h3>
<?php
if ($result->num_rows > 0) {
    while($row = $result->fetch_assoc()) {
        echo "<div style='border:1px solid #ccc; padding:10px; margin:5px;'>";
        echo "<strong>Name:</strong> " . $row["name"] . "<br>";
        echo "<img src='" . $row["photo_path"] . "' width='100'><br>";
        echo "</div>";
    }
} else {
    echo "No users found.";
}
?>

```

---

## Part 5: Final Testing

Navigate to the **Proxy IP**: `http://192.168.100.12/profile.php`

### 1. The Write Test

* Enter a Name (e.g., "Test User").
* Upload a small photo.
* Click **Save Profile**.

### 2. The Persistence Test

* Refresh the page 5 times.
* **Observation:** The "Served By" IP should switch between `.10` and `.11`, but the Name and Photo must remain visible. This proves both servers are reading from the centralized SQL and NFS.
