### 🛡️ Hardened Linux Server Infrastructure & Secure Remote Access

### Project Architecture

Deployed a dedicated, self-hosted Linux server architecture leveraging repurposed hardware running **Ubuntu Server 22.04 LTS (Minimal Infrastructure)**. The core objective of this project was to establish a resilient web service hosting environment while engineering around real-world Layer 3 networking limitations (CGNAT), deploying proactive threat deception, and mitigating automated perimeter threats. 

### 📁 Technical Documentation Directory

* **[OS Installation & Hardening](setup-installation.md):** Bare-metal provisioning instructions, storage layout configuration using LVM, anti-forensics file sanitization via shredding, and full-disk encryption using LUKS.
* **[Network Engineering (Netplan)](netplan-config.md):** Configuration files for dynamic interface routing, persistent static IP addressing, and high-availability secondary wireless interface backhaul failovers.
* **[Host Hardening & SSH Security](ssh-setup.md):** Mitigation of unauthorized remote access vectors through enforced cryptographic Key-Based Authentication, disabled root logins, custom port mapping, and system-level alerts.
* **[Active Defense & Perimeter Protection](port-forwarding_and_firewall.md):** Implementation of an aggressive perimeter security layer using Uncomplicated Firewall (UFW) and dynamic Layer 4 filtering to identify transport layer blocks.
* **[Threat Deception & Honeypot Configuration](cowrie-setup.md):** Deployment of an isolated, unprivileged medium-interaction Cowrie honeypot to attract, contain, and analyze automated brute-force attacks on default port 22.
* **[Application Deployment (Nextcloud)](nextcloud.md):** Hardening data storage application access via restricted trusted domain parsing and enforced Multi-Factor Authentication (2FA/MFA) access controls.
* **[Network Bypass & Egress Tunneling](ngrok-setup.md):** Architectural implementation of reverse egress tunneling (Ngrok free tier vs. enterprise profiles) to bypass Carrier-Grade NAT (CGNAT) topologies.

### 🚨 Architectural Case Study: The CGNAT & Perimeter Exposure Incident

* **Vulnerability Ingestion:** Deploying an outbound public proxy (Ngrok Free Tier) successfully bypassed ISP-enforced CGNAT restrictions but exposed raw application endpoints to the public internet without an upstream edge firewall, attracting global automated reconnaissance botnets.
* **Active Threat Detection:** By intentionally routing port 22 traffic into a sandboxed **Cowrie Honeypot**, an active brute-force intrusion was caught in real time. The threat actor executed a complex malicious shell deployment script with developer notes written entirely in a Slavic language (Czech/Serbian).
* **Remediation & Architectural Pivot:** To remediate the threat exposure, the public proxy framework was deprecated. The entire server data plane was migrated onto an authenticated, completely hidden **Zero Trust WireGuard overlay mesh network (Tailscale)**, reducing the public perimeter attack surface to absolute zero.

---
## ✍️ Author
Created and maintained by **knx427**  
_Network Technician | Cybersecurity Infrastructure & Monitoring Specialist_  

📫 *Contributions, suggestions, or technical discussions are always welcome!*
