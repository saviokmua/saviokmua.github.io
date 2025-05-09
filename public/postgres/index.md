# Postgres


### Install PostgreSql

```bash
sudo apt install postgresql
```

### Create new Role
```sql
create role user;
```
### Show all roles
```sql
\du
```

### Change attributes on existing roles
```sql
ALETR ROLE "user" WITH LOGIN CREATEDB;
```

### Change role password

### Show all database
```sql
ALTER ROLE "user" WITH PASSWORD 'password';
```


### Set password for postgres user
```bash
sudo -u postgres psql
```
and
```sql
\password
```

### Enable remote access to Postgres
1. Modify the PostgreSQL configuration file
```
sudo nano /etc/postgresql/12/main/postgresql.conf
```
Then, find the line `#listen_addresses = 'localhost'` and uncomment it 
next, change the value of `listen_addresses` to `*`.

2. Modify the pg_hba.conf file

```
sudo nano /etc/postgresql/12/main/pg_hba.conf
```
append next line
```
host    all             all             0.0.0.0/0            md5
```
3. Restart PostgreSQL
```
sudo service postgresql restart
```

### Connect to remove postgres server
```
psql postgres://user:password@ip_add_or_domain:port/db_name
```
or
```
psql -h <REMOTE HOST> -p <REMOTE PORT> -U <DB_USER> <DB_NAME> -W
```
`-W` option will prompt for password

check port
```
# sudo netstat -peanut | grep "postgres"
tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN      123        614086     92337/postgres      
tcp6       0      0 :::5432                 :::*                    LISTEN      123        614087     92337/postgres      
udp        0      0 127.0.0.1:41721         127.0.0.1:41721         ESTABLISHED 123        614090     92337/postgres  
```
Change PostgreSQL user password
```sql
ALTER USER user_name WITH PASSWORD 'new_password';
```

