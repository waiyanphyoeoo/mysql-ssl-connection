# SSL Connection Setup Between MySQL Server and Clients
This guide provides steps to configure MySQL with SSL for secure connections. It covers generating SSL certificates, enforcing SSL for users, configuring Django with SSL, and verifying secure communication. Ideal for enhancing database security.



# Figure 1: SSL Connection Between MySQL Server and Clients

The image illustrates an SSL connection setup between a MySQL server and two MySQL client servers. The MySQL server (IP: 203.0.113.22) connects securely to two MySQL client servers via SSL. These client servers (IP: 54.123.45.67 and 104.248.123.45) run Django and Nginx. The allowed ports include MySQL's default port (3306) on the MySQL server and ports 443 (HTTPS) and 80 (HTTP) on the client servers for Django. The diagram visually represents the secure communication setup between the database and its clients
![figure1](https://github.com/waiyanphyoeoo/mysql-ssl-connection/blob/219d0ba2313097f3b05a1dc8b4db49f8ee646784/mysql-ssl-connection.png)
