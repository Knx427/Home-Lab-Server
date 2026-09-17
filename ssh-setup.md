### 🔐 Host Hardening: Secure Shell (SSH) Isolation & Active Log Defense

This technical blueprint details the security configurations required to protect the remote administration plane of the Linux server. It details the process of disabling password authentication, shifting network sockets, implementing an automated real-time login alert mechanism, and deploying Fail2Ban to establish a dynamic network defense perimeter against automated threat actors. 

### 🔒 1. Management Plane Hardening (OpenSSH Configuration)

### ✅ Architectural Impact

* **Credential Guessing Neutralization:** Disabling password authentication forces the OpenSSH subsystem to accept cryptographic keys exclusively. This completely neutralizes internet-wide automated brute-force attacks.
* **Privileged Session Restriction:** Disabling direct root entry restricts the initial entry point exclusively to low-privileged accounts, enforcing standard identity accountability and forcing the explicit use of sudo tracking logs.

### ➤ Secure Daemon Modification

Open the primary daemon configurations with administrative privileges: 

```bash

sudo nano /etc/ssh/sshd_config

```

Enforce the following baseline structural variables within the file parameters: 

```text

PasswordAuthentication no
PermitRootLogin no
Port 2222 
```

* **Analyst Note:** Relocating the socket to a non-standard port (e.g., 2222) implements basic port obfuscation. While it does not substitute for true cryptographic authentication, it significantly reduces systemic background scanning noise from generic internet worms.

### ➤ Service Lifecycle Integration

Validate configuration consistency, commit changes, and update the runtime environment state: 

```bash

# Enable persistent service states on system boot execution
sudo systemctl enable ssh

# Restart the daemon container to bind the alternative port socket
sudo systemctl restart ssh

# Inspect the active listener state and audit local log entries
sudo systemctl status ssh

```

### 📧 2. Incident Response Automation: Real-Time Login Alert Dispatches

### ✅ Architectural Impact

* **Continuous Infrastructure Visibility:** Employs an event-driven session script that immediately hooks into any successful authentication loop, extracting client source IPs and pushing high-priority out-of-band alerts straight to the security administrator's mail hub.

### ➤ Step A: Mail Transfer Configuration (ssmtp)

Deploy the secure Simple Mail Transfer Protocol utility to act as your localized telemetry forwarding client: 

```bash

sudo apt install ssmtp -y

```

Modify the global outbound email parameters configuration mapping file: 

```bash

sudo nano /etc/ssmtp/ssmtp.conf

```

Append the matching parameters to establish an authenticated, TLS-secured handshake connection string via Google's relay channels: 

```text

root=your_email@gmail.com
mailhub=smtp.gmail.com:587
AuthUser=your_email@gmail.com
AuthPass=your_16_character_app_password
UseSTARTTLS=YES

```

* **Analyst Note:** To secure authentications, you must navigate to Google Account Security, activate 2FA, and generate a dedicated, non-spaced 16-character **App Password** to fill the AuthPass variable space.

### ➤ Step B: Session Hook Scripting (sshrc)

The OpenSSH engine natively reads the ```/etc/ssh/sshrc``` configuration block instantly upon completing a user session handshake before dropping them into a shell. 

Initialize the persistent notification engine file: 

```bash

sudo nano /etc/ssh/sshrc

```

Inject the structured notification string pattern down the pipeline: 

```bash

#!/bin/bash
echo "SSH Session Initialization Detected from Source IP: $SSH_CLIENT to Host Target: $(hostname) on Domain Time: $(date)" | mail -s "CRITICAL: Host SSH Management Access Alert" your_email@example.com

```

Apply precise security execution parameters to allow the host shell parser to trigger it: 

```bash

sudo chmod +x /etc/ssh/sshrc

```
### 🛡️ 3. Intrusion Prevention Engine Integration (Fail2Ban)

### ✅ Architectural Impact

* **Automated Threat Mitigation:** Integrates log parsing loops directly with the local host-level firewall (iptables / nftables). If a remote host IP exceeds the threshold of failed log-in handshakes, Fail2Ban drops their traffic at the network edge before it can impact target system resources.

### ➤ Safe Configuration Ingestion

Install the active defense client application: 

```bash

sudo apt install fail2ban -y

```

* **Production Configuration Rule:** Never modify the default ```/etc/fail2ban/jail.conf``` master configuration array directly, as upstream system updates will overwrite your modifications. Always clone it to a persistent .local file:

```bash

sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

```

Locate the dedicated OpenSSH monitoring container header blocks and adjust parameters to monitor your customized ingress socket: 

```ini

[sshd]
enabled = true
port    = 2222      # Matches your custom OpenSSH Port configuration
logpath = %(sshd_log)s
backend = systemd

```

### ➤ Fine-Tuning Global Threat Metrics

Within the identical ```/etc/fail2ban/jail.local``` layout framework, locate the [DEFAULT] cluster metrics configuration workspace to alter penalty durations: 

```ini

bantime  = 3600     # Duration in seconds to drop malicious connection strings (1 Hour)
findtime = 600      # Analysis window to evaluate incoming log failure tracking (10 Minutes)
maxretry = 5        # Absolute threshold of failed authentication loops before triggering a ban

```

### ➤ Service Lifecycle & Operational Auditing

Initialize the prevention system framework daemons and request runtime statistics metrics: 

```bash

# Enable and spawn active prevention background daemons
sudo systemctl enable fail2ban
sudo systemctl restart fail2ban

# Query the real-time operational defense state of the secure shell tracking container
sudo fail2ban-client status sshd

```

### 📝 4. Operational Risk Mitigation Notes

* **Network Ingress Inversion Constraints:** If your perimeter environment operates inside an Internet Service Provider **Carrier-Grade NAT (CGNAT)** IP lease network topology, standard external port-forwarding mappings are blocked at the provider boundary.
* **Remediation Mapping:** To re-establish a functional external administration terminal line without creating public vulnerabilities, refer back to the automated **Ngrok Outbound Proxy Tunneling Setup** layout, or leverage an encrypted, identity-authenticated connection via the **Tailscale Mesh Overlay VPN Architecture**.
* **Key Protection Lifecycle Rules:** Always implement high-entropy passphrases to encrypt your local tracking private keys (id_ed25519). A private key without a password functions as a Master Key; if an external attacker compromises your client desktop device, they instantly gain lateral access to your target servers.
