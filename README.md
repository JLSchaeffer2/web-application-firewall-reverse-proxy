# Web Application Firewall Implementation (ModSecurity + Reverse Proxy)

## Overview
This project documents the implementation of a Web Application Firewall (WAF) using ModSecurity and the OWASP Core Rule Set (CRS), deployed behind an Apache reverse proxy to protect a vulnerable WordPress web application.

The goal was to move from a basic Layer 4 firewall model to a layered security architecture capable of inspecting, detecting, and actively blocking malicious web traffic at the application layer.

---

## Objective
To secure a public-facing web application by introducing Layer 7 traffic inspection, enforcing HTTPS, and validating that common web attacks (e.g., XSS) can be detected and blocked in real time.

---

## Environment Architecture

### Original State (Vulnerable)
- pfSense forwarded HTTP traffic directly to WordPress server
- No Layer 7 inspection
- No WAF protection
- No HTTPS enforcement
- Web server exposed to web-based attacks

### Remediated State (Secure)
- pfSense routes traffic to Apache reverse proxy (10.20.30.220)
- Reverse proxy forwards requests to backend WordPress server (10.20.30.130)
- ModSecurity + OWASP CRS inspects traffic
- HTTPS enforced via SSL offloading
- Malicious requests detected and blocked

---

## Key Components

- pfSense (Firewall / NAT)
- Apache (Reverse Proxy)
- ModSecurity (WAF Engine)
- OWASP Core Rule Set (CRS)
- WordPress (Backend Application)
- Graylog (SIEM monitoring)
- Wireshark / PCAP (traffic validation)

---

## Implementation Details

### 1. Reverse Proxy Deployment
- Deployed Apache reverse proxy on LAN (10.20.30.220)
- Updated pfSense NAT rule:
  - Redirect target changed from WordPress server → reverse proxy
- Verified traffic flow using packet capture (PCAP)

### 2. Traffic Flow Validation
- Confirmed:
  - WAN → Reverse Proxy → Backend Server
- Verified using pfSense packet captures and Wireshark analysis

---

### 3. Web Application Firewall (WAF)

#### Detection Mode
- Enabled ModSecurity with OWASP CRS
- Configured:
  - SecRuleEngine DetectionOnly
- Monitored alerts using Graylog
- Triggered alerts using XSS test payload

#### Prevention Mode
- Updated configuration:
  - SecRuleEngine On
- Restarted Apache
- Re-tested XSS payload

✅ Result:
- Malicious payload blocked
- HTTP 403 response returned
- Legitimate traffic unaffected

---

### 4. SSL Offloading & HTTPS Enforcement

- Created HTTPS VirtualHost:
  - /etc/apache2/sites-available/000-default-ssl.conf
- Configured:
  - SSLEngine On
  - Certificate and key paths
- Implemented redirect:
  - HTTP → HTTPS (301 redirect)

- Updated pfSense:
  - Added port 443 forwarding to reverse proxy

✅ Result:
- All external traffic encrypted
- Internal proxy-to-server communication remains HTTP

---

## Evidence of Remediation

### Traffic Flow (PCAP)
- Before:
  - Direct WAN → WordPress communication
- After:
  - WAN → Reverse Proxy → Backend

### WAF Blocking Behavior
- Detection mode:
  - XSS payload successfully reached backend
- Prevention mode:
  - Payload blocked
  - HTTP 403 response observed

### SIEM Visibility
- ModSecurity alerts observed in Graylog
- Attack patterns detected and logged

---

## Security Improvements

- Introduced Layer 7 inspection
- Prevented XSS and injection attacks
- Enforced encrypted communication (HTTPS)
- Reduced direct exposure of backend server
- Improved monitoring and detection capabilities

---

## Skills Demonstrated

- Network Security & Traffic Flow Analysis  
- Reverse Proxy Configuration (Apache)  
- Web Application Firewall Implementation  
- ModSecurity & OWASP CRS Integration  
- TLS / HTTPS Configuration  
- SIEM Monitoring (Graylog)  
- Attack Simulation & Validation  

---

## Project Files

- [MegaQuagga Remediation Report](./%5BSprint%2010%5D%20MegaQuagga%20Remediation%20Report%20%7BJerome%20Schaeffer%7D.docx)
- [PCAP Worksheet](./%5BSprint%2010%5D%20PCAP%20worksheet%20%7BJerome%20Schaeffer%7D.xlsx)
- pfSense PCAP files (WAN and LAN captures)
- Red Team traffic capture (pcapng)

---

## Notes
All testing was conducted in a controlled lab environment. No production systems were impacted.
