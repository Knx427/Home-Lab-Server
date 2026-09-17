### 🧱 Perimeter Security: Layer 4 Transport Filtering & Ingress Port Forwarding

This engineering document details the edge routing configurations, network address translation (NAT) mapping, and local host firewall rules required to securely expose specified internal network services while establishing a strict default-deny perimeter boundary. 

### 🗺️ 1. Edge Layer 4 Network Address Translation (NAT)

### ✅ Architectural Impact

* **Port Obfuscation:** Mapping a non-standard external port to an alternative internal port decreases exposure to automated public scanner traffic looking for default listeners (like port 22).
* **Target Ingress Mapping:** Explicitly routes traffic across the edge gateway router straight to the dedicated static IP of the hardened host platform.

### ➤ Ingress Mapping Blueprint

Before configuring edge routing, confirm the local SSH listening socket configuration on the server host file: 

```bash

grep -i "Port" /etc/ssh/sshd_config
```

Configure your boundary gateway or router configuration matrix according to the following parameter specifications: 

| External Ingress Port | Target Internal IPDest |  Internal Port  | Transport Protocol |      Service Target     |
|-----------------------|------------------------|-----------------|--------------------|-------------------------|
|      ****2222****     |    192.168.1.100       |     **5555**    |         TCP        | Hardened SSH Management |
|  ****80** (or 443)**  |    192.168.1.100       | **80** (or 443) |         TCP        |    Nextcloud Web Core   |


### 🛡️ 2. Host-Level Firewalls (Uncomplicated Firewall - UFW)

### ✅ Architectural Impact

* **Default-Deny Posture:** Enforces an absolute drop policy on all unsolicited inbound network connections, ensuring that ports not explicitly whitelisted are completely invisible to network probes.
* **Stateful Inspection Access:** Explicitly permits incoming TCP handshakes only onto verified application and administration server sockets.

### ➤ Firewall Implementation Command Sequence

Execute the following commands on the Ubuntu Linux host terminal to define and enforce local packet-filtering policies: 

```bash

# Explicitly whitelist the customized internal secure shell socket
sudo ufw allow 5555/tcp

# Whitelist standard HTTP/HTTPS channels for local application delivery
sudo ufw allow 80/tcp

# Establish global edge boundaries (Block all Ingress / Allow all Egress)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Enforce rules and initialize the firewall engine at system boot
sudo ufw enable

```

To verify the active operational runtime status and inspect the live rule tables, execute: 

```bash

sudo ufw status verbose

```

### 🚨 3. WAN Interface Diagnostics & CGNAT Inversions

### ✅ Architectural Constraint

Traditional port forwarding maps a public WAN IP directly to a local LAN asset. However, if your Internet Service Provider implements **Carrier-Grade NAT (CGNAT)**, your router does not possess a true public IP address. Instead, it is assigned a shared middleman IP, which renders standard inbound port forwarding entirely non-functional. 

### ➤ How to Diagnose CGNAT Constraints

Log into your local boundary router and inspect the assigned **WAN/Internet Interface IP Address**. If the address falls within any of the following blocks, you are trapped behind a double-NAT network framework: 

* ```100.64.0.0 through 100.127.255.255``` (Official IANA Shared Transition Space)
* ```10.0.0.0/8, 172.16.0.0/12, or 192.168.0.0/16``` (Private RFC 1918 Spaces misconfigured as WAN leases)

### ➤ Remediation Paths

If your network analysis confirms a CGNAT lease, native port forwarding cannot span the internet boundary. You must bypass Layer 3 routing limitations by employing an outbound-initiated architecture: 

1. Review the legacy **Ngrok Outbound Tunnel Orchestration Manual** for public web access proxy configurations.
2. Review the modern **Tailscale WireGuard Mesh Network Integration Log** to establish a private, authenticated Zero Trust connection boundary.
