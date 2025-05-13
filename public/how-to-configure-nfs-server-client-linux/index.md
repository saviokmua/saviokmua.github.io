# How to Configure NFS Server and Client on Linux


# 📁 How to Configure NFS Server and Client on Linux

NFS (Network File System) is a protocol that allows file sharing between systems over a network. This guide shows you how to configure an **NFS server** and a **client** on Ubuntu-based systems.

---

## 🖥️ NFS Server Setup

### 1. Install NFS server

```bash
sudo apt-get update
sudo apt-get install nfs-kernel-server
```

### 2. Create shared directory

```bash
sudo mkdir /nfs_share
sudo chown nobody:nogroup /nfs_share
sudo chmod 777 /nfs_share
```

> You can replace `/nfs_share` with any folder you want to share.

### 3. Export the directory

Edit the file `/etc/exports` and add:

```bash
/nfs_share 192.168.1.10(rw,sync,no_subtree_check)
```

📌 `192.168.1.10` — IP address of the NFS client.

### 4. Export and start the NFS service

```bash
sudo exportfs -a
sudo systemctl enable nfs-kernel-server
sudo systemctl start nfs-kernel-server
```

---

## 💻 NFS Client Setup

### 1. Create mount point

```bash
sudo mkdir /mnt/nfs_share
```

### 2. Mount the NFS share

```bash
sudo mount -t nfs 192.168.1.100:/nfs_share /mnt/nfs_share  
```

📌 Replace `192.168.1.100` with the IP of your NFS server.

### 3. Auto-mount on boot

Add the following line to `/etc/fstab`:

```bash
192.168.1.100:/nfs_share /mnt/nfs_share nfs defaults 0 0
```

---

## 🛠️ Useful Tips

- Use `showmount -e 192.168.1.100` on the client to list exported NFS directories.
- Check NFS service logs with `journalctl -u nfs-server` for troubleshooting.
- Make sure the firewall allows NFS-related ports (e.g., 2049/tcp).

---
