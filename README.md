*This project has been created as part of the 42 curriculum by manmanuk.*

#  Born2beRoot — v5.2

##  Description
**Born2beRoot** is a foundational System Administration project in the 42 School curriculum. The core objective is to configure a secure, standalone server using virtualization software, completely devoid of a graphical user interface (GUI). The project introduces essential infrastructure practices, including encrypted disk partitioning via LUKS and LVM, strict user account privilege controls (sudo), comprehensive network security via firewall rules, automated monitoring configurations, and service setups (SSH and web stack).

---

##  Instructions

### 1. Generating the Signature
To submit the project, you must calculate the SHA-1 hash of your virtual machine's virtual disk (`.vdi` or `.qcow2`). 
*Run the appropriate command on your host machine inside the virtual machine directory:*

*   **Linux:** `sha1sum manmanuk42.vdi`
*   **macOS:** `shasum manmanuk42.vdi`
*   **macOS (M1/UTM):** `shasum Linux.utm/Data/03794DE3-E46C-4B76-8CE1-D8B540A5A595.qcow2`
*   **Windows:** `certUtil -hashfile manmanuk42.vdi sha1`

> [!IMPORTANT]
> Save the exact resulting string directly inside your `signature.txt` file at the root of your repository. Do not boot the machine after calculating the final hash, as boot operations modify the signature.

### 2. Basic Management Commands
Once logged into the server as `manmanuk` or `root`, use these core components to verify the setup:

*   **Check Firewall Status:** `sudo ufw status numbered`
*   **Check SSH Status:** `sudo systemctl status ssh`
*   **Check Active Users:** `who` or `w`
*   **Check LVM Structure:** `sudo lsblk` or `sudo lvs`

---

## ⚙️ Project Choices & Technical Overview

###  Operating System: Debian vs Rocky Linux
For this implementation, **Debian** was selected.
*   **Debian Advantages:** Highly stable, utilizes the predictable `apt` package manager, lighter out-of-the-box configuration, and contains a massive support ecosystem.
*   **Debian Disadvantages:** Packages inside the stable branch update slower to preserve system safety, leading to older software versions.
*   **Rocky Linux Alternative:** Built as a downstream binary-compatible release of Red Hat Enterprise Linux (RHEL). It is optimized for enterprise servers but carries complex baseline setups (like mandatory SELinux rules instead of AppArmor).

###  Security Mechanisms: AppArmor vs SELinux
*   **AppArmor (Debian standard):** Path-based Mandatory Access Control (MAC). It bounds individual application capabilities using profiles assigned to exact paths (e.g., `/usr/sbin/sshd`). It is straightforward to read, configure, and maintain.
*   **SELinux (Rocky Linux standard):** Label-based MAC system. It assigns security contexts and labels to every process, file system block, and network port. It is significantly more precise but harder to configure and troubleshoot.

###  Firewalls: UFW vs firewalld
*   **UFW (Uncomplicated Firewall):** A simplified front-end wrapper for `iptables`. Perfect for small server profiles where opening and closing absolute structural ports (like port 4242) requires minimal syntax.
*   **firewalld:** Zone-based dynamic firewall controller supporting runtime changes without connection resets. Standard on RHEL/Rocky, but introduces unnecessary layers for a single-service isolated server.

###  Virtualization Hypervisors: VirtualBox vs UTM
*   **VirtualBox:** A Type 2 hosted hypervisor compatible across x86 architectures (Intel/AMD). It provides precise hardware controls, native snapshots, and networking options like NAT and Bridged adapters.
*   **UTM:** A lightweight container built specifically for macOS Apple Silicon (ARM64 M1/M2/M3 chips) using Apple's Hypervisor framework and QEMU. 

---

## 🔒 Implemented Security Policies

### 🔑 Strong Password Requirements
Configured inside `/etc/login.defs` and `/etc/pam.d/common-password`:
*   **Expiration:** Enforced every 30 days maximum, with a minimum interval of 2 days between updates.
*   **Alert Time:** Warns users 7 days prior to threshold breach.
*   **Complexity Rules:** Minimum length of 10 characters. Enforces at least 1 uppercase letter, 1 lowercase letter, and 1 numeric digit. Maximum of 3 identical consecutive characters allowed. Reject matches including the active username.

###  Sudo Group Restrictions
Configured using `sudo visudo`:
*   Limited to 3 baseline credential entry attempts.
*   Enforced a custom insults/error statement on failure.
*   Every single operation input and terminal log output archived within `/var/log/sudo/`.
*   Enforced strict paths utilizing `secure_path`.

---

##  Monitoring Script (`monitoring.sh`)
An automated `bash` routine deployed via system `cron` tasks running every 10 minutes to broadcast systems telemetry via `wall`:
*   **Collected Vectors:** Kernel architecture, physical processor footprint, virtual CPU core totals, RAM utility ratios, storage footprint metrics, baseline CPU performance load percentages, time context of the last boot initialization, active LVM mapping states, active TCP hooks, terminal network user counts, current local IPv4/MAC mapping, and global operational counters tracking `sudo`.

---

##  Resources & AI Statement

### Traditional References
*   [Debian Documentation & Hardening Manual](https://www.debian.org/doc/)
*   [LVM Architecture and Management Tutorial](https://wiki.debian.org/LVM)
*   [LUKS/dm-crypt Disk Encryption Guide](https://gitlab.com/cryptsetup/cryptsetup)

### AI Usage & Countermeasures Disclosure
In adherence to the **42 AI Instructions**, an AI assistant was treated solely as an interactive conceptual mentor. 
*   **What it was used for:** To clarify complex logic strings involving nested tool commands like `awk`, `free`, and `sysfs` inside the `monitoring.sh` script, and to contrast structural engineering features like AppArmor path boundaries versus SELinux labels.
*   **What it was NOT used for:** No raw terminal configuration scripts or layout files were directly auto-generated or copied. All structural partition sizing selections, userspace definitions, and structural passwords were formulated independently to ensure full understanding during peer defense reviews.