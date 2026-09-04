# Wireshark Analysis — vsftpd Backdoor Exploit (Defender's View)

## Summary
Captured live network traffic during the vsftpd 2.3.4 backdoor exploit
(see writeup #2) using Wireshark, to analyze the attack from a defensive
/ SOC analyst perspective rather than an attacker's.

## Capture Setup
- Tool: Wireshark, capturing on the `eth0` interface (host-only network
  between Kali and Metasploitable2)
- Re-ran the exploit from a clean VM snapshot while the capture was active,
  to record the full attack from the very first packet

## Key Findings

### 1. Service version disclosure
The FTP server's banner immediately announces its exact software version: 220 (vsFTPd 2.3.4)

This kind of banner-grabbing is one of the first things an attacker (or a
defender auditing their own network) checks — a specific version number
makes it trivial to look up known vulnerabilities for that exact release.

### 2. The malicious login trigger
Following the TCP stream on the FTP conversation revealed the exact
attack signature:
USER j:)
331 Please specify the password.
PASS pDaQA

A username containing a smiley face (`:)`) is not something a legitimate
FTP client would ever generate — this is the literal trigger string for
the vsftpd 2.3.4 backdoor. From a detection standpoint, this is a clean,
reliable indicator: any FTP login attempt with a username matching this
pattern should be treated as a near-certain compromise attempt.

### 3. Backdoor shell traffic on port 6200
Filtering on `tcp.port==6200` showed a separate TCP session opening
immediately after the malicious login — this is the actual backdoor shell
Metasploit connected to. In a real network, unexpected traffic on port
6200 immediately following an FTP session would be a strong anomaly:
it's not a standard service port, and its appearance correlated directly
with the suspicious FTP login is a clear indicator of compromise.

## Key takeaway
This exercise showed the same attack from both sides. On the offensive
side (writeup #2), the exploit looked like a single command. On the
network, it left a clear, identifiable trail: a suspicious username string
and an unexpected port opening right after. This is exactly the kind of
pattern-matching a SOC analyst relies on — a detection rule alerting on
FTP `USER` commands containing non-standard characters like `:)`, or on
any traffic to port 6200, would have caught this attack in real time.
