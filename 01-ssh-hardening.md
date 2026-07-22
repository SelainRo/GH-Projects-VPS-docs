# Runbook: SSH Hardening and Key-Based Authentication Setup

**Date:** July 2026

**Target OS:** Debian 13 (Bookworm)

**Linux Kernel:** 6.12.43+deb13-amd64

**OpenSSH Server:** OpenSSH_10.0p2 Debian-7 (OpenSSL 3.5.1 1 Jul 2025)

**OpenSSH Client:** OpenSSH_9.6p1 Ubuntu-3ubuntu13.15 (OpenSSL 3.0.13 30 Jan 2024) 

**Infrastructure:** 1 vCPU, 1 GB RAM VPS

---

## 1. Objective

To secure a newly provisioned Debian 13 remote virtual private server (VPS) by completely disabling password authentication over SSH, enforcing cryptographic public-key authentication, and validating the configuration against external brute-force vectors.

---

## 2. Prerequisites

* A local client workstation running Linux Mint 22.2 with an existing SSH key pair (`~/.ssh/id_ed25519`).
* An Android device running Termux with an existing SSH key pair.
* Administrative (`root`) access to the remote VPS.
* The public keys from both devices pre-loaded into `/root/.ssh/authorized_keys` on the remote server.

---

## 3. Implementation Procedure

### Step 3.1: Main Configuration Modification

Open the primary SSH daemon configuration file:

```bash
nano /etc/ssh/sshd_config

```

Ensure the following parameter directives are explicitly configured and uncommented:

```text
PermitRootLogin prohibit-password
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no

```

---

## 4. Troubleshooting & Drop-In Configuration Conflicts

During initial implementation, the server continued to prompt for passwords despite the main `/etc/ssh/sshd_config` file being set to `PasswordAuthentication no`.

### Diagnostic Process:

1. Inspected active configuration directives:

```bash
grep -v '^#' /etc/ssh/sshd_config | grep -v '^$'

```

2. Identified inclusion directive: `Include /etc/ssh/sshd_config.d/*.conf`.
3. Checked drop-in directory:

```bash
ls -l /etc/ssh/sshd_config.d/

```

4. Located `50-cloud-init.conf`, which contained a conflicting `PasswordAuthentication yes` directive.

### Resolution:

Modified `/etc/ssh/sshd_config.d/50-cloud-init.conf` to enforce our security policy:

```text
PasswordAuthentication no

```

### Apply Changes:

Validated syntax accuracy and restarted the daemon:

```bash
sshd -t
systemctl restart sshd

```

---

## 5. Verification

### Verification Test A: Enforced Rejection

Attempt authentication while forcing password mode:

```bash
ssh -o PubkeyAuthentication=no root@[IP_ADDRESS]

```

* **Result:** Connection immediately terminated by remote server (`Permission denied (publickey)`).

### Verification Test B: Authorized Access

Attempt standard key connection:

```bash
ssh root@[IP_ADDRESS]

```

* **Result:** Handshake succeeded, granting immediate access.

---

*Note: This technical document was generated with the assistance of an LLM.*
