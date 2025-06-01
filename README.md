# Network Reconnaissance Report

## Overview
This repository contains Project 2 for CSCI-UA.0480 (Intro to Security & Cryptography, Fall 2024). The assignment performs network reconnaissance on three HTTPS domains (`netflix.com`, `eff.org`, `india.gov.in`) to compare their security configurations, including WHOIS data, DNS records, traceroute paths, open ports, and TLS/SSL certificate details.

## Tech Stack
- **CLI Tools**: `whois`, `dig`, `traceroute` (or `tracert` on Windows), `nmap`  
- **Browser Inspection**: Certificate viewer, Certificate Transparency lookup (crt.sh)  
- **Online Service**: Qualys SSL Labs (ssltest)

## Reconnaissance Steps
1. **WHOIS Lookup**  
   Retrieved registrar, creation/expiration dates, and contact details.
2. **DNS Analysis**  
   Queried A, NS, CNAME, and MX records; checked for DNSSEC.
3. **Traceroute**  
   Mapped IP hops and performed geolocation of the last public node.
4. **Port Scan (Nmap)**  
   Identified open TCP ports for each domain.
5. **Certificate Inspection**  
   Examined cipher suites, key types/sizes, issuer chains, validity, and OCSP/CRL endpoints.
6. **TLS/SSL Analysis (Qualys SSL Labs)**  
   Evaluated supported protocol versions, preferred ciphers, HSTS status, and overall grade.

## Summary of Findings
- **Netflix** uses a commercial ECC certificate (DigiCert), supports TLS 1.0–1.3, and enables HSTS for one year. Legacy protocol support lowered its SSL Labs grade.
- **EFF** relies on Let’s Encrypt RSA certificates, supports only TLS 1.2, and enforces HSTS with a long max-age.
- **India.gov.in** also uses Let’s Encrypt RSA, supports TLS 1.2/1.3, and has a valid HSTS configuration but with minor header misconfigurations.
- None of the three domains implement DNSSEC.
- All expose only ports 53 (DNS), 80 (HTTP), and 443 (HTTPS).
- Traceroutes revealed differing hop counts and data-center locations, reflecting organizational infrastructure choices.

## Key Takeaways
- **CA Choice & Key Types**: Commercial sites often prefer ECC for performance; non-profit/government sites frequently use Let’s Encrypt to reduce cost.
- **Protocol Support**: Modern TLS versions (1.2/1.3) are standard among EFF and India.gov.in; supporting deprecated versions (TLS 1.0/1.1) can lower security grades.
- **DNSSEC Adoption**: Even security-focused organizations sometimes omit DNSSEC, indicating slow adoption.
- **HSTS Configuration**: Proper HSTS deployment varies—EFF’s long max-age is exemplary, while others either misconfigure or use shorter durations.

## Quick Start
All commands assume a Unix-like shell. Windows users can substitute equivalents (e.g., `tracert` for `traceroute`).

1. **Clone the repo**  
   ```bash
   git clone https://github.com/your-username/network-reconnaissance-report.git
   cd network-reconnaissance-report
