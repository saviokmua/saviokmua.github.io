# How to install specific Mysql version


For example, we should install Mysql version 8.0.34

1) Check version
```bash
apt-cache policy mysql-server

mysql-server:
  Installed: 8.0.34-0ubuntu0.20.04.1
  Candidate: 8.0.34-0ubuntu0.20.04.1
  Version table:
 *** 8.0.34-0ubuntu0.20.04.1 500
        500 http://ua.archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages
        500 http://ua.archive.ubuntu.com/ubuntu focal-security/main amd64 Packages
        100 /var/lib/dpkg/status
     8.0.19-0ubuntu5 500
        500 http://ua.archive.ubuntu.com/ubuntu focal/main amd64 Packages
```
2) Install mysql of specific version 
```bash
sudo apt install mysql-server=8.0.34-0ubuntu0.20.04.1
```

