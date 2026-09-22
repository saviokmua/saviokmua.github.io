---
date: '2026-09-21'
lastmod: '2026-09-21'
draft: false
title: 'How to Set Up a Clean VPS on Debian 13: Hostname, User, SSH Keys, and Port'
slug: 'debian-13-vps-initial-setup'
authors: ['Alex']
enableReadingTime: true
description: "A step-by-step guide to securing a fresh Debian 13 VPS: setting the hostname, creating a sudo user, enabling key-only SSH and disabling root login, and changing the SSH port."
keywords: ['debian 13 vps setup', 'secure vps debian', 'disable root ssh login', 'ssh key only authentication', 'change ssh port', 'debian initial server setup']
tags: ['debian', 'vps', 'ssh', 'security', 'server']
categories: ['DevOps']
hiddenFromHomePage: false
featuredImage: ''
---

### How to Set Up a Clean VPS on Debian 13: Hostname, User, SSH Keys, and Port

---

You just got a fresh Debian 13 VPS and logged in as `root` with a password. Before installing anything else, let's lock the server down: set a proper hostname, create a non-root user with sudo rights, switch SSH to key-only authentication, disable root login, and move SSH off the default port.

## Step 1: Set the Hostname

Check the current hostname:

```bash
hostnamectl
```

Set a new one (replace `myserver` with whatever fits your naming scheme):

```bash
hostnamectl set-hostname myserver
```

Update `/etc/hosts` so the hostname resolves locally — open the file and make sure it has a line like this:

```
127.0.1.1   myserver
```

Verify the change:

```bash
hostnamectl
hostname -f
```

You may need to log out and back in for the new hostname to show up in your shell prompt.

---

## Step 2: Create a New User with Sudo Rights

Working as `root` for everyday tasks is risky. Create a dedicated user instead:

```bash
adduser deploy
```

You'll be prompted for a password and some optional info (full name, etc.) — fill in what you need, the rest can be left blank.

Install `sudo` if it's not already present, and add the user to the `sudo` group:

```bash
apt update
apt install sudo -y
usermod -aG sudo deploy
```

Switch to the new user and confirm sudo works:

```bash
su - deploy
sudo whoami
```

`sudo whoami` should print `root`. If it does, the user is correctly set up.

---

## Step 3: SSH Access — Keys Only, No Root Login

### Generate an SSH key pair (on your local machine)

If you don't already have one:

```bash
ssh-keygen -t ed25519 -C "deploy@myserver"
```

### Copy the public key to the server

```bash
ssh-copy-id deploy@your_server_ip
```

If `ssh-copy-id` isn't available, do it manually:

```bash
cat ~/.ssh/id_ed25519.pub | ssh deploy@your_server_ip \
  "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

### Test key-based login before changing anything else

```bash
ssh deploy@your_server_ip
```

Make sure you can log in **without** a password prompt before continuing — if key auth doesn't work yet, fix it now, not after you've disabled passwords.

### Disable password authentication and root login

Edit the SSH daemon config:

```bash
sudo nano /etc/ssh/sshd_config
```

Set (or uncomment and change) these directives:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

> [!NOTE]
> Debian 13 may split settings across `/etc/ssh/sshd_config.d/*.conf`. If a drop-in file overrides one of these directives, edit it there too — the last matching value wins.

Restart SSH to apply the changes:

```bash
sudo systemctl restart ssh
```

**Before closing your current session**, open a second terminal and test a fresh connection as `deploy` to confirm you still have access. Only close the original root session once the new one works.

---

## Step 4: Change the SSH Port

Running SSH on the default port 22 makes it a constant target for automated scans. Moving it to a non-standard port cuts down on that noise significantly.

Open the config again:

```bash
sudo nano /etc/ssh/sshd_config
```

Find (or add) the `Port` directive and set a custom port:

```
Port 2222
```

Pick any free port in the 1024–65535 range. Restart SSH:

```bash
sudo systemctl restart ssh
```

If you use `ufw`, allow the new port **before** you lose access, and only then remove the old one:

```bash
sudo ufw allow 2222/tcp
sudo ufw delete allow 22/tcp
sudo ufw status
```

Test the new port from a separate terminal, keeping your current session open until it's confirmed:

```bash
ssh -p 2222 deploy@your_server_ip
```

> [!NOTE]
> If your VPS provider has an external firewall (a security group in the cloud console, for example), open the new port there as well — `ufw` alone won't help if the provider's firewall blocks it first.

To avoid typing `-p 2222` every time, add an entry to your local `~/.ssh/config`:

```
Host myserver
    HostName your_server_ip
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

Now you can just run:

```bash
ssh myserver
```

---

## Summary

| Step | What you did |
|------|--------------|
| 1 | Set a proper hostname with `hostnamectl` |
| 2 | Created a non-root user and added it to `sudo` |
| 3 | Switched SSH to key-only auth and disabled root login |
| 4 | Moved SSH to a custom port and opened it in the firewall |

At this point you have a Debian 13 server with no password-based root access exposed on a predictable port — a solid baseline before installing anything else. Keep your session open until every change is verified from a second connection; that one habit prevents almost every "I locked myself out" story.
