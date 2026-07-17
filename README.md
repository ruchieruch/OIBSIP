# OIBSIP
Oasis Infobyte SIP task submissions — Cyber Security track
# Cyber Security Track — Task 1: Basic Network Scanning with Nmap

## What is Nmap?

Nmap (Network Mapper) is a free, open-source tool used to discover hosts and
services on a network by sending packets and analyzing the responses. It's one
of the most widely used tools in both offensive security (reconnaissance) and
defensive security (auditing your own network exposure).

## Why network scanning matters

Before attackers exploit a system, they first map out what's running on it —
open ports, active services, and the operating system. Security professionals
run the same scans defensively, to find out what's exposed *before* an
attacker does, and to verify that only intended services are reachable.

## Ethics Note

⚠️ This scan was performed only against `127.0.0.1` (localhost) / a machine I
own and control. Scanning systems you do not own or have explicit written
permission to test is illegal in most jurisdictions and against the terms of
this task.

## Installation

Installed Nmap on **Windows 11** by downloading the installer from
https://nmap.org/download.html and running it with default options.

Verified installation by running a scan directly from Command Prompt
(`C:\Users\91885>`) — Nmap 7.99 was detected and ran successfully, confirming
a working installation.

## Scans Performed

Target: `127.0.0.1` (localhost — my own machine, Windows 11)

### 1. Basic scan
```
nmap 127.0.0.1
```
Result: 3 open ports found — 135/tcp, 445/tcp, 3306/tcp (997 ports closed/reset).

### 2. Service version scan
```
nmap -sV 127.0.0.1
```
Result: Identified exact service versions — Microsoft Windows RPC on 135,
microsoft-ds on 445, and MySQL 5.0.41-community-nt on 3306.

### 3. OS detection scan
```
nmap -O 127.0.0.1
```
Result: Correctly fingerprinted the OS as Microsoft Windows 11 (24H2–25H2)
based on TCP/IP stack behavior, with 0 hops network distance (confirming
local scan).

Full raw output for all three scans is saved in [`nmap_scan_results.txt`](./nmap_scan_results.txt).

## Findings Summary

| Port | Service | Description | Security Risk |
|------|---------|-------------|----------------|
| 135/tcp | msrpc (Microsoft Windows RPC) | Windows' RPC endpoint mapper, used for inter-process and inter-service communication on Windows systems | **Medium–High** — historically a major attack vector (e.g., the Blaster worm exploited MS03-026 via this port). Should never be exposed beyond localhost/trusted LAN |
| 445/tcp | microsoft-ds (SMB) | Used for Windows file and printer sharing over the SMB protocol | **High** — this is the exact port exploited by EternalBlue/WannaCry in 2017, one of the most damaging ransomware outbreaks in history. Must never be exposed to the internet |
| 3306/tcp | MySQL 5.0.41-community-nt | A MySQL database server running locally | **High** — if this port were ever exposed beyond localhost, it could allow direct database connection attempts or brute-forcing. Also notable: MySQL 5.0.41 is a very outdated version (~2007), meaning it likely carries unpatched CVEs and should be upgraded even for local development use |

**Overall assessment:** All three open ports are currently only reachable via
localhost, which is safe. However, all three represent services that have
been historically exploited when exposed externally without a firewall —
underscoring why default-deny firewall rules (see Task 2: UFW Configuration)
matter even on a personal machine.

## Screenshots

See `/screenshots` folder — includes a terminal capture showing all three
scans (basic, `-sV`, `-O`) run sequentially against `127.0.0.1`.

## Conclusion

This exercise showed that even a personal Windows machine has a small but
real attack surface — three listening services (RPC, SMB, and a MySQL
instance) exist by default/installation, two of which (135 and 445) have
well-documented histories of exploitation when exposed to untrusted
networks. My main takeaway is that "closed by default, opened only when
needed" is the right posture: none of these ports need to be reachable from
outside my own machine, so the priority recommendation is to ensure Windows
Firewall (or a tool like UFW on Linux) blocks inbound connections to them
from any network profile other than trusted/private, and to update the
MySQL installation, which is running a version from 2007.
