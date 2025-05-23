# How to configure NFS server/client


### Server

```bash
sudo apt-get update 
sudo apt-get install nfs-kernel-server 
```

```bash
sudo mkdir /nfs_share
sudo chown nobody:nogroup /nfs_share 
sudo chmod 777 /nfs_share 
```

Configure NFS share folder(`/etc/exports`)
```
/nfs_share 192.168.1.10(rw,sync,no_subtree_check)
```
192.168.1.10 - ip of NFS client

####export, enable and start NFS share
```
sudo exportfs -a
sudo systemctl enable nfs-kernel-server 
sudo systemctl start nfs-kernel-server
``` 

### Client

Mount the NFS Share on the Client Machine
```
sudo mkdir /mnt/nfs_share
sudo mount -t nfs 192.168.1.100:/nfs_share /mnt/nfs_share  
```

Configure the NFS Share to Auto-Mount at Boot Time(`/etc/fstab`)
```
	
192.168.1.100:/nfs_share /mnt/nfs_share nfs defaults 0 0
```

