# network-reconnaissance-report

## Overview
In this assignment, I performed a network reconnaissance analysis of three HTTPS domains—`netflix.com`, `eff.org`, and `india.gov.in`—and documented their security configurations. The primary goals were to:

- Retrieve and interpret WHOIS registration data  
- Conduct DNS resolution and record inspection  
- Map the IP routing path via traceroute  
- Scan for open TCP ports with Nmap  
- Examine HTTPS certificates (cipher suites, key sizes, validity, certificate chains)  
- Analyze TLS/SSL support using Qualys SSL Labs  

Throughout the process, I compared the security posture of a large commercial site (Netflix), a nonprofit organization (EFF), and a government domain (India.gov.in) to highlight differences in certificate usage, protocol support, and overall best practices. 

## Tech Stack
All reconnaissance steps were performed using standard command-line tools and web‐based services:

- **WHOIS** (built-in on Linux/macOS or via `whois` package)  
- **DNS utilities**  
  - `dig` (with `+trace` and `ANY` queries)  
  - `nslookup` (optional)  
- **IP geolocation**  
  - Online service: iplocation.net (browser‐based lookup)  
- **Traceroute**  
  - `traceroute` (Linux/macOS) or `tracert` (Windows)  
- **Port scanning**  
  - `nmap` (network mapper)  
- **Browser Developer Tools**  
  - HTTPS certificate inspection (Chrome/Firefox certificate viewer)  
- **Certificate Transparency lookup**  
  - crt.sh (web interface)  
- **TLS/SSL analysis**  
  - Qualys SSL Labs SSL Test (ssltest/ssltest)  

No programming languages or frameworks were needed—this project focuses on using existing security utilities and interpreting their outputs. :contentReference[oaicite:1]{index=1}

## Features
Each of the three domains was evaluated according to the following reconnaissance steps:

1. **Domain Registration (WHOIS)**  
   - Retrieved registrant name, registration privacy status, contact details, creation date, and expiration date.  
   - Example findings:  
     - `netflix.com` is openly registered to Netflix, Inc. (created 11 Nov 1997, expires 10 Nov 2025).  
     - `eff.org` uses privacy protection (registered 10 Oct 1990, expires 9 Oct 2025).  
     - `india.gov.in` is registered to the National Informatics Centre (created 26 Sep 2005, expires 26 Sep 2025). :contentReference[oaicite:2]{index=2}

2. **DNS Analysis**  
   - Performed `dig +trace <domain> ANY` to enumerate A, NS, CNAME, and MX records.  
   - Checked DNSSEC support.  
   - Example findings:  
     - All three domains returned NS and CNAME records; only `eff.org` and `india.gov.in` had explicit MX records pointing to Microsoft Outlook and `mailgw.nic.in` respectively.  
     - None of the domains supported DNSSEC. :contentReference[oaicite:3]{index=3}

3. **IP Routing (Traceroute)**  
   - Ran `traceroute <domain>` to count total hops and map intermediate nodes.  
   - Determined the number of unidentified (private/data-center) hops.  
   - Performed an IP geolocation lookup on the last identifiable router.  
   - Example findings:  
     - `netflix.com` path: 64 hops (57 internal hops); last public hop located in Columbus, OH (IP 108.166.248.8).  
     - `eff.org` path: 64 hops (60 internal hops); last public hop located in Miami, FL (IP 140.222.19.111).  
     - `india.gov.in` path: 9 hops (5 internal hops); last public hop located in Secaucus, NJ (IP 23.57.90.103). :contentReference[oaicite:4]{index=4}

4. **TCP Port Scanning (Nmap)**  
   - Ran `nmap <domain>` to detect open TCP ports.  
   - All three domains consistently exposed ports 53 (DNS), 80 (HTTP), and 443 (HTTPS); no other externally visible ports were open. :contentReference[oaicite:5]{index=5}

5. **PKI / HTTPS Certificate Inspection**  
   - Used browser certificate viewers to determine negotiated cipher suite, public key type/size, certificate issuer (root CA), validity period, and presence of OCSP/CRL endpoints.  
   - Queried the Certificate Transparency logs (crt.sh) to see how many CT logs include the certificate and its first log date.  
   - Example findings:  
     - **Netflix (netflix.com)**  
       - Cipher Suite: `TLS_AES_256_GCM_SHA384`  
       - Leaf Key: ECC P-256 (256 bits)  
       - Expires: 24 Sep 2025 (≈366 days validity)  
       - Root CA: DigiCert Secure Site ECC CA–1 (RSA 2048 bits, expires 9 Nov 2031)  
       - Shared domains: `account.netflix.com`, `www.netflix.com`, `tv.netflix.com`, etc.  
       - Intermediate certificates in chain: 1  
       - OCSP responder: `http://ocsp.digicert.com`  
       - CRL distribution points: two DigiCert URIs  
       - CT logs: 10 entries, first logged 3 Oct 2024 :contentReference[oaicite:6]{index=6}  
     - **EFF (eff.org)**  
       - Cipher Suite: `ECDHE-RSA-CHACHA20-POLY1305`  
       - Leaf Key: RSA 2048 bits  
       - Expires: 28 Jan 2025 (≈89 days validity as of analysis)  
       - Root CA: Let’s Encrypt R11 (RSA 4096 bits, expires 4 Jun 2035)  
       - Shared domains: `*.eff.org`, `*.staging.eff.org`  
       - Chain length: 1 intermediate  
       - OCSP responder: `http://r11.o.lencr.org`  
       - CT logs: 2 entries, first logged 30 Oct 2024 :contentReference[oaicite:7]{index=7}  
     - **India.gov.in**  
       - Cipher Suite: `TLS_AES_256_GCM_SHA384`  
       - Leaf Key: RSA 2048 bits  
       - Expires: 16 Dec 2024 (≈89 days validity)  
       - Root CA: Let’s Encrypt R11 (RSA 4096 bits, expires 4 Jun 2035)  
       - Shared domains: `www.india.gov.in`, punycode variant for Hindi subdomain  
       - Chain length: 1 intermediate  
       - OCSP responder: `http://r11.o.lencr.org`  
       - CT logs: 7 entries, first logged 17 Sep 2024 :contentReference[oaicite:8]{index=8}

6. **TLS/SSL Analysis (Qualys SSL Labs)**  
   - Submitted each domain to the SSL Labs test to identify supported protocol versions, preferred cipher suites, handshake compatibility, and HSTS settings.  
   - Example findings:  
     - **Netflix**  
       - Supported TLS: 1.0, 1.1, 1.2, 1.3  
       - Most-preferred ciphers: `TLS_AES_128_GCM_SHA256` (TLS 1.3), `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256` (TLS 1.2) – both provide forward secrecy.  
       - Handshake failures: Certain legacy clients (hence SSL Labs grade B).  
       - HSTS: Enabled, `max-age=31536000`.  
       - Grade B due to outdated TLS 1.0/1.1 support. :contentReference[oaicite:9]{index=9}  
     - **EFF**  
       - Supported TLS: 1.2 only  
       - Preferred ciphers: `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384` – forward-secrecy capable.  
       - Handshake: No failures (grade A+).  
       - HSTS: Enabled, `max-age=63072000`. :contentReference[oaicite:10]{index=10}  
     - **India.gov.in**  
       - Supported TLS: 1.2, 1.3  
       - Preferred ciphers: `TLS_AES_256_GCM_SHA384` (TLS 1.3), `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384` (TLS 1.2) – forward secrecy enabled.  
       - Handshake: No failures (grade A).  
       - HSTS: Misconfigured (multiple HSTS headers), but present. :contentReference[oaicite:11]{index=11}

## Key Takeaways / Lessons Learned
- **Certificate Authorities & Key Types**  
  - Large commercial sites like Netflix often use high-assurance ECC certificates (DigiCert ECC CA) for performance benefits.  
  - Nonprofits and government domains generally rely on Let’s Encrypt (RSA 4096 bits for root) to minimize cost. :contentReference[oaicite:12]{index=12}  

- **DNSSEC Adoption**  
  - Surprisingly, none of the three domains implemented DNSSEC—even `eff.org`, an organization that advocates for security and privacy.  
  - This highlights that DNSSEC adoption remains lagging, even among security-focused entities. :contentReference[oaicite:13]{index=13}  

- **TLS/SSL Versions & Deprecation**  
  - Netflix still supports deprecated TLS 1.0/1.1, which resulted in a lower SSL Labs grade.  
  - Both EFF and India.gov.in only support modern protocols (TLS 1.2 and TLS 1.3), demonstrating better adherence to current best practices. :contentReference[oaicite:14]{index=14}  

- **HSTS Configuration**  
  - EFF’s HSTS configuration is robust (`max-age=2 years`).  
  - India.gov.in’s multiple HSTS headers indicate misconfiguration despite supporting HSTS.  
  - Netflix correctly applies HSTS for one year (`max-age=31536000`) but still lags due to legacy TLS support. :contentReference[oaicite:15]{index=15}  

- **Geographic Routing Differences**  
  - Netflix and EFF traffic terminated in U.S. data centers (Columbus, OH and Newark, NJ respectively), while India.gov.in’s routing ended in Secaucus, NJ.  
  - Government-run domains often rely on centralized hosting outside their home country for certificate provisioning or load balancing. :contentReference[oaicite:16]{index=16}  

Overall, this exercise demonstrated how organizational priorities (cost, audience, scale) influence security decisions—ranging from CA choice to protocol support—and highlighted technical gaps (e.g., lack of DNSSEC) that even security-minded entities sometimes overlook. :contentReference[oaicite:17]{index=17}

## Quick Start
To reproduce the reconnaissance steps on your own machine, follow the instructions below. All commands assume a Unix-like environment (Linux/macOS). Windows users can substitute `tracert` for `traceroute` and install necessary utilities via Cygwin or WSL.

1. **Clone or Download This Repo**  
   ```bash
   git clone https://github.com/your-username/network-reconnaissance-report.git
   cd network-reconnaissance-report
