### 💿 OS Deployment & Bare-Metal Infrastructure Hardening (Ubuntu Server)

This engineering deployment guide details the operating system provisioning, storage layout logic (LVM), and Full Disk Encryption (FDE) security parameters executed to harden a recycled mobile laptop asset repurposed as a dedicated headless Linux server. 

### 🌐 1. Secure Media Acquisition & Provisioning

### ✅ Security Analysis

* **Trusted Source Verification:** ISO download is restricted exclusively to authenticated Canonical distribution nodes to prevent supply chain image tampering.
* **Long-Term Support (LTS) Baseline:** The LTS tracking branch is selected to guarantee continuous automated security patches, stabilizing the platform against zero-day infrastructure exploits.

### ➤ Execution Steps

1. Download the official authenticated Ubuntu Server LTS ISO: [Ubuntu Server Download](https://ubuntu.com/download/server).
2. Employ an image-flashing utility (e.g., *Rufus*) to write the image file to a clean USB medium.
3. Boot the target laptop hardware from the external USB interface and initialize the installer framework.
4. **Language & Keyboard:** Select your preferred language parameters and precise keyboard tracking layouts.

### 🔌 2. Initial Network Ingestion

During the bare-metal installation loop, configure the main network interface to leverage dynamic **DHCP** (Dynamic Host Configuration Protocol). 

* **Analyst Strategy Note:** Allocating a temporary dynamic lease at boot ensures the server can immediately query upstream repositories to fetch missing system packages and critical security hotfixes. Transitioning to a persistent static IP and wireless failover matrix is deferred until post-install layer configurations via Netplan.

### 🔒 3. Storage Layer Architecture & Hardening (LVM & LUKS)

### ✅ Architectural Impact

* **Logical Volume Management (LVM):** Grants system administrators the power to dynamically upscale, split, or migrate logical storage layers without reformatting raw partitions, allowing Nextcloud storage data matrices to scale elastically over time.
* **LUKS (Linux Unified Key Setup) Full Disk Encryption:** Implements hardware block-level encryption. Because this deployment runs on a mobile laptop form factor, LUKS is the primary control safeguarding data integrity against physical host theft or external drive extraction.

### ➤ Cryptographic Key Management Lifecycle

Once the base OS installation is completed, the initial recovery credentials must be hardened and duplicated safely. 

Discover your exact hardware block mapping profiles by executing: 

```bash

lsblk
```

### Secure Recovery Key Isolation

Encrypt the plain-text recovery key using symmetric GPG encryption before executing an off-host data export sequence: 

```bash

gpg -c /root/luks-recovery.key
```

### Provisioning an Alternative Key Slot (Up to 8 Supported Slots)

LUKS partition tables accept multiple access variables. Generate a secondary cryptographic key directly from the kernel random number generator and link it to the drive array: 

```bash

# Generate a 64-byte high-entropy binary file from the random pool
sudo dd if=/dev/urandom of=/root/luks-recovery.key bs=1 count=64

# Inject the random token into an available LUKS key slot descriptor
sudo cryptsetup luksAddKey /dev/[disk] /root/luks-recovery.key

```
### Anti-Forensics Safe Wipe Protocol

* **Analyst Note:** Standard system deletion tools (rm) only decouple pointers, leaving binary markers intact on disk. To prevent forensic reconstruction of cryptographic keys, overwrite the storage blocks with pseudorandom arrays before dropping the file system hooks:

```bash

sudo shred -u /root/luks-recovery.key

```

### 🔑 4. Identity & Access Management (SSH Key-Based Authentication)

### ✅ Architectural Impact

* **Ed25519 Elliptic Curve Cryptography:** Enforces the Edwards-curve Digital Signature Algorithm for remote ingress. Ed25519 provides superior cryptographic security, faster computation times, and optimal verification performance compared to legacy RSA layouts, making it highly resilient against advanced compute brute-force attempts.
* **Zero-Password Perimeter:** Ingesting public keys directly at the operating system bootstrapping phase establishes a secure administrative plane from minute one, rendering traditional credential-guessing vectors completely useless.

### ➤ Keypair Generation & GitHub Synchronization

If you do not possess an active modern keypair, initialize a secure Ed25519 token set on your remote management client machine: 

```bash

ssh-keygen -t ed25519 -C "Home-Lab Server Admin Key"

```

This execution drops two critical files into your local directory: 

* ~/.ssh/id_ed25519 (**Private Key** - Must be strictly guarded on your client machine)
* ~/.ssh/id_ed25519.pub (**Public Key** - The shareable transport token)

### Automated GitHub Ingestion Loop

During the Ubuntu installation interface prompt, select **"Import SSH key from GitHub"** and input your specific user handler. The installation manager will automatically fetch your public keys registered on your profile configuration parameters (Github -> Settings -> SSH and GPG keys). 

### 📦 5. Installation Finalization & Lifecycle Handover

1. **Server Software Provisioning (SAPS):** Select the target baseline server software applications needed for execution (Nextcloud for data cloud storage, OpenSSH for secure system maintenance).
2. **Post-Execution Reboot:** Conclude the automated deployment loop and select reboot. Disconnect the physical USB media when prompted by the kernel. *(If a display lockout occurs during system teardown, execute a manual hardware cycle).*

### ➡️ Next Steps in Deployment

Once the server reboots successfully and presents a clean login terminal prompt, pivot directly to the secondary infrastructure layer: 

* [Netplan Configuration](netplan-config.md): Static IP Assignment & High-Availability WiFi Failover.
