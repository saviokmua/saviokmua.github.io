# Basic Auth in Nginx


```
server {
    server_name site.com;
    location / {
        ...
	      auth_basic           "Administrator Area";
        auth_basic_user_file /etc/nginx/protections/site.com.htpasswd;
        ...
    }
}
```
Install
```
sudo apt-get install apache2-utils
```
and create new password file and set password for user admin
```
htpasswd -c site.com.htpasswd admin
```


