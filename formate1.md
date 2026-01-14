# VulnHub Machine Solution Writeup: [Machine Name]

## 1. Introduction & Overview
* **Machine Name:** [Name of the VM, e.g., Kioptrix Level 1]
* **Difficulty:** [Easy / Medium / Hard]
* **Description:** [Brief summary of the machine's purpose. Mention if it focuses on specific vulnerabilities like SQLi, Buffer Overflows, or misconfigurations.]
* **Goal:** [e.g., Acquire root access, capture the flag.txt]
* **Tools Used:**
    * [Tool 1: e.g., Nmap]
    * [Tool 2: e.g., Netdiscover]
    * [Tool 3: e.g., Searchsploit]
    * [Tool 4: e.g., GCC]

---

## 2. Lab Setup
* **Virtualization Software:** [e.g., VMware Workstation / VirtualBox]
* **Attacker Machine:** [e.g., Kali Linux 2024.x]
* **Target Machine:** [Name of the VulnHub VM]
* **Network Configuration:** [e.g., NAT Network / Bridged Adapter]

---

## 3. Reconnaissance & Initial Access

### Network Discovery
**Goal:** Identify the IP address of the target machine within the network.

* **Tool:** `netdiscover` / `arp-scan`
* **Command:**
    ```bash
    sudo netdiscover -i [interface] -r [IP_range]
    ```
* **Result:**
    * **Target IP:** `[Insert IP Address]`
    * **MAC Address:** `[Insert MAC Address]`

### Port Scanning & Service Enumeration
**Goal:** Identify open ports and the services running on them to map the attack surface.

* **Tool:** `nmap`
* **Command:**
    ```bash
    nmap -sC -sV -A -p- [Target_IP]
    ```
    *(Flags: `-sC` default scripts, `-sV` version detection, `-A` aggressive scan, `-p-` all ports)*

**Key Findings:**

| Port | Protocol | Service | Version | Key Notes |
| :--- | :--- | :--- | :--- | :--- |
| **22** | TCP | SSH | OpenSSH [Ver] | [e.g., Outdated version] |
| **80** | TCP | HTTP | Apache [Ver] | [e.g., Default page visible] |
| **139** | TCP | SMB | Samba | [e.g., Workgroup name] |
| **443** | TCP | HTTPS | OpenSSL | [e.g., Mod_ssl version] |

---

## 4. Detailed Enumeration & Vulnerability Identification

### Service Enumeration (SMB / RPC)
**Goal:** Extract user lists, shares, or configuration info from file-sharing services.

* **Tool:** `enum4linux`
* **Command:**
    ```bash
    enum4linux -a [Target_IP]
    ```
* **Findings:**
    * **Users Found:** `[List users, e.g., root, admin, guest]`
    * **Workgroup:** `[Workgroup Name]`
    * **Shares:** `[List accessible shares]`

* **Tool:** `rpcclient`
* **Command:**
    ```bash
    rpcclient -U "" [Target_IP]
    ```
* **Result:** [Did null session work? Did you find RID cycling opportunities?]

### Web Server Enumeration
**Goal:** Find hidden directories, sensitive files, or vulnerable CGI scripts.

* **Tool:** `dirb` / `gobuster` / `feroxbuster`
* **Command:**
    ```bash
    dirb http://[Target_IP]
    ```
* **Findings:**
    * Found Directories: `/cgi-bin`, `/manual`, `/admin`
    * Interesting Files: `robots.txt`, `config.php`

### Vulnerability Search
**Goal:** Correlate service versions with known exploits.

* **Tool:** `searchsploit` / Exploit-DB / Google
* **Search Queries:**
    1.  `searchsploit OpenSSH [Version]`
    2.  `searchsploit Apache [Version]`
    3.  `"Apache mod_ssl OpenSSL exploit [Version]"`

**Selected Vulnerability:**
* **Vulnerability Name:** [e.g., OpenSSL < 2.8.7 Remote Buffer Overflow]
* **CVE ID:** [e.g., CVE-2002-0082]
* **Why this one?** [Explain why this specific vulnerability was chosen over others—e.g., "Version match exactly confirmed via Nmap"]

---

## 5. Exploitation

### Exploit Retrieval & Preparation
* **Exploit Source:** [e.g., Exploit-DB ID 764.c]
* **Retrieval Command:**
    ```bash
    searchsploit -m 764.c
    ```

### Compilation & Troubleshooting
**Challenge:** [Describe any errors encountered, e.g., "Exploit failed to compile due to missing SSL libraries."]

**Fixes Applied:**
1.  Installed dependencies: `sudo apt-get install libssl-dev`
2.  Modified source code: [e.g., Added `#include <openssl/rc4.h>`, updated wget URL for payload]
3.  **Compilation Command:**
    ```bash
    gcc -o exploit 764.c -lcrypto
    ```

### Execution
**Goal:** Trigger the buffer overflow/exploit to gain a shell.

* **Identifying Offset:**
    * Ran `./exploit` to view supported offsets.
    * Selected Offset: `[e.g., 0x6b]` (Matches RedHat 7.2 / Apache 1.3.20).
* **Final Payload Command:**
    ```bash
    ./exploit [Target_Offset] [Target_IP] [Connection_Range]
    # Example: ./openf 0x6b 192.168.1.106 50
    ```
* **Outcome:** [e.g., "Exploit successfully executed, spawning a shell."]

---

## 6. Post-Exploitation

### Privilege Verification
**Goal:** Confirm root access.

* **Command:** `id`
* **Result:** `uid=0(root) gid=0(root)`
* **Command:** `whoami`
* **Result:** `root`

### Flag Capture
* **Location:** `/root/flag.txt` or `/tmp`
* **Proof:**
    ```text
    [Paste the contents of the flag here or insert screenshot]
    ```

---

## 7. Conclusion & Lessons Learned

### Summary
[Brief recap: "Starting with Nmap, we identified an outdated Apache server. Enumeration pointed to a mod_ssl vulnerability. After fixing compilation errors in an old C exploit, we successfully overflowed the buffer to gain root access directly."]

### Key Learnings
1.  **Enumeration is Key:** [e.g., "Aggressive scanning reveals specific version numbers vital for exploit selection."]
2.  **Code Adaptation:** [e.g., "Exploits from 2002 rarely work out of the box; learning to debug C code and install missing headers is a critical skill."]
3.  **Research:** [e.g., "Searchsploit isn't enough; Google often holds the context needed for older CVEs."]
