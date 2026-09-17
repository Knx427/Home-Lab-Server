### 🌐 Network Engineering: Multi-Interface Redundancy & Netplan Configuration

This engineering guide details the implementation of Layer 3 link redundancy on an Ubuntu Linux server infrastructure using the Netplan utility. The configuration establishes a dual-homed network architecture featuring persistent static IP addressing and an automated, metric-based network interface failover mechanism. 

### 🏗️ 1. High-Availability Interface Failover Design

### ✅ Architectural Impact

* **Layer 3 Link Redundancy:** Prevents total host isolation by pairing a primary wired connection with a secondary wireless infrastructure backhaul.
* **Metric-Cost Routing:** Employs dynamic routing metrics to prioritize data traffic over high-bandwidth links while maintaining an inactive backup path ready to ingest traffic instantly during an outage.

### 📊 Interface Priority Matrix

* **Primary Interface (eno1):** Configured with a lower metric cost (100), ensuring all data egress defaults through the high-performance wired interface.
* **Fallback Interface (wlan0):** Configured with a higher metric cost (200). The Linux kernel routing table keeps this link dormant until the primary wired link drops carrier signal.

### 🛠️ 2. Core Netplan Configuration

### ➤ Interface Discovery

Prior to mutating network configurations, extract the exact hardware interface designations mapped by the kernel using the iproute2 stack: 

```bash

ip address show
# Short variant: ip a
```

### ➤ Editing the Configuration State

Open the primary network definition configuration file using an elevated text editor: 

```bash

sudo nano /etc/netplan/01-netcfg.yaml

```

Inject the following YAML configuration block. Replace placeholder values (e.g., [YourEthAdapter], [YourSSID]) with your exact discovered hardware parameters: 

```yaml

network:
  version: 2
  renderer: networkd

  ethernets:
    eno1:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
      routes:
        - to: 0.0.0.0/0
          via: 192.168.1.1
          metric: 100

  wifis:
    wlan0:
      dhcp4: no
      addresses:
        - 192.168.1.101/24
      access-points:
        "YourSSID":
          password: "YourPassword"
      routes:
        - to: 0.0.0.0/0
          via: 192.168.1.1
          metric: 200

```

### ⚡ 3. Configuration Validation & Commitment

### ➤ Safe Runtime Testing

To avoid permanent lockout from a remote terminal due to an invalid syntax configuration, initiate Netplan's automated rollback sandbox: 

```bash

sudo netplan try

```

* **Analyst Note:** This command requires a confirmation input from the operator within a 120-second window. If connectivity drops or the config is malformed, Netplan automatically drops changes and rolls back to the previous stable networking state.

### ➤ Hard Committing Configuration Changes

Once the sandbox run succeeds, permanently apply the structural state to the systemd-networkd backend: 

```bash

sudo netplan apply

```

### 📝 4. Technical Implementation Notes

* **YAML Indentation Syntax:** Netplan relies strictly on precise spacing hierarchies. Tab characters must be completely avoided; indentation must consist exclusively of standard spaces aligned uniformly, or parsing runtime exceptions will display.
* **Gateway Best Practices:** Note that the legacy gateway4 key is officially deprecated in modern Netplan core engines. Defining egress gateways explicitly using the nested structured routes object (- to: 0.0.0.0/0 via: [Gateway_IP]) is the standard pattern required to ensure forward compatibility.
