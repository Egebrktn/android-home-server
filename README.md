# Android Home Server

This repository documents how to turn an **Android device** into a low-power,
always-on **home server** using a Debian Linux chroot environment.

No public IP or port forwarding is required.

---

## Table of Contents

- Overview
- Requirements
- Debian Chroot Setup
- Navidrome Music Server
- Cloudflare Tunnel
- Notes
- License

---

## Overview

Modern Android devices are powerful, energy-efficient and always connected.
This project shows how to repurpose an old or spare Android phone as a personal
home server.

Features:
- Debian Linux via real chroot (not proot)
- Navidrome music streaming server
- Cloudflare Tunnel for external access
- Low power consumption

---

## Requirements

- Rooted Android device
- Termux (F-Droid version recommended)
- Working `su` (Magisk or KernelSU)
- At least 4 GB free storage
- Stable internet connection

---

## Debian Chroot Setup

### Install Required Packages

Open Termux and run:

```bash
pkg update && pkg upgrade -y
pkg install wget tar -y
```

Check root access:

```bash
su
```

If you get a root shell, continue.

---

### Download Debian RootFS

This setup uses a minimal Debian 13 (Trixie) root filesystem provided as a compressed `rootfs.tar.xz` archive.

```bash
cd /data/local/tmp
wget <DEBIAN_ROOTFS_URL>
```

Replace `<DEBIAN_ROOTFS_URL>` with a Debian 13 (Trixie) arm64 rootfs archive.

Choose the correct architecture:
- arm64 → most modern devices
- armhf → older devices

---

### Extract RootFS

```bash
mkdir -p /data/debian
tar -xpf rootfs.tar.xz -C /data/debian
```

---

### Create start script

Create a file named `start.sh`:

```bash
nano start.sh
```

Paste the following content:

```bash
#!/data/data/com.termux/files/usr/bin/bash

DEBIAN_DIR=$HOME/debian

echo "[*] Mounting filesystems..."

mount -o bind /dev $DEBIAN_DIR/dev
mount -t proc proc $DEBIAN_DIR/proc
mount -t sysfs sys $DEBIAN_DIR/sys
mount -o bind /sdcard $DEBIAN_DIR/sdcard

echo "nameserver 1.1.1.1" > $DEBIAN_DIR/etc/resolv.conf

echo "[*] Entering Debian chroot..."
chroot $DEBIAN_DIR /bin/bash
```

Make the script executable:

```bash
chmod +x start.sh
```

---
### Start Debian

Run the script as root:

```bash
su
./start.sh
```

You are now inside the Debian chroot environment.

### Post Installation Setup

```bash
apt update
apt install sudo nano curl locales -y
```

Configure locale:

```bash
dpkg-reconfigure locales
```

Create a user:

```bash
adduser server
usermod -aG sudo server
su - server
```

---

## Navidrome Music Server

Navidrome is a lightweight, open-source music streaming server.

### Install Navidrome

```bash
curl -LO https://github.com/navidrome/navidrome/releases/latest/download/navidrome_linux_arm64.tar.gz
tar -xvf navidrome_linux_arm64.tar.gz
sudo mv navidrome /usr/local/bin/
```

Create directories:

```bash
mkdir -p ~/navidrome/music
mkdir -p ~/.config/navidrome
```

Create config file:

```bash
nano ~/.config/navidrome/navidrome.toml
```

Example configuration:

```toml
MusicFolder = "/home/server/navidrome/music"
Address = "0.0.0.0"
Port = "4533"
```

Start Navidrome:

```bash
navidrome
```

---

## Cloudflare Tunnel

Cloudflare Tunnel allows exposing services without opening ports.

### Install cloudflared

```bash
curl -LO https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64
chmod +x cloudflared-linux-arm64
sudo mv cloudflared-linux-arm64 /usr/local/bin/cloudflared
```

Authenticate:

```bash
cloudflared tunnel login
```

Create and run tunnel:

```bash
cloudflared tunnel create android-server
cloudflared tunnel run android-server
```

---

## Notes

- Disable battery optimizations for Termux
- Keep the device plugged in if possible
- Ideal for personal use and learning
- More services can be added later

---

## License

MIT License
