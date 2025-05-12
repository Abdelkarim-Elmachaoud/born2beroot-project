# Born2beRoot

## 🧠 Project Overview

**Born2beRoot** is a system administration project from the 42 curriculum that challenges you to install and configure a secure, stable Linux virtual machine. You'll learn essential sysadmin skills such as partitioning with LVM, user and group management, enforcing password policies, configuring services, and securing access.

---

## 🧭 Step-by-Step Guide

### 1. 🚀 Virtual Machine Setup

- **Choose OS**: Debian 11 (recommended) or Rocky Linux 9.
- **Virtualization**: Use VirtualBox or UTM.
- **Create VM**:
  - 2GB RAM minimum
  - 2 CPU cores
  - 10-20 GB disk space (dynamically allocated)
  - Enable EFI and allocate 1 network adapter (NAT or Bridged)

---

### 2. 💽 Partitioning & LVM

During OS installation:
- Create at least two partitions:
  - `/boot` – standard ext4
  - A physical volume for LVM
- Inside LVM, create:
  - A root logical volume (`/`)
  - A swap logical volume
  - Optional: `/home`, `/var`, `/srv`

---

### 3. 👤 User & Group Configuration

- Set **hostname**: your login + `42` (e.g. `jdoe42`)
- Create a new user: `login42`
- Add the user to:
  - `sudo` group (admin access)
  - Custom group `user42`

```bash
sudo adduser login42
sudo usermod -aG sudo,user42 login42

### 4. 🔐 Password Policy (PAM)
Install PAM module:

bash
Copy
Edit
sudo apt install libpam-pwquality
Edit /etc/pam.d/common-password and add:

ruby
Copy
Edit
password requisite pam_pwquality.so retry=3 minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7
Set password aging policy:

bash
Copy
Edit
sudo chage -M 30 -m 2 -W 7 login42
### 5. ⚙️ Sudo Configuration
Edit with sudo visudo and apply:

bash
Copy
Edit
Defaults        passwd_tries=3
Defaults        badpass_message="Password is incorrect. Please try again."
Defaults        logfile="/var/log/sudo/sudo.log"
Defaults        log_input, log_output
Defaults        requiretty
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
Create log directory:

bash
Copy
Edit
sudo mkdir -p /var/log/sudo
sudo touch /var/log/sudo/sudo.log
sudo chmod 600 /var/log/sudo/sudo.log
6. 🔒 SSH Configuration
Edit /etc/ssh/sshd_config:

conf
Copy
Edit
Port 4242
PermitRootLogin no
Restart SSH:

bash
Copy
Edit
sudo systemctl restart ssh
7. 🔥 Firewall Configuration
Install and enable UFW:

bash
Copy
Edit
sudo apt install ufw
sudo ufw allow 4242/tcp
sudo ufw enable
### 8. 📊 Monitoring Script
Create /usr/local/bin/monitoring.sh:

bash
Copy
Edit
#!/bin/bash
ARCH=$(uname -a)
CPU=$(nproc)
MEM=$(free -m | awk '/Mem:/ {printf("%s/%sMB (%.2f%%)", $3, $2, $3/$2 * 100)}')
DISK=$(df -h / | awk 'NR==2 {print $3 "/" $2 " (" $5 ")"}')
BOOT=$(who -b | awk '{print $3, $4}')
LVM=$(lsblk | grep -q "lvm" && echo "yes" || echo "no")
TCP=$(ss -t | grep ESTAB | wc -l)
USER_LOGGED=$(users | wc -w)
IP=$(hostname -I | awk '{print $1}')
MAC=$(ip link show | awk '/ether/ {print $2}')
SUDO=$(journalctl _COMM=sudo | grep COMMAND | wc -l)

wall " 
#Architecture: $ARCH
#CPU physical: $CPU
#Memory Usage: $MEM
#Disk Usage: $DISK
#Last boot: $BOOT
#LVM use: $LVM
#TCP Connections: $TCP ESTABLISHED
#User log: $USER_LOGGED
#Network: IP $IP MAC $MAC
#Sudo: $SUDO cmds used
"
Make executable:

bash
Copy
Edit
sudo chmod +x /usr/local/bin/monitoring.sh
Schedule it every 10 mins:

bash
Copy
Edit
sudo crontab -e
Add:

cron
Copy
Edit
*/10 * * * * /usr/local/bin/monitoring.sh
### ✅ Final Checklist
 VM installed with proper partitioning and LVM

 Hostname set correctly

 User login42 with groups sudo, user42

 Password policy enforced via PAM

 Sudo logging and restrictions applied

 SSH configured on port 4242

 Firewall enabled with correct rules

 Monitoring script created and scheduled
