## Nmap (Network Mapper) Technical Reference Guide

# Description

Nmap is an open-source tool used for network discovery, port scanning, and vulnerability detection. This reference sheet categorizes commands based on their operational impact and purpose.

## ⚡ Essential Scanning Commands

# 1. Host Discovery (Ping Sweep)

Find live hosts on a network without scanning their ports:

  nmap -sn 192.168.1.0/24


# 2. SYN Stealth Scan (Requires root/sudo)

Scans ports quickly and stealthily by not completing the full 3-way TCP handshake:

  sudo nmap -sS <target_ip>


# 3. Service and Version Detection

Interrogates open ports to determine what software and version are running:

nmap -sV <target_ip>


# 4. Operating System Detection

Attempts to identify the host operating system:

sudo nmap -O <target_ip>


# 5. Scanning All Ports

By default, Nmap scans the top 1,000 ports. To scan all 65,535 ports:

nmap -p- <target_ip>


## 🛡️ Scan Category Reference Guide

These commands are categorized by operational impact and behavior. Choose carefully based on the assessment's rules of engagement (RoE).

## 🔵 Safe Scans

These commands will not crash services or negatively impact target hosts.

1.Simple ping scan - just checks if host is up

nmap -sn 192.168.1.1

2.Ping scan entire subnet

nmap -sn 192.168.1.0/24

3.DNS resolution only (no port scan)

nmap -sL 192.168.1.0/24

4.SYN scan on common ports (read-only, no connection completed)

nmap -sS 192.168.1.1

5.Scan with no ping (assume host is up)

nmap -Pn 192.168.1.1

6.Only show open ports quietly

nmap --open 192.168.1.1


## 🟡 Intrusive Scans

These commands are louder on the network, complete active connections, or gather extensive service details, potentially raising security alerts.

1.Full TCP connect scan (completes 3-way handshake)

nmap -sT 192.168.1.1

2.Aggressive scan (OS detect + version + scripts + traceroute)

nmap -A 192.168.1.1

3.Full port scan (all 65535 ports)

nmap -p- 192.168.1.1

4.Service + version detection

nmap -sV 192.168.1.1

5.OS fingerprinting

nmap -O 192.168.1.1

6.UDP scan (can cause service disruption due to ICMP rate limiting)

nmap -sU 192.168.1.1

7.Intrusive + version intensity max

nmap -sV --version-intensity 9 192.168.1.1


## 🔴 Vulnerability Scanning (NSE vuln)

Scans the target against known vulnerabilities. Useful for identifying high-risk exposures.

1.Run all vuln scripts

nmap --script vuln 192.168.1.1

2.Check for specific CVEs

nmap --script vuln --script-args vulns.showall 192.168.1.1

3.Heartbleed (OpenSSL) check

nmap -p 443 --script ssl-heartbleed 192.168.1.1

4.SMB vulnerabilities (EternalBlue, MS17-010)

nmap -p 445 --script smb-vuln-ms17-010 192.168.1.1

5.All SMB vuln scripts

nmap -p 445 --script smb-vuln-* 192.168.1.1

6.HTTP vulnerabilities

nmap -p 80,443 --script http-vuln-* 192.168.1.1

7.SSL/TLS weaknesses

nmap -p 443 --script ssl-enum-ciphers 192.168.1.1

6.FTP vulnerabilities

nmap -p 21 --script ftp-vuln-* 192.168.1.1

7.Shellshock

nmap -p 80 --script http-shellshock 192.168.1.1

8.Cross-site scripting (XSS) check

nmap -p 80 --script http-stored-xss 192.168.1.1

9.Slowloris DoS vulnerability check

nmap -p 80 --script http-slowloris-check 192.168.1.1


## 💀 Exploitation Scans

Attempts to exploit identified vulnerabilities. Use with extreme caution on production servers!

1.Exploit ms08-067 via nmap script

nmap -p 445 --script smb-vuln-ms08-067 --script-args unsafe=1 192.168.1.1

2.Run exploit-category NSE scripts

nmap --script exploit 192.168.1.1

3.HTTP PUT method exploit (upload shell)

nmap -p 80 --script http-method-tamper 192.168.1.1

4.Attempt RCE via shellshock

nmap -p 80 --script http-shellshock --script-args uri=/cgi-bin/test.cgi 192.168.1.1

5.Exploit unsafe SMB (use with caution!)

nmap -p 445 --script smb-vuln-* --script-args unsafe=1 192.168.1.1


## 🔑 Authentication Auditing (Bypass & Config Check)

Checks for anonymous, null, or weak default configuration settings.

1.FTP anonymous login check

nmap -p 21 --script ftp-anon 192.168.1.1

2.Check for anonymous FTP + list files

nmap -p 21 --script ftp-anon,ftp-syst 192.168.1.1

3.SMB null/anonymous session check

nmap -p 445 --script smb-security-mode 192.168.1.1

4.Check for open SMTP relay

nmap -p 25 --script smtp-open-relay 192.168.1.1

5.SNMP default community strings

nmap -sU -p 161 --script snmp-info 192.168.1.1

6.LDAP anonymous bind

nmap -p 389 --script ldap-rootdse 192.168.1.1

7.MongoDB no-auth check

nmap -p 27017 --script mongodb-info 192.168.1.1

8.MySQL anonymous login

nmap -p 3306 --script mysql-empty-password 192.168.1.1

9.VNC no-password check

nmap -p 5900 --script vnc-info 192.168.1.1

10.RDP auth check

nmap -p 3389 --script rdp-enum-encryption 192.168.1.1


## 🔓 Brute Force Auditing (NSE brute)

Performs dictionary attacks to guess login credentials. Extremely noisy.

1.SSH brute force

nmap -p 22 --script ssh-brute 192.168.1.1

2.FTP brute force

nmap -p 21 --script ftp-brute 192.168.1.1

3.HTTP Basic Auth brute force

nmap -p 80 --script http-brute 192.168.1.1

4.SMB brute force

nmap -p 445 --script smb-brute 192.168.1.1

5.MySQL brute force

nmap -p 3306 --script mysql-brute 192.168.1.1

6.PostgreSQL brute force

nmap -p 5432 --script pgsql-brute 192.168.1.1

7.Telnet brute force

nmap -p 23 --script telnet-brute 192.168.1.1

8.SNMP community string brute force

nmap -sU -p 161 --script snmp-brute 192.168.1.1

9.SMTP brute force

nmap -p 25 --script smtp-brute 192.168.1.1

10.RDP brute force

nmap -p 3389 --script rdp-enum-encryption 192.168.1.1

11.Brute force with custom wordlists

nmap -p 22 --script ssh-brute --script-args userdb=users.txt,passdb=pass.txt 192.168.1.1


## 🟢 Active Discovery & Info Gathering

Queries open services to retrieve diagnostic structures and metadata.

1.SNMP full walk (network info, interfaces, processes)

nmap -sU -p 161 --script snmp-walk 192.168.1.1

2.SNMP system info

nmap -sU -p 161 --script snmp-sysdescr 192.168.1.1

3.SNMP network interfaces

nmap -sU -p 161 --script snmp-interfaces 192.168.1.1

3.SNMP running processes

nmap -sU -p 161 --script snmp-processes 192.168.1.1

4.DNS service discovery

nmap -p 53 --script dns-service-discovery 192.168.1.1

5.DNS zone transfer

nmap -p 53 --script dns-zone-transfer --script-args dns-zone-transfer.domain=example.com 192.168.1.1

6.SMB share enumeration

nmap -p 445 --script smb-enum-shares 192.168.1.1

7.SMB users enumeration

nmap -p 445 --script smb-enum-users 192.168.1.1

8.NetBIOS info

nmap -p 137 --script nbstat 192.168.1.1

9.HTTP headers, methods, and page title gathering

nmap -p 80 --script http-headers,http-methods,http-title 192.168.1.1

10.LDAP info

nmap -p 389 --script ldap-search 192.168.1.1

11.NFS shares showmount

nmap -p 111 --script nfs-showmount 192.168.1.1

12.RPC service info

nmap -p 111 --script rpcinfo 192.168.1.1

13.MySQL diagnostic metadata

nmap -p 3306 --script mysql-info,mysql-databases 192.168.1.1

14.Run all discovery-class scripts at once

nmap --script discovery 192.168.1.1


## ⚡ ADVANCED: Firewall & IDS Evasion (Missing Pro Commands)

Techniques used to sneak past Intrusion Detection Systems (IDS) and firewalls during active pentests.

1.Fragment packets (splits TCP headers over several packets to bypass simple firewalls)

nmap -f 192.168.1.1

2.Specify a custom MTU (Maximum Transmission Unit) - must be a multiple of 8

nmap --mtu 24 192.168.1.1

2.Spoof Source Port (forces scan to come from a common trusted port, e.g., DNS port 53)

nmap -g 53 192.168.1.1

3.Spoof Source IP (makes the target think the scan is coming from a different address)

nmap -S 10.0.0.5 -e eth0 192.168.1.1

4.Decoy Scan (mixes your IP with random spoofed decoy IPs so defenders can't tell who started it)

nmap -D RND:10 192.168.1.1

5.Append random data to sent packets (pads payload size to look like normal traffic)

nmap --data-length 25 192.168.1.1


## 📊 ADVANCED: Smart Port Selection

Ways to quickly filter specific protocol variations.

1.Fast scan (only scans the top 100 most common ports instead of 1,000)

nmap -F 192.168.1.1

2.Scan both TCP and UDP ports at the same time in one command

nmap -p T:21-25,80,U:53,111 192.168.1.1


## 💾 ADVANCED: Output Formats & Reporting

How to save your scanning results so other security tools can parse them.

1.Normal Text Output (exactly what you see on the screen)

nmap -oN scan.txt 192.168.1.1

2.Grepable Output (easy to search with command-line tools like grep/awk/cut)

nmap -oG scan.gnmap 192.168.1.1

3.XML Output (Required if you want to import results into Metasploit, Zenmap, or Dradis)

nmap -oX scan.xml 192.168.1.1

4.Save in ALL THREE major formats at the same time (Highly Recommended)

nmap -oA initial_recon 192.168.1.1


## ⚡ Quick Reference — Most Used Commands

1.Standard recon scan (labs/CTFs)

sudo nmap -sS -sV -O -T4 -oA scan_results <target_ip>

2.Fast top 100 ports

nmap -F -T4 <target_ip>

3.Full port scan with version detection

sudo nmap -p- -sV -T4 -oN full_scan.txt <target_ip>

4.Vulnerability scan

nmap --script vuln -oN vuln_scan.txt <target_ip>


## ⏱️ Tuning & Timing Template Speeds

1. -T controls how fast Nmap scans. Scale is 0 (slowest) to 5 (fastest).

2. -T0 (Paranoid) / -T1 (Sneaky): Used for bypassing IDS detection. Extremely slow.

3. -T2 (Polite): Slows down the scan to consume less bandwidth and avoid crashing the target.

4. -T3 (Normal): The default behavior if you don't specify anything.

5. -T4 (Aggressive): Speeds up the scan. Highly recommended for labs, CTFs, and fast network environments.

6. -T5 (Insane): Extremely fast. Can miss ports due to timeout drops, or trigger network bottlenecks.





