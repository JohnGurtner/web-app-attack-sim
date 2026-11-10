# Web App &amp; Network Attack Simulation

> Scanned, enumerated, and exploited a deliberately vulnerable target (Metasploitable 2 / DVWA)
> across the web and service layers — SQL injection, XSS, session hijacking, and Metasploit modules.

*CNIT 271 — Cybersecurity Fundamentals II, Lab 5. Authorized lab target only.*

## Summary

Using Kali against a Metasploitable 2 VM, I mapped the attack surface with Nmap, then walked through
a set of classic web and service exploits in DVWA and the Metasploit Framework — raising the DVWA
security level to see how each defense changed the outcome.

## Lab setup

- Kali Linux VM + Metasploitable 2 VM (template deploy) on Purdue's Nutanix cluster
- DVWA hosted on the Metasploitable target
- Tools: `nmap` 7.95, Metasploit Framework, a browser, Kali CLI

## What I found

| Technique | What I did | Takeaway |
| --- | --- | --- |
| Recon | `nmap -sV -p-` full TCP + version scan | Enumerated open ports/services to prioritize targets |
| SQL injection | `' OR '1'='1`, `UNION SELECT ... user()`, schema enumeration | Raising DVWA to *medium* broke the naive payloads — input handling matters |
| Reflected XSS | `<script>alert(...)</script>`, then `document.cookie` | Proved script injection and cookie disclosure |
| Session hijacking | Reused a captured session cookie in a second browser | Took over a session with no username/password |
| VSFTPD backdoor | Metasploit module → shell, read `/etc/shadow` | Showed impact of a known-vulnerable service |
| VNC weak creds | `vnc_login` module → remote desktop access | Weak remote-access creds = full interactive access |

**Countermeasures I documented:** parameterized queries / ORM and input validation for SQLi;
output encoding and CSP for XSS; secure, `HttpOnly` session cookies and rotation for hijacking;
patching and removing unneeded services for the Metasploit findings.

## Screenshots

See [`screenshots/`](screenshots/) for the Nmap results, each injection's output, the cookie theft,
and the Metasploit sessions.

## Skills demonstrated

Network recon (Nmap) · SQL injection · XSS · session hijacking · Metasploit Framework ·
service exploitation · writing up countermeasures
