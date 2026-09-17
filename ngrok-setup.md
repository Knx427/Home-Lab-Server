### 🌐 Edge Routing: Bypassing CGNAT via Public Reverse Egress Tunnels (Legacy Setup)

This document details the configuration, automation scripting, and systemd deployment required to establish an outbound reverse tunneling architecture using Ngrok. This setup was engineered to expose internal HTTP and TCP services (Nextcloud and SSH management planes) over an ISP-enforced Carrier-Grade NAT (CGNAT) perimeter. 

[!WARNING]
**Architectural Exposure & Threat Intelligence Notice:**
Exposing internal host infrastructure directly to the public internet using generic reverse proxies (Ngrok subdomains) removes perimeter boundary controls. Within hours of establishing these open ingress vectors, local SIEM monitoring (Wazuh/Fail2ban) registered a dramatic spike in automated malicious reconnaissance probes, brute-force dictionary attacks, and widespread botnet scanning. 

*Service Tier Security Analysis:* It is critical to note that this exposure is an inherent limitation of the **Ngrok Free Service Tier**, not a failure of Ngrok as an enterprise service platform. The free tier lacks native ingress filtering, meaning any automated internet scanner can hit your tunnel endpoint directly. Paid enterprise tiers mitigate this risk entirely by embedding critical edge defense layers—such as mandatory IP Whitelisting, Geo-blocking, and native OpenID Connect (OIDC) / OAuth identity provider integrations—blocking malicious traffic at Ngrok's edge before it ever traverses the egress tunnel into your home network. 

*Architectural Evolution:* Because the free service tier provided insufficient edge filtering for this specific deployment, the topology was deprecated in favor of a private, identity-driven WireGuard mesh overlay network. For details on the hardened architecture, see the **Tailscale Integration & Mesh Network Migration Log**. 

### 🛠️ 1. Infrastructure Requirements & Ingestion Initialization

### ✅ Architectural Impact

* **Outbound Egress Traversal:** Initiates a persistent outbound TCP handshake to Ngrok's public relays. This creates a bi-directional tunnel, bypassing local router port-forwarding restrictions and CGNAT boundaries completely.

### Installation & Client Provisioning

Deploy the foundational tunnel client software natively via the primary package tree: 

```bash

sudo apt install ngrok -y

```

1. Secure an active account profile via the administrative panel ([ngrok.com](ngrok.com)) to acquire a unique host authorization token and static subdomain reservation.
2. Inject the cryptographic identifier into the local environment to authenticate your agent runtime:

```bash

ngrok authtoken YOUR_AUTH_TOKEN

```

### 📁 2. Central Tunnel Manifest Engineering (ngrok.yml)

[!NOTE]
Ensure the underlying Nextcloud storage application framework has been primed to receive incoming HTTP host traffic distributions before initializing the tunnel routing matrix. 

Open the structural configuration layout stored inside the snap profile boundary workspace: 

```bash

sudo nano ~/snap/ngrok/current/.config/ngrok/ngrok.yml

```

Inject the multi-tunnel definition matrix. This file defines simultaneous ingestion endpoints for web application routing and secure shell multiplexing: 

```yaml

version: 3
tunnels:
    nextcloud:
        proto: http
        addr: 80
        domain: [Your-static-Domain-Name].ngrok-free.app
        auth: "Username:Password" # Secondary HTTP Basic Auth validation gate protecting the application boundary
    ssh:
        proto: tcp
        addr: 5555 # Hardened local non-standard SSH listener port
agent:
    authtoken: [Your-Auth-Token]

```

### Sandbox Runtime Evaluation

Validate syntax configurations and initialize the tunnel arrays manually before committing to a persistent background service state: 

```bash

ngrok start --all --config ~/snap/ngrok/current/.config/ngrok/ngrok.yml

```

Verify secure remote connectivity over the public TCP relay and browser interfaces: 

```bash

# Terminal SSH Verification Tunnel Cross
ssh youruser@0.tcp.ngrok.io -p [ASSIGNED_EXTERNAL_PORT]

# Web GUI Verification Ingress 
http://[Your-Static-Domain-Name].ngrok-free.app

```

### 📜 3. Automated Telemetry Parsing & Notification Scripting

Because Ngrok assigns dynamic port mappings randomly to TCP/SSH streams upon initial boot execution, a shell script was compiled to query the agent's internal administrative API, harvest the dynamic parameters via jq, and email the data straight to the administrator. 

Create the automation utility at ```~/bin/ngrok-notify.sh```: 

```bash

sudo nano ~/bin/ngrok-notify.sh

```

```bash

#!/bin/bash

# Hold script execution to allow the ngrok core daemon to negotiate handshakes completely
sleep 10

# Query the local agent api loopback loop to scrape public connection strings
URLS=$(curl -s http://127.0.0.1:4040/api/tunnels | jq -r '.tunnels[] | .public_url')

# Persist active string maps to a local directory for monitoring verification
echo "$URLS" > "$HOME/ngrok_urls.txt"

# Forward live connectivity coordinates to the centralized operations email account
echo -e "Ngrok tunnels are up:\n\n$URLS" | mail -s "Ngrok Public Routing Strings: SSH & Nextcloud" your@email.com

```

Apply operational file-system execution constraints to the automation binary: 

```bash

chmod +x ~/bin/ngrok-notify.sh
```

### ⚙️ 4. Systemd Service Orchestration

To ensure persistent system uptime, link recovery, and hands-free initialization at system boot, establish dual dependent initialization definitions inside the system initialization hierarchy. 

### Service 1: The Core Ingress Tunnel Engine (ngrok.service)

```bash

sudo nano /etc/systemd/system/ngrok.service

```

```ini

[Unit]
Description=Ngrok Tunnel Infrastructure for SSH and Nextcloud Services
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/snap/ngrok/current/ngrok start --all --config=/home/[user]/snap/ngrok/current/.config/ngrok/ngrok.yml
WorkingDirectory=/home/[user]
Restart=on-failure
RestartSec=5
User=[user]

[Install]
WantedBy=multi-user.target

```

### Service 2: The One-Shot Ingest Notification Trigger (ngrok-notify.service)

```bash

sudo nano /etc/systemd/system/ngrok-notify.service

```

```ini

[Unit]
Description=Automated Telemetry Dispatcher - Scrape and Forward Public URLs
After=ngrok.service
Requires=ngrok.service

[Service]
ExecStart=/home/[user]/bin/ngrok-notify.sh
Type=oneshot
User=[user]

[Install]
WantedBy=multi-user.target

```

### Registering and Spawning the Infrastructure Daemons

Reload the system supervisor kernel, commit the systemd configurations, and hook the services into the default multi-user target array: 

```bash

sudo systemctl daemon-reload
sudo systemctl enable --now ngrok.service
sudo systemctl enable --now ngrok-notify.service

```

### 🔍 5. Infrastructure Auditing & Daemon Verification

Verify the active runtime state and pid socket tracking mapping tables directly from the system process landscape: 

```bash

# Check the operational health of the background tunnel workers
systemctl status ngrok.service --no-pager

# Cross-examine operational PIDs running out of the system binary root
ps -ef | grep ngrok

```
### 📝 6. Operational Risk Mitigation Notes

* **Data Confidentiality Rules:** Public endpoint routing strings (ngrok-free.app / 0.tcp.ngrok.io) serve as discoverable paths. Never expose these unique links in public spaces, forums, or open documentation logs to prevent automated malicious reconnaissance campaigns.
