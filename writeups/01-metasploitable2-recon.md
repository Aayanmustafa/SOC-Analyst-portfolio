# Metasploitable2 — Initial Reconnaissance

Ran an nmap scan against the Metasploitable2 target (`nmap 192.168.56.101`)
from my Kali Linux attack machine, both on an isolated host-only VirtualBox
network.

The scan found 21 open TCP ports, including several outdated and high-risk
services — FTP (21), Telnet (23), SMB (445), and exposed database ports
(MySQL 3306, PostgreSQL 5432) — indicating a deliberately vulnerable attack
surface.

**Next step:** enumerate service versions with `nmap -sV` and investigate
the FTP service for known vulnerabilities.
