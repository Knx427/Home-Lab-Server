# 🛡️ Hardened Linux Server Infrastructure & Secure Remote Access

## Project Architecture
Deployed a dedicated, self-hosted Linux server architecture leveraging repurposed hardware running **Ubuntu Server 22.04 LTS (Minimal Infrastructure)**. The core objective of this project was to establish a resilient web service hosting environment while engineering around real-world Layer 3 networking limitations (CGNAT) and automated perimeter threats.

## 📁 Technical Documentation Directory
*   **[Network Engineering (Netplan)](netplan-config.md):** Configuration files for dynamic interface routing, persistent static IP addressing, and secondary wireless interface backhaul failovers.
*   **[Host Hardening & SSH Security](ssh-setup.md):** Mitigation of unauthorized remote access vectors through enforced cryptographic Key-Based Authentication, disabled root logins, custom port mapping, and system-level alerts.
*   **[Active Defense & Perimeter Protection](port-forwarding_and_firewall.md):** Implementation of an aggressive perimeter security layer using Uncomplicated Firewall (UFW) and system log monitoring (Fail2Ban) to isolate malicious scanners.
*   **[Application Deployment (Nextcloud)](nextcloud.md):** Hardening data storage application access via restricted trusted domain parsing and enforced Multi-Factor Authentication (2FA) enforcement.
*   **[Network Bypass & Egress Tunneling](ngrok-setup.md):** Architectural implementation of reverse egress tunneling (Ngrok) to bypass Carrier-Grade NAT (CGNAT) topologies without standard inbound router port configuration.

## 🚨 Architectural Lessons Learned (The CGNAT Problem)
*   **Vulnerability Detection:** Running a public egress proxy (Ngrok) successfully bypassed ISP-enforced CGNAT restrictions but introduced significant automated reconnaissance traffic (botnets).
*   **Dynamic Defense Mitigation:** Utilized aggressive firewall controls to parse inbound application traffic and mitigate automated malicious login attempts at the edge.
