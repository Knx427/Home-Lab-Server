### 🍯 Threat Deception: Cowrie Medium-Interaction SSH/Telnet Honeypot Deployment

This deployment guide details the architecture, isolation configurations, and firewall log-routing mechanics implemented to deploy a Cowrie medium-interaction honeypot. This system is designed to attract, log, and analyze automated brute-force attacks and malicious tool deployments without risking host infrastructure. 

### 🚨 Incident Response Case Study: Tracking an Active Perimeter Breach

### 🔍 How I Caught a Real Attacker

"When I first configured this home lab infrastructure, I exposed the environment to the internet using the **Ngrok Free Service Tier** so I could connect remotely. Because my network was out in the open, automated internet bots found it almost instantly. 

I had deliberately bound the **Cowrie Honeypot** to default **SSH Port 22** using local firewall redirection rules to act as an early-warning tripwire. Shortly after going live, an automated threat actor successfully brute-forced their way into the fake honeypot shell. 

Once inside, the attacker initialized an interactive terminal session and executed a massive, complex malicious shell deployment script. When I parsed the captured keystrokes and command logs inside the SIEM dashboard, I discovered the embedded script text and developer comments were written completely in a foreign Slavic dialect—either **Czech or Serbian**. 

Seeing a live threat actor executing scripts inside my environment was the ultimate wake-up call. Even though they were trapped safely inside the honeypot sandbox, it proved that the free tier of Ngrok left my perimeter far too exposed to global background noise. This exact incident is the reason I tore down the public proxy setup entirely and **migrated the whole lab onto an authenticated, hidden Tailscale WireGuard mesh network** to drop my public attack surface to zero." 

### ⚠️ 1. Critical Security Constraint: Privilege Isolation

[!CAUTION]
**Enforcing the Principle of Least Privilege (No Sudo Execution):**
The Cowrie application binary tracking scripts (bin/cowrie start) **must never be executed using sudo or running under a privileged root account.** Cowrie's core source code features native kernel identity checks that actively block execution if root escalation is detected. 

*Risk Analysis:* Honeypots are intentionally exposed targets. If a highly sophisticated attacker discovers a zero-day vulnerability to break out of the Python terminal simulation wrapper (Sandbox Escape), their shell inherits the execution privileges of the process owner. Running Cowrie with sudo gives an attacker instant root kernel control over the entire actual host system. Isolating execution under an unprivileged user ensures an attacker remains trapped inside a powerless local boundary with zero administrative capability. 

### 🏗️ 2. Architectural Design & Network Isolation

### ✅ Architectural Impact

* **Production Simulation:** Cowrie mimics a vulnerable Linux operating system environment. When automated botnets or attackers connect, they are dropped into a fake shell that records their keystrokes, downloaded malware payloads, and credentials.
* **Management Plane Protection:** To safely run a honeypot on default **Port 22**, the server's actual management SSH daemon must be relocated to a hidden, non-standard socket (e.g., Port 5555). This ensures real admin access remains isolated from the public deception layer.

### 🛠️ 3. Sandbox Installation & Dependency Provisioning

To prevent a compromised honeypot from affecting the underlying host kernel, Cowrie is isolated within a dedicated system user space running inside a virtualized Python environment. 

### Step A: System User Creation & Package Ingestion

Create an unprivileged system user with disabled password login matrices to run the daemon safely: 

```bash

# Provision an isolated service user
sudo adduser --disabled-password --gecos "" cowrie

# Ingest required python development environments and virtualized containers
sudo apt update
sudo apt install git python3-venv python3-pip libssl-dev libffi-dev build-essential -y

```

### Step B: Environment Initialization

Switch to the isolated user space, clone the deployment repository, and establish the Python virtual environment: 

```bash

# Pivot to the secure service account profile
sudo su - cowrie

# Clone the core deception source code
git clone http://github.com/cowrie/cowrie.git
cd cowrie

# Initialize and activate the isolated virtual environment
python3 -m venv cowrie-env
source cowrie-env/bin/activate

# Upgrade pipeline managers and ingest requirements
pip install --upgrade pip
pip install -r requirements.txt

```

### 📁 4. Configuration & Port Redirection (iptables)

### Step A: Activating the Frontend Manifest

Copy the distribution baseline template into a localized active runtime override file: 

```bash

cp etc/cowrie.cfg.dist etc/cowrie.cfg
```

* **Analyst Note:** By default, Cowrie initializes its listening socket on non-root port **2222** because unprivileged users cannot bind directly to low-system ports (0-1023).

### Step B: Layer 4 Boundary Port Redirection

To trick automated botnets into hitting the honeypot seamlessly, configure the host firewall framework (iptables or ufw) to silently redirect incoming traffic from standard **Port 22** over to Cowrie's internal **Port 2222**: 

```bash

# Redirect all incoming traffic targeting Port 22 straight into the honeypot engine
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
```

To make these routing rules persistent across host server reboots, ingest the persistence manager: 

```bash

sudo apt install iptables-persistent -y
```

### 🧠 5. SIEM Log Shipping (Wazuh & Filebeat Integration)

### ✅ Threat Hunting Visibility

Cowrie stores all captured attacker keystrokes, used passwords, and executed command structures inside a standardized JSON format at /home/cowrie/cowrie/var/log/cowrie/cowrie.json. 

To forward this rich threat intelligence data directly into your **Wazuh SIEM Manager**, append an explicit log-harvesting file tracker to your centralized configuration blocks: 

```xml

<localfile>
  <location>/home/cowrie/cowrie/var/log/cowrie/cowrie.json</location>
  <log_format>json</log_format>
</localfile>
```

* **Analyst Monitoring Note:** Once synced, custom Wazuh decoders parse the JSON stream automatically, allowing you to build real-time Kibana dashboards tracking the top source countries attacking your lab, the most common passwords attempted, and downloaded malicious shell scripts.
