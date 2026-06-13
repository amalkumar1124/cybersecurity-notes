# ⚠️ **Disclaimer:** These notes are for educational purposes and authorized security testing only. Always have explicit written permission before running recon against any target.

# Subdomain Enumeration & Asset Discovery

## 📌 Description
Subdomain enumeration is the process of finding valid subdomains for a target domain. This is a critical initial phase in reconnaissance (Recon) because it expands the target's attack surface, exposing forgotten staging environments, internal portals, or unmaintained web applications.

## 🛠️ Passive Subdomain Gathering
Passive reconnaissance collects subdomains using third-party data aggregators without directly interacting with the target's servers. This keeps your activity completely invisible to their blue team.

# 1. Subfinder (ProjectDiscovery)
Fast, reliable, and uses multiple passive APIs.

subfinder -d target.com -o subfinder_subs.txt

# 2. Assetfinder (Tomnomnom)
A lightweight, fast utility that queries places like WayBackMachine, CRT.sh, and virus total.

assetfinder --subs-only target.com > assetfinder_subs.txt

# 3. Amass (OWASP)
An enterprise-grade asset mapping tool. It is deeper but takes more time.

amass enum -passive -d target.com -o amass_subs.txt

# 4. GAU (GetAllUrls) - Historical Data
Fetches known URLs from AlienVault's OTX, the Wayback Machine, Common Crawl, and URLScan.

gau target.com > gau_urls.txt

## ⚡ Active Subdomain Gathering (Brute-Forcing)
Active reconnaissance interacts directly with target DNS servers to guess subdomains using automated wordlists.

# 1. ShuffleDNS (ProjectDiscovery)
A fast DNS brute-forcer that uses trusted wildcards and valid resolvers to prevent false positives.

shuffledns -d target.com -list wordlist.txt -r resolvers.txt -o shuffledns_subs.txt

# 2. PureDNS
An extremely fast brute-forcing tool optimized for dealing with mass DNS queries.

puredns bruteforce wordlist.txt target.com -r resolvers.txt -o puredns_subs.txt

# 3. DNSx (ProjectDiscovery) - Resolution Check
Filters your subdomain list down to only the ones that actually resolve to an IP.

cat unique_subs.txt | dnsx -silent -o resolved_subs.txt

# 🔍 Validating Active Web Targets (Probing)
Gathering subdomains gives you raw hostnames. To see which ones are actually running operational web servers on HTTP or HTTPS, we use web probing tools.

# 1. httpx (ProjectDiscovery)
The industry standard tool to filter out dead subdomains and collect status codes, titles, and web server banners.

# A.Run a basic probe on your compiled list:

httpx -l all_subs.txt -o live_web_targets.txt

# B.Pro and Detailed Probe (Extracts Status Codes, Content-Length, and Page Titles):

httpx -l all_subs.txt -sc -cl -title -o detailed_targets.txt

# 2. Gowitness - Screenshotting Live Targets
Takes screenshots of every live target so you can visually triage hundreds of hosts quickly.

gowitness file -f live_web_targets.txt -P screenshots

## 🎯 The Ultimate Recon Pipeline (The One-Liner Script)
Instead of running these tools one by one, professionals combine them into a single Linux terminal pipeline using cat, sort, and uniq:

# 1. Gather all unique passive subdomains
   
subfinder -d target.com -silent > raw.txt; assetfinder --subs-only target.com >> raw.txt

# 2. Clean out duplicates
   
cat raw.txt | sort -u > unique_subs.txt

# 3. Find which ones are running live web servers instantly
   
httpx -l unique_subs.txt -sc -title -silent -o final_web_targets.txt

# 4. Resolve and check for live hosts in one go

cat unique_subs.txt | dnsx -silent | httpx -sc -title -silent -o final_web_targets.txt





