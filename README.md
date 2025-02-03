
# SSL Connection Setup Between MySQL Server and Clients  

<p align="justify">
This guide provides steps to configure MySQL with SSL for secure connections. It covers generating SSL certificates, enforcing SSL for users, configuring Django with SSL, and verifying secure communication. Ideal for enhancing database security.
</p>  

<br>  

## Figure 1: SSL Connection Between MySQL Server and Clients  

<p align="justify">
The image illustrates an SSL connection setup between a MySQL server and two MySQL client servers. The MySQL server (IP: 203.0.113.22) connects securely to two MySQL client servers via SSL. These client servers (IP: 54.123.45.67 and 104.248.123.45) run Django and Nginx. The allowed ports include MySQL's default port (3306) on the MySQL server and ports 443 (HTTPS) and 80 (HTTP) on the client servers for Django. The diagram visually represents the secure communication setup between the database and its clients.
</p>  

<br>  

![figure1](https://github.com/waiyanphyoeoo/mysql-ssl-connection/blob/219d0ba2313097f3b05a1dc8b4db49f8ee646784/mysql-ssl-connection.png)

# Table of Contents  

1. [Check OpenSSL Version](#check-openssl-version)  
2. [MySQL Server Site](#mysql-server-site)  
3. [Generate SSL Certificates](#generate-ssl-certificates)  
   - [Generate CA Certificate](#generate-ca-certificate)  
   - [Generate Server Key and Certificate](#generate-server-key-and-certificate)  
   - [Generate Client Key and Certificate](#generate-client-key-and-certificate)  
   - [Set Correct Permissions](#set-correct-permissions)  
4. [Copy SSL Certificates to Server](#copy-ssl-certificates-to-server)  
5. [Edit MySQL Configuration](#edit-mysql-configuration)  
6. [Restart MySQL](#restart-mysql)  
7. [Verify SSL Status](#verify-ssl-status)  
8. [Create New User with SSL Requirements](#create-new-user-with-ssl-requirements)  
9. [Enforce SSL on User](#enforce-ssl-on-user)  
10. [Check SSL Requirement for User](#check-ssl-requirement-for-user)  
11. [Allow MySQL Port through UFW](#allow-mysql-port-through-ufw)  
12. [Check Active Connections with SSL](#check-active-connections-with-ssl)  
13. [Install mysqlclient for Python](#install-mysqlclient-for-python)  
14. [Django Database Configuration](#django-database-configuration)  
15. [Set Permissions for SSL Certificates](#set-permissions-for-ssl-certificates)  
16. [Test MySQL Connection with SSL](#test-mysql-connection-with-ssl)  
17. [Grant Privileges for User](#grant-privileges-for-user)  

## Check OpenSSL Version  
To check your OpenSSL version, run:  
```
openssl version
```

## MySQL Server Site  
Check if SSL is enabled in MySQL by running:  
```
SHOW VARIABLES LIKE '%ssl%';
```

## Generate SSL Certificates  

### Generate CA Certificate  
Generate the Certificate Authority (CA) certificate:
```
mkdir /etc/mysql/ssl
cd /etc/mysql/ssl
openssl genrsa 2048 > ca-key.pem
openssl req -new -x509 -nodes -days 3650 -key ca-key.pem -out ca-cert.pem
```

### Generate Server Key and Certificate  
Generate the server key and certificate:
```
openssl req -newkey rsa:2048 -days 3650 -nodes -keyout server-key.pem -out server-req.pem
openssl rsa -in server-key.pem -out server-key.pem
openssl x509 -req -in server-req.pem -days 3650 -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem
```

### Generate Client Key and Certificate  
Generate the client key and certificate:
```
openssl req -newkey rsa:2048 -days 3650 -nodes -keyout client-key.pem -out client-req.pem
openssl rsa -in client-key.pem -out client-key.pem
openssl x509 -req -in client-req.pem -days 3650 -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem
```

### Set Correct Permissions  
Ensure the certificates have the correct permissions:
```
sudo chown mysql:mysql /etc/mysql/ssl/*.pem
chmod 600 *.pem
```

## Copy SSL Certificates to Server  
Transfer the SSL certificates to the MySQL server:
```
scp /etc/mysql/ssl/ca-cert.pem root@203.0.113.22:/etc/mysql/ssl/
scp /etc/mysql/ssl/client-cert.pem root@203.0.113.22:/etc/mysql/ssl/
scp /etc/mysql/ssl/client-key.pem root@203.0.113.22:/etc/mysql/ssl/
```

## Edit MySQL Configuration  
Edit the MySQL configuration file to enable SSL:
```
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
Add the following under `[mysqld]`:
```
[mysqld]
ssl-ca = /etc/mysql/ssl/ca-cert.pem
ssl-cert = /etc/mysql/ssl/server-cert.pem
ssl-key = /etc/mysql/ssl/server-key.pem

require_secure-transport = ON
bind-address = 0.0.0.0
```

## Restart MySQL  
Restart the MySQL service to apply the changes:
```
sudo systemctl restart mysql
```

## Verify SSL Status  
To check the SSL status in MySQL, run:
```
SHOW VARIABLES LIKE '%ssl%';
```

## Create New User with SSL Requirements  
Create a new user with SSL requirements for secure connections:
```
CREATE USER 'export-server'@'203.0.113.22' IDENTIFIED BY 'securepassword';
GRANT ALL PRIVILEGES ON database_name.* TO 'export-server'@'203.0.113.22';
FLUSH PRIVILEGES;
EXIT;
```

## Enforce SSL on User  
Enforce SSL for the user:
```
ALTER USER 'export-server'@'203.0.113.22' REQUIRE SSL;
```

## Check SSL Requirement for User  
Verify SSL requirements for the user:
```
SELECT user, host, ssl_type 
FROM mysql.user
WHERE ssl_type = 'ANY' OR ssl_type != '';
```

## Allow MySQL Port through UFW  
Allow MySQL traffic through the firewall by running:
```
sudo ufw allow from <client_server_ip> to any port 3306
```

## Check Active Connections with SSL  
To check the active connections using SSL, run:
```
SHOW PROCESSLIST;
```

## Install mysqlclient for Python  
Install the MySQL client for Python to connect using SSL:
```
pip install mysqlclient
```

## Django Database Configuration  
Configure Django to connect to MySQL using SSL by editing the `settings.py` file:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'database_name',
        'USER': 'username',
        'PASSWORD': 'password',
        'HOST': '203.0.113.22',  # MySQL server's public IP
        'PORT': '3306',  # Default MySQL port
        'OPTIONS': {
            'init_command': "SET sql_mode='STRICT_TRANS_TABLES'",
            'ssl': {
                'ca': '/etc/mysql/ssl/ca-cert.pem',  # Path to CA certificate
                'cert': '/etc/mysql/ssl/client-cert.pem',  # Path to client certificate
                'key': '/etc/mysql/ssl/client-key.pem',  # Path to client key
                'check_hostname': False,  # This disables hostname checking
            }
        },
    },
}
```

## Set Permissions for SSL Certificates  
Ensure that the certificates have proper permissions:
```
sudo chown devops:devops /etc/mysql/ssl/*
chmod 600 /etc/mysql/ssl/*.pem
```

## Test MySQL Connection with SSL  
Test the MySQL connection to the server using SSL:
```
mysql -h 203.0.113.22 -u export-server -p --ssl-mode=VERIFY_CA --ssl-ca=/etc/mysql/ssl/ca-cert.pem
SHOW STATUS LIKE 'Ssl_cipher';
```

## Grant Privileges for User  
Grant privileges to a user while enforcing SSL:
```
GRANT ALL PRIVILEGES ON database_name.* TO 'user_name'@'client_ip' IDENTIFIED BY 'password';
```

---

### Conclusion  
This guide provides steps to configure MySQL for secure SSL-based connections, including generating certificates, setting up secure user access, and integrating with Django. Follow these steps to ensure a secure MySQL database setup and SSL encryption for your connections.
