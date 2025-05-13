# Install and configure Mysql Server




## Install MySql Server
```bash
sudo apt install mysql-server
```

## Configure MySql Server

### Open Mysql console
```bash
sudo mysql
```

### Set root password
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
### Allow use default authentication method
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;
```
### Run the security script
```bash
sudo mysql_secure_installation
```
### Creating a MySql User
```sql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```
### Grant the appropriate privileges
```sql
GRANT ALL PRIVILEGES ON database.table TO 'username'@'host';
```
### Flush privileges
```sql
FLUSH PRIVILEGES;
```

## Check Mysql

```bash
systemctl status mysql.service
``` 



