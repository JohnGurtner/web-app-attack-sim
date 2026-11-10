# Web App & Network Attack Simulation

> Deployed a deliberately vulnerable target (Metasploitable 2 with DVWA), mapped it with Nmap, and
> exploited it at both the web and service layers: SQL injection, reflected XSS, session hijacking, a
> backdoored FTP server and a weak VNC login.

*CNIT 271 (Cybersecurity Fundamentals II), Lab 5. Authorized lab target on an isolated network only.*

## Lab setup

| Machine | Role | Address |
| --- | --- | --- |
| Kali Linux | Attacker | lab subnet |
| Metasploitable 2 (with DVWA) | Target | 44.67.37.50 |

Tools: Nmap 7.95, Metasploit Framework, a web browser and its developer tools.

## Procedure

### 1. Deploy the target
1. Deployed Metasploitable 2 from the cluster's VM template.
2. **Deleted the template's two preconfigured NICs** and added one on the group subnet with a static IP.
   That kept the vulnerable box on the lab network only.
3. Set VM ownership and checked connectivity by pinging out and to the Kali VM.

### 2. Reconnaissance
```bash
nmap --version          # 7.95
nmap -sV -p- 44.67.37.50
```
`-p-` scans **all 65,535 TCP ports**, and `-sV` **identifies the service and version** on each one. The
scan showed a very large attack surface: a long list of open ports and services, including the web
server hosting DVWA, FTP (vsftpd) and VNC. That list drove everything after this step.

### 3. SQL injection (DVWA)
Set DVWA security to **Low**, then on the *SQL Injection* page:

| Payload | What it does |
| --- | --- |
| `' or '1'='1` | Closes the string and adds an always-true condition, so **every row** comes back |
| `%' or 0=0 union select null, user() #` | Always-true condition, then `UNION` adds a second query asking **which DB user** runs the app. `#` comments out the rest |
| `' union select null, table_name from information_schema.tables where table_name like 'user%' #` | Uses the database's own catalog to **find tables named `user…`**, which is the step before dumping credentials |

Raised security to **Medium** and tried again. At this level DVWA escapes quote characters, which is aimed
at the string-based payloads above. Then I tried a numeric payload that needs no quotes, `1 OR 1=1`. The
lesson: **escaping is not the same as parameterization.**

**Countermeasures:** parameterized queries / prepared statements, ORM frameworks that separate input from
SQL, allow-list input validation, and a least-privilege DB account.

### 4. Reflected XSS and session hijacking
1. Entered a unique string (`VulnerabletoXSS`) and found it **reflected unencoded** in the page source.
2. Ran `<script>alert("Vulnerable to XSS attack.")</script>`. The script executed, confirming reflected XSS.
3. Ran `<script>alert(document.cookie)</script>`. It showed the **session cookie** (`PHPSESSID`).
4. **Hijacked the session:** in a separate private browser window, replaced that window's session ID
   with the stolen one and refreshed. I was logged in as the victim **with no username or password**.

**Countermeasures:** output encoding, Content-Security-Policy, `HttpOnly` and `Secure` cookies (so
JavaScript can't read them), and rotating session IDs on login.

### 5. Exploiting a backdoored service: vsftpd 2.3.4
```
msfconsole
search vsftpd
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 44.67.37.50
run
```
That gave a **root shell**. Confirmed with `whoami` and `uname -a`, then read `/etc/shadow`, which holds
the password hashes for offline cracking. This vsftpd release shipped with a **malicious backdoor** in its
source code. Patching and software supply-chain checks are the defense.

### 6. Weak remote-desktop credentials: VNC
```
use auxiliary/scanner/vnc/vnc_login
set RHOSTS 44.67.37.50
run
```
The scanner found **valid weak VNC credentials**. Connecting with a VNC viewer gave full interactive
desktop access, and `/etc/shadow` was readable again.

## Findings summary
| Finding | Impact | Fix |
| --- | --- | --- |
| SQL injection | Read the whole database, enumerate schema | Prepared statements, least-privilege DB user |
| Reflected XSS | Runs attacker scripts in the victim's browser | Output encoding, CSP |
| Session token exposure | Account takeover with no credentials | `HttpOnly`/`Secure` cookies, session rotation |
| vsftpd 2.3.4 backdoor | Unauthenticated **root** | Patch or remove, verify software integrity |
| Weak VNC password | Full remote desktop | Strong creds, tunnel over SSH/VPN, or disable |
| Huge exposed surface | Every service is a way in | Minimize services, host firewall |

## Skills demonstrated

Network recon (Nmap) · SQL injection (union-based, schema enumeration) · reflected XSS · session hijacking ·
Metasploit Framework (exploit and auxiliary modules) · service exploitation · countermeasure design

## Screenshots

See [`screenshots/`](screenshots/) for the Nmap results, each injection's output, the cookie theft and
session reuse, and the vsftpd and VNC sessions.
