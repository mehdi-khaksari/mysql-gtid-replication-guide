# 🚀 MySQL GTID-Based Replication Setup Guide
This document outlines the steps to configure GTID-based replication between MySQL servers (source → replica).

### 🔍 Prerequisites
- Two MySQL servers (source and replica)

- Network connectivity between them (port 3306)

- MySQL installed on both servers


## 1️⃣ Install MySQL on Both Servers
```bash
sudo apt update  
sudo apt install mysql-server  
sudo systemctl start mysql.service  
```

## 🔥 Firewall Configuration
### 🌐 Allow Replica Access (Source Server):
```bash
sudo ufw allow from replica_server_ip to any port 3306  
```


## 2️⃣ Configure the Source Server
### 📝 Edit MySQL Configuration:
```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf  
```
### ⚙️ Add/Modify Settings:
```
[mysqld]  
bind-address          = source_server_ip  # Allow remote connections  
server-id             = 1                 # Must be unique  
log_bin               = /var/log/mysql/mysql-bin.log  # Enable binary logging  
gtid_mode             = ON                # Enable GTID  
enforce-gtid-consistency = ON             # Ensure GTID compatibility  
```
### 🔄 Restart MySQL:
```bash
sudo systemctl restart mysqld.service  
```
### 📌 Notes:

- server-id=1 uniquely identifies the source.

- GTID mode simplifies replication management.

## 3️⃣ Configure the Replica Server
###  Edit MySQL Configuration:
```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf  
```
### ⚙️ Add/Modify Settings:
```
[mysqld]  
server-id             = 2                 #  Different from source  
log_bin               = /var/log/mysql/mysql-bin.log  
relay-log             = /var/log/mysql/mysql-relay-bin.log  
gtid_mode             = ON  
enforce-gtid-consistency = ON  
log-replica-updates   = ON                #  Record replica updates  
skip-replica-start    = ON                #  Prevent auto-start before setup  
```
### 🔄 Restart MySQL:
```bash
sudo systemctl restart mysqld.service  
```
## 4️⃣ Create Replication User on Source
### 🔑 Run in MySQL Shell (Source Server):
```sql
CREATE USER 'replication_user'@'replica_server_ip'  
IDENTIFIED WITH mysql_native_password BY 'secure_password';  

GRANT REPLICATION SLAVE ON *.* TO 'replication_user'@'replica_server_ip';  

FLUSH PRIVILEGES;  

SET @@GLOBAL.read_only = ON;  -- Optional: Make source read-only during setup  
```

## 5️⃣ Configure Replication on Replica Server
### 🔄 Run in MySQL Shell (Replica Server):
```sql
CHANGE REPLICATION SOURCE TO  
SOURCE_HOST='source_ip_address',  
SOURCE_USER='replication_user',  
SOURCE_PASSWORD='secure_password',  
SOURCE_AUTO_POSITION=1;  --  Critical for GTID-based replication  

START REPLICA;  

SET @@GLOBAL.read_only = OFF;  --  (Optional) Allow writes on replica  
```

## 6️⃣ Verify Replication Status
### ✅ Check Replica Status:
```sql
SHOW REPLICA STATUS\G  
```
### Look for:

- Replica_IO_Running: Yes

- Replica_SQL_Running: Yes

- No errors in Last_IO_Error or Last_SQL_Error

- Retrieved_Gtid_Set and Executed_Gtid_Set showing progress

# 📬 Contact
Feel free to fork, improve, and contribute!

Author: mehdi khaksari 

Email: mahdikhaksari36@gmail.com
