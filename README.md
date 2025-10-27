# Server-Setup

This guide documents my personal setup process for building a simple yet secure home or remote server — starting from mounting drives to configuring SSH and network protection tools like **fail2ban** and **UFW**.

---

## 📂 Mounting NTFS Drives on Boot

Add this line to your `/etc/fstab` to automatically mount your NTFS drive at boot:

```bash
UUID=PARTITION-UID(see using lsblk -f) /mnt/Files ntfs-3g defaults,uid=1000,gid=1000,umask=022 0 0
```

---

## 🔐 Starting with SSH

All you need is a base system and **OpenSSH** to start.  
Remember, SSH follows a **server–client model**.

### Installation

For **Ubuntu**:
```bash
sudo apt install openssh
```

For **Void Linux**:
```bash
sudo xbps-install -Syy openssh
```

### Starting the SSH Service

For **Ubuntu (Systemd)**:
```bash
sudo systemctl enable --now ssh
```

For **Void Linux (Runit)**:
```bash
sudo ln -s /etc/sv/ssh /var/service/sshd
```

### Hardening SSH Configuration

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

### Copy your public key to the server
```bash
ssh-copy-id -i ~/.ssh/filename username@192.168.0.X
```

### Connect to the server
```bash
ssh -p 2222 username@192.168.0.X
```

### Simplify with SSH Config

You can make connections effortless by creating a config file at `~/.ssh/config`:

```ini
Host servername
    HostName 192.168.0.X
    IdentityFile ~/.ssh/filename
    User Tanvir
    Port 2222
```

---

## 🌐 Tailscale — Access From Anywhere

**Tailscale** is a mesh VPN that makes your SSH server accessible **from anywhere with internet**.  
It assigns private static IPs to devices, allowing secure and direct communication without port forwarding.

---

## 🛡️ fail2ban — Brute Force Protection

Create or edit the jail config for SSH at `/etc/fail2ban/jail.d/sshd.conf`:

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
sudo ufw allow ssh         # Allow default SSH port 22
sudo ufw allow 80/tcp      # HTTP traffic
sudo ufw allow 443/tcp     # HTTPS traffic
sudo ufw limit 2222/tcp    # Rate-limits connections to port 2222, good to prevent bruteforce attacks
```

Enable UFW:
```bash
sudo ufw enable
sudo ufw status verbose
```

---

### Final Thoughts

This setup forms the foundation of a secure, functional server environment.  
From here, you can build on top of it — hosting websites, automating backups, or experimenting with network services.
