# HTB Machine: Ignition
* **OS:** Linux
* **Difficulty:** Very Easy (Tier 0)
* **Date Started:** July 2026
## Step-by-Step Walkthrough

### Step 1
Run nmap to determine the version of the service running on port 80:
```bash
sudo nmap -sV -sC 10.129.1.27
```
Key Findings:
* Target IP: 10.129.1.27
* Port 80 (HTTP) is open.
* Nginx web server software version: nginx 1.14.2.

### Step 2
I initially tried to navigate over to the web page at `http://10.129.1.27`, but it resolved to `http://ignition.htb/` and gave the error: "Hmm. We’re having trouble finding that site." 

I checked the network tab but the request headers did not display an explicit error code. Checking the hint for the problem revealed we were supposed to use curl to investigate:
```bash
curl -v http://10.129.1.27
```
This outputted the redirection error code we were looking for: 302 Found.

### Step 3
To understand why the browser failed to load the page, I researched the issue. An IP address is just a physical location on a network. A single Nginx server at an IP address can host dozens of completely different websites simultaneously using virtual hosting. 

When typing the raw IP address into the browser, Nginx looks at the HTTP request and sees `Host: 10.129.1.27`. Because it does not recognize that IP as a specific website configuration, it fails to route it properly. Nginx needs to see a specific domain name (like `Host: ignition.htb`) in the HTTP header to route traffic to the Magento application.

To fix this, I edited the local hosts file:
```bash
sudo vim /etc/hosts
```
I added the line `10.129.1.27 ignition.htb`, saved, and exited. Navigating to the web application then worked.

### Step 4
Directory busting was performed using gobuster to find hidden paths:
```bash
gobuster dir -u http://ignition.htb -w /usr/share/wordlists/dirb/common.txt -t 30
```
The `-t 30` flag stands for threads, meaning the tool checks 30 words from the wordlist at a time. This yielded a few results, including the `/admin` page.

### Step 5
The platform requires an administrative password. Magento features defenses against brute forcing passwords; too many incorrect attempts lock the user out for a couple of minutes. This required guessing manual entries through trial and error.

The process involved researching common default administrative passwords. The correct password was entry number 48 on the list: `qwerty123`.

*Note: I did not manually guess all 48 passwords; because this is an older machine, I looked up the standard password list reference. This mimics real-world scenarios where known password lists are prioritized based on platform defaults.*
