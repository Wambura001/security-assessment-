## Small-Business Network Security Assessment Report
## Course Project: Ethical Hacking (Project 1 / Task 1)
------------------------------
## Project Overview
This repository contains the comprehensive Security Assessment Report and testing methodology conducted on a simulated small-business infrastructure (approximately 30 employees). The primary objective of this engagement was to map the corporate attack surface, execute controlled host discovery, analyze service exposures, and identify critical vulnerabilities to improve the organization's defensive posture.
## Repository Contents

*  Network_Asset_Inventory.xlsx - Fully mapped target spreadsheet tracking asset IDs, IP addresses, OS definitions, network placements, and open ports.
*  Network_Topology_Diagram.png - Visual architecture architecture showing the isolated Demilitarized Zone (DMZ) and Internal LAN segments.
*  README.md - Technical framework, testing commands, and project methodology overview.

------------------------------
##  Phase 1: Attack Surface Discovery Methodology
The assessment was executed in sequential phases starting from passive observation to active probe identification using standard tooling.
## 1. Active Host Discovery
To map out live systems on the localized /24 subnet without triggering heavy defensive alerts:

nmap -sn 192.168.1.0/24

## 2. Comprehensive Port Scanning & Service Profiling
Once live targets were cataloged, an exhaustive scan across all 65,535 logical ports was executed to detect non-standard or hidden listening endpoints:

nmap -sV -sC -O -p- <Target_IP_or_Range>


* -sV: Service version detection
* -sC: Default vulnerability script enumeration
* -O: Operating system fingerprinting

## 3. Manual Verification (Eliminating False Positives)
Automated scanners can produce misleading indicators. Manual service banner-grabbing was performed to confirm vulnerable software versions.

* HTTP/HTTPS Header Probe:

curl -I http://192.168.2.20
curl -Ik https://192.168.2.20

* Raw Connection Protocol Verification (FTP/Telnet/SSH):

nc -vn 192.168.1.30 23
nc -vn 192.168.2.20 21


------------------------------
## Phase 2: Vulnerability Analysis & Risk Matrix
Risk scoring follows the standard quantitative industry formula:
$$\text{Risk Score} = \text{Likelihood (1-5)} \times \text{Impact (1-5)}$$ 

| Finding ID | Vulnerability / Issue | Likelihood | Impact | Score | Risk Rating | Business Context |
|---|---|---|---|---|---|---|
| VULN-01 | Anonymous FTP Login Allowed | 5 | 4 | 20 | Critical | Publicly accessible via DMZ server. Exposes critical web directories to write access. |
| VULN-02 | Cleartext Telnet Management | 3 | 5 | 15 | High | Internal network positioning. Allows sniffing of infrastructure root admin credentials. |
| VULN-03 | Legacy SMBv1 Active | 3 | 5 | 15 | High | Enabled on Domain Controller. Vulnerable to remote code execution and lateral ransomware spread. |
| VULN-04 | Exposed Unauthenticated RDP | 4 | 3 | 12 | Medium | Exposed IT Admin PC. High exposure to brute-force credential stuffing. |
| VULN-05 | HTTP Directory Listing Enabled | 4 | 2 | 8 | Low | Public Web Server. Leaks backend file structure paths, simplifying exploit orchestration. |

------------------------------
##  Phase 3: Remediation & Retesting Standard Operating Procedures## VULN-01: Anonymous FTP Remediation

   1. Modify the configuration file (/etc/vsftpd.conf):
   
   anonymous_enable=NO
   
   2. Restart the daemon to drop unauthorized access:
   
   sudo systemctl restart vsftpd
   
   3. Retesting Command: Ensure the script returns no valid login paths.
   
   nmap --script ftp-anon -p 21 192.168.2.20
   
   
## VULN-02: Cleartext Telnet Remediation

   1. Unload and disable the legacy cleartext listening service:
   
   sudo systemctl stop inetd && sudo systemctl disable inetd
   
   2. Implement encrypted SSH binaries as the mandatory remote configuration interface.
   3. Retesting Command: Confirm socket rejection response.
   
   nc -vn 192.168.1.30 23
   
   
## VULN-03: Legacy SMBv1 Deactivation

   1. Open an Administrative PowerShell prompt on the Domain Controller.
   2. Execute the system feature disabling flag:
   
   Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
   
   3. Restart the server host to complete configuration.
   4. Retesting Command: Verify only modern, cryptographically sound dialects (SMBv2/v3) exist.
   
   nmap -p 445 --script smb-protocols 192.168.1.10
   
   
------------------------------
## Grading Criteria Verification Checklist

* Create a network asset inventory spreadsheet (Done)
* Outline structural annotated topology parameters (Done)
* Map out and identify ports, protocols, and exposed vectors (Done)
* Differentiate between expected profiles vs. unexpected attack surfaces (Done)
* Isolate and manually validate automated scanner findings (Done)
* Document 5 solid findings evaluated through likelihood, impact, and business context (Done)
* Deliver specific remediation steps and diagnostic verification scans (Done)



