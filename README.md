# SSL Connection Setup Between MySQL Server and Clients  

<p align="justify">
This guide provides steps to configure MySQL with SSL for secure connections. It covers generating SSL certificates, enforcing SSL for users, configuring Django with SSL, and verifying secure communication. Ideal for enhancing database security.
</p>  

<br>  

# Figure 1: SSL Connection Between MySQL Server and Clients  

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

#check-openssl-version
Checking OPEN SSl
