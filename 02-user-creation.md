# Runbook: Non-Root User Creation, Sudo Privilege Allocation, and SSH Key Setup

## Metadata

* **Date:** July 2026


* **Target Operating System:** Debian 13 (Bookworm) (Kernel `6.12.43+deb13-amd64`)


* **Hardware Configuration:** 1 vCPU, 1 GB RAM, x86_64 architecture


* **Software Binaries:** OpenSSH Server `10.0p2 Debian-7`, OpenSSH Client `9.6p1`

* **Workstation Environment:** Linux Mint 22.2



## Objective

Establish an unprivileged system administration user account (`sysadmin`), grant administrative command elevation via `sudo`, configure POSIX permissions for Ed25519 public key authentication, and resolve system hostname mapping to eliminate direct operational reliance on the `root` account.

## Prerequisites

* Interactive root shell access to the headless remote server over SSH.


* OpenSSH client installed on the local administration workstation (`Linux Mint 22.2`).


* Inbound TCP port 22 access enabled.



## Implementation

### Step 1: User Account Provisioning and Privilege Allocation

Execute the following commands as `root` on the remote server to provision the user account and attach it to the `sudo` group:

```bash
# Provision the unprivileged user account
adduser sysadmin

# Attach the user to the sudoers group
usermod -aG sudo sysadmin

```

### Step 2: Client SSH Key Pair Generation

Execute on the local client workstation to generate an Ed25519 key pair dedicated to the administrative account:

```bash
# Generate Ed25519 key pair with destination path
ssh-keygen -t ed25519 -C "sysadmin@vps" -f ~/.ssh/id_ed25519_vps_sysadmin

```

### Step 3: Public Key Installation and Permission Hardening

Deploy the generated public key to the remote host and set explicit POSIX file permissions:

```bash
# Create the SSH configuration directory for sysadmin
mkdir -p /home/sysadmin/.ssh

# Append the public key string to authorized_keys
nano /home/sysadmin/.ssh/authorized_keys

# Apply restrictive POSIX filesystem permissions
chmod 700 /home/sysadmin/.ssh
chmod 600 /home/sysadmin/.ssh/authorized_keys
chown -R sysadmin:sysadmin /home/sysadmin/.ssh

```

### Step 4: Hostname Loopback Resolution

Map the local fully qualified domain name (FQDN) and short hostname to the IPv4 loopback address inside `/etc/hosts` to eliminate name resolution delays during `sudo` calls:

```bash
echo "127.0.0.1 cv7412659.novalocal cv7412659" >> /etc/hosts

```

## Troubleshooting & Edge Cases

### Issue 1: PAM Shadow Database Lock

* **Symptom:** `adduser` terminates with `passwd: Authentication token manipulation error`.


* **Root Cause:** Pluggable Authentication Modules (PAM) reject password generation during uninitialized or locked shadow database entries on cloud-init images.


* **Resolution:** Abort the prompt wizard and execute `passwd sysadmin` directly as `root` to overwrite the entry in `/etc/shadow`.



### Issue 2: Hostname Resolution Failure During Sudo Elevation

* **Symptom:** Terminal emits `sudo: unable to resolve host [HOSTNAME]: Name or service not known`.


* **Root Cause:** `sudo` queries `gethostname()` and fails to resolve the system network identifier via `/etc/hosts` or local DNS.


* **Resolution:** Append `127.0.0.1 [HOSTNAME]` directly to `/etc/hosts`.



## Verification

### 1. Test SSH Public Key Authentication

Execute from the local workstation terminal:

```bash
ssh -i ~/.ssh/id_ed25519_vps_sysadmin sysadmin@[SERVER_IP]

```

*Expected Result:* Authentication completes via the key passphrase, yielding interactive shell `sysadmin@cv7412659:~$` without password prompts.

### 2. Test Sudo Privilege Elevation

Execute inside the remote `sysadmin` session:

```bash
sudo whoami

```

*Expected Result:* Prompts for the `sysadmin` account password and outputs `root` with no hostname lookup warnings.

---

*Note: This technical document was generated with the assistance of an LLM.*
