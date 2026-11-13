# reven-1-boot2root-writeup
# 🎯 Reven 1 – Complete Boot2Root Walkthrough

## 📖 Introduction

Reven 1 is an intentionally vulnerable machine used to practice penetration testing, network enumeration, web application security, Linux authentication, and privilege escalation.

The objective of this challenge was to investigate the target, identify security weaknesses, obtain initial access, capture all four flags, and ultimately achieve root access.

## 🎯 Objectives

- Discover the target machine's IP address.
- Perform network scanning and service enumeration.
- Investigate the web application.
- Identify usernames and credentials.
- Obtain initial access through SSH.
- Discover all four flags.
- Investigate privilege escalation opportunities.
- Achieve root access.

**Attacker Machine:** Kali Linux  
**Target Machine:** Reven 1  
**Virtualization:** VirtualBox  
**Network:** Host-Only Adapter  
**Target IP:** TARGET_IP

---

## 🔎 Phase 1 – Reconnaissance

### 🌐 Step 1 – Network Discovery

First, I needed to identify the target machine's IP address on the virtual network.

For this purpose, I used Netdiscover, a network discovery tool that identifies active devices on a local network.

**Command:**

```bash
sudo netdiscover -r YOUR_LAB_SUBNET/24
```

**Alternative:**

```bash
sudo netdiscover
```

**What does this command do?**

- `sudo` – Runs the command with administrative privileges.
- `netdiscover` – Starts network discovery.
- `-r` – Specifies the network range.
- `YOUR_LAB_SUBNET/24` – The Host-Only network range.

After examining the discovered devices, I identified the target machine.

**Target IP:** `TARGET_IP`

![Netdiscover Output](screenshots/netdiscover.png)

*Add your Netdiscover screenshot as `netdiscover.png` inside the screenshots folder.*

### 📡 Step 2 – Nmap Scanning

After discovering the target IP, I performed an Nmap scan to identify open ports, running services, and service versions.

**Command:**

```bash
sudo nmap -sC -sV -p- TARGET_IP
```

**Command Breakdown**

- `sudo` – Runs Nmap with elevated privileges.
- `nmap` – Network scanning tool.
- `-sC` – Runs default NSE scripts.
- `-sV` – Detects service versions.
- `-p-` – Scans all TCP ports.
- `Target IP` – Specifies the machine being scanned.

**Scan Results**

The important findings included:

| Port | Service | Version / Details |
|---|---|---|
| 22/tcp | SSH | OpenSSH 6.7p1 |
| 80/tcp | HTTP | Apache 2.4.10 |
| 56642/tcp | Status | RPC service |

**Additional RPC Enumeration**

I also used:

```bash
rpcinfo -p TARGET_IP
```

This command helps enumerate RPC services running on the target.

**Why was this step important?**

Port scanning helped identify the available services and provided possible directions for further investigation.

![Nmap Results](screenshots/nmap.png)

### 🕸️ Phase 2 – Web Enumeration

### 🌍 Step 3 – Website Investigation

Since port 80 was open, I opened the target's web server in Firefox.

**URL:**

```text
http://TARGET_IP
```

The website displayed the title:

**Raven Security**

I inspected the website, navigated through available pages, and examined the HTML source code.

**What I investigated**

- Website structure.
- Available pages.
- HTML source code.
- Interesting information exposed in webpages.
- WordPress-related content.

I also investigated the web technology using WhatWeb.

**Command:**

```bash
whatweb http://TARGET_IP
```

WhatWeb identifies technologies and software used by a website.

![Website Investigation](screenshots/website.png)

### 📂 Step 4 – Directory Enumeration

Next, I performed directory enumeration to discover directories and files that were not immediately visible on the website.

**Tool:** Gobuster

**Command:**

```bash
gobuster dir -u http://TARGET_IP -w /usr/share/wordlists/dirb/common.txt -t 10
```

**Command Breakdown**

- `gobuster` – Directory enumeration tool.
- `dir` – Enables directory discovery mode.
- `-u` – Specifies the target URL.
- `-w` – Specifies the wordlist.
- `-t 10` – Uses 10 concurrent threads.

**Interesting Findings**

The scan identified several directories:

```text
/wordpress/
/manual/
/css/
/fonts/
/img/
/js/
/vendor/
```

The `/wordpress/` directory was particularly interesting and required further investigation.

**Alternative Tool: DIRB**

The following command can also perform directory enumeration:

```bash
dirb http://TARGET_IP
```

![Gobuster Output](screenshots/gobuster.png)

### 📝 Step 5 – WordPress Enumeration

After discovering the WordPress directory, I investigated the installation to identify available user accounts.

**Tool:** WPScan

**Command:**

```bash
wpscan --url http://TARGET_IP/wordpress --enumerate u
```

**Command Breakdown**

- `wpscan` – WordPress security scanner.
- `--url` – Specifies the WordPress installation.
- `--enumerate u` – Enumerates WordPress users.

**Discovered Usernames**

During enumeration, I identified usernames including:

- `micheal`
- `steven`

These usernames provided useful information for further authentication investigation.

![WPScan Results](screenshots/wpscan.png)

---

## 🔐 Phase 3 – Initial Access

### 🔑 Step 6 – Credential Investigation

After discovering usernames, I investigated the authentication attack surface to identify valid credentials.

The SSH service was available on port 22.

During the authorized lab investigation, I used credential testing against the vulnerable target and identified account information.

**SSH service:**

```text
22/tcp - OpenSSH 6.7p1
```

The following is a command recovered from your earlier lab notes for the SSH credential-testing stage:

```bash
hydra ssh://TARGET_IP -l micheal -P /usr/share/wordlists/rockyou.txt -V -t 4
```

**Command Breakdown**

- `hydra` – Login authentication testing tool.
- `ssh://` – Specifies SSH as the service.
- `-l micheal` – Specifies a username.
- `-P` – Specifies a password wordlist.
- `-V` – Displays detailed attempts.
- `-t 4` – Uses four parallel tasks.

*This records the command from your lab notes; verify which account actually produced the successful login before publishing the result.*

**Credentials Obtained**

The account you later confirmed accessing was:

```text
Username: steven
Password: [REDACTED]
```

Never publish your actual password in a public GitHub repository.

![Credential Discovery Evidence](screenshots/credentials.png)

### 💻 Step 7 – SSH Login

After obtaining the valid credentials, I established a remote connection to the target machine through SSH.

**Command:**

```bash
ssh steven@TARGET_IP
```

**Command Explanation**

- `ssh` – Secure Shell client.
- `steven` – Target username.
- `TARGET_IP` – Target IP address.

After entering the password, I successfully logged into the target machine.

**Verify Initial Access**

Once logged in, I used:

```bash
whoami
```

To identify the current user.

I also used:

```bash
id
```

To check the user's UID, GID, and group memberships.

**Result:** Initial access successfully obtained.

![SSH Login](screenshots/ssh-login.png)

---

## 🚩 Phase 4 – Flag Collection

### 🗂️ Step 8 – Filesystem Exploration

After gaining SSH access, I started investigating the target's filesystem to locate the available flags.

**Useful Linux Commands**

Check the current directory:

```bash
pwd
```

List directory contents:

```bash
ls -la
```

Navigate to the web directory:

```bash
cd /var/www
```

Explore its contents:

```bash
ls -la
```

Search for flag-related files in accessible locations:

```bash
find /var/www -type f -iname '*flag*' 2>/dev/null
```

Read an identified flag file:

```bash
cat flag2.txt
```

*The filename above is an example from your earlier walkthrough format. Keep it only if it matches your actual Reven findings.*

**Flag Collection**

During the challenge, I successfully captured all four flags.

| Flag | Status |
|---|---|
| Flag 1 | Found |
| Flag 2 | Found |
| Flag 3 | Found |
| Flag 4 | Found |

**Total: 4/4 Flags**

![Flag Collection](screenshots/flags.png)

---

## 🗄️ Phase 5 – WordPress Database Investigation

### ⚙️ Step 9 – WordPress Configuration Investigation

I continued investigating the WordPress installation and its configuration files.

The WordPress configuration file is:

```text
/var/www/html/wordpress/wp-config.php
```

To navigate to the directory:

```bash
cd /var/www/html/wordpress
```

Inspect the configuration file:

```bash
cat wp-config.php
```

This file contains important WordPress configuration settings, including database connection information.

**Security Observation:**

Exposed configuration files can reveal sensitive information, including database credentials.

![WordPress Configuration](screenshots/wp-config.png)

### 🛢️ Step 10 – MySQL Database Investigation

After identifying the database configuration information, I investigated the WordPress database.

**Command:**

```bash
mysql -u root -p
```

**Command Explanation**

- `mysql` – MySQL command-line client.
- `-u root` – Specifies the database username.
- `-p` – Prompts for the database password.

After successfully connecting, I selected the WordPress database.

```sql
SHOW DATABASES;
```

Select the database:

```sql
USE wordpress;
```

List the available tables:

```sql
SHOW
