# Server-Setup

This guide documents my personal setup process for building a simple yet secure home or remote server — starting from mounting drives to configuring SSH and network protection tools like **fail2ban** and **UFW**.

---

## 🔄 First make sure the system is up to date
```bash
# Ubuntu/Debian
sudo apt update && sudo apt upgrade -y

# Void Linux
sudo xbps-install -Syy
```

Note: Some updates may require a reboot after completion.

## 📂 Mounting NTFS Drives on Boot

Add this line to your `/etc/fstab` to automatically mount your NTFS drive at boot:

```bash
UUID=PARTITION-UID /mnt/Files ntfs-3g defaults,uid=1000,gid=1000,umask=022 0 0
```

To see the partition uid:
``` bash
lsblk -f
```
---

## 🔐 Starting with SSH

All you need is a base system and **OpenSSH** to start.  
Remember, SSH follows a **server–client model**.

### Installation

```bash
# Ubuntu/Debian
sudo apt install openssh

# Void Linux
sudo xbps-install -S openssh
```

### Starting the SSH Service:
```bash
# For systemd (Ubuntu, Debian, Fedora etc)
sudo systemctl enable --now ssh

# For runit (Void Linux, Artix etc)
sudo ln -s /etc/sv/ssh /var/service/
```

### Hardening SSH Configuration

It's a good idea to backup the default config file before making any changes, make the backup in this case by:
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

Edit your SSH daemon config at `/etc/ssh/sshd_config`:

```ini
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
KbdInteractiveAuthentication no
```

---

## 🧩 Setting Up SSH Keys (Client-Side)

SSH keys are the backbone of secure connections.  
It’s good practice to use **unique keys for different servers** — like using different keys for different doors.

### Generate a new SSH key
```bash
ssh-keygen -f ~/.ssh/filename -C "comment"
```
Sometimes users run into “UNPROTECTED PRIVATE KEY FILE” errors. You can simply fix it by:
```bash
chmod 600 ~/.ssh/filename
```

### Find Server IP (run this on the server)
```bash
ip addr show
```

### Copy your public key to the server
```bash
ssh-copy-id -i ~/.ssh/filename username@<IP_ADDRESS_or_HOSTNAME>
```

### Connect to the server
```bash
ssh -p 2222 username@<IP_ADDRESS_or_HOSTNAME>
```

### Simplify with SSH Config

You can make connections effortless by creating a config file at `~/.ssh/config`:

```ini
Host servername
    HostName <IP_ADDRESS_or_HOSTNAME>
    IdentityFile ~/.ssh/filename
    User Username
    Port 2222
```

---

## 🌐 Tailscale — Access From Anywhere

**Tailscale** is a mesh VPN that makes your SSH server accessible **from anywhere with internet**.  
It assigns private static IPs to devices, allowing secure and direct communication without port forwarding.

First, install tailscale in Ubuntu by: 
```bash
# Ubuntu
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list

sudo apt-get update
sudo apt-get install tailscale

# Void Linux
sudo xbps-install -S tailscale
```

Enable Tailscale on boot:
```bash
# For systemd
sudo systemctl enable --now tailscaled

# For runit
sudo ln -s /etc/sv/tailscaled /var/service/
```

To get private static ip with tailscale:
```bash
tailscale up
```
It will show or open a link in your browser. Go there and sign in with your account to get the private IP. Connect your other devices with the same account to establish connection between devices securely.

To get tailscale ip:
```bash
tailscale ip
```

---

## 🛡️ fail2ban — Brute Force Protection

To install:
```bash
# Ubuntu/Debian
sudo apt install fail2ban

# Void Linux
sudo xbps-install -S fail2ban
```

Start the service by:
```bash
# For systemd
sudo systemctl enable --now fail2ban

# For runit
sudo ln -s /etc/sv/fail2ban /var/service/
```

Create or edit the jail config at `/etc/fail2ban/jail.d/sshd.conf`:

```ini
[sshd]
enabled  = true
port     = 2222
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 5
bantime  = 1h
findtime = 10m
```

---

## 🔥 UFW (Uncomplicated Firewall)

Start with a safe default configuration:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Then allow essential ports:

```bash
sudo ufw allow ssh         # AIf you’ve changed your SSH port (e.g. 2222), you don’t need to allow port 22.
sudo ufw allow 80/tcp      # HTTP traffic
sudo ufw allow 443/tcp     # HTTPS traffic
sudo ufw limit 2222/tcp    # Rate-limits connections to port 2222, good to prevent bruteforce attacks
```

Enable UFW:
```bash
sudo ufw enable
```

To verify UFW status and active rules:
```bash
sudo ufw status verbose
```

## 💾 Configure Zram (Optional)
Zram creates compressed swap space in RAM instead of using slow disk storage. It's much faster than traditional swap and doesn't use your SSD/HDD.

To install zram:
```bash
# Ubuntu/Debian
sudo apt install zram-tools

# Void Linux
sudo xbps-install -S zramen
```

To start the service simply:
```bash
# For systemd
sudo systemctl enable --now zram-config

# For runit
sudo ln -s /etc/sv/zramen /var/service/
```

---

### Final Thoughts

This setup forms the very basic setup for of a secure, functional server environment. My plan it to make it even more useful by using docker or making it a VPN or something similar, rather than than just a server to backup my files.
