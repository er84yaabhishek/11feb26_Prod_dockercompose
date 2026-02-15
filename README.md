🚀 Code Server Setup (Docker + Apache + SSL)

This project runs code-server (VS Code in Browser) using Docker and exposes it securely via:

Docker Compose

Apache Reverse Proxy

Let's Encrypt SSL

Subdomain: editor.soft84ya.shop

📦 1️⃣ Docker Setup
📁 File: docker-compose.editor.yml
version: "3.8"

services:
  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - PASSWORD=1234
      - SUDO_PASSWORD=1234
      - DEFAULT_WORKSPACE=/config/workspace
      - PWA_APPNAME=code-server
    volumes:
      - ./app_Staging:/config/workspace
    ports:
      - "8079:8443"
    restart: unless-stopped

▶ Start Container
docker-compose -f docker-compose.editor.yml up -d --build

🌐 2️⃣ Apache Reverse Proxy Setup
📁 HTTP Config

File: /etc/apache2/sites-available/editor.soft84ya.shop.conf

<VirtualHost *:80>
    ServerName editor.soft84ya.shop
    Redirect permanent / https://editor.soft84ya.shop/
</VirtualHost>

🔐 HTTPS + SSL Config

File: /etc/apache2/sites-available/editor.soft84ya.shop-le-ssl.conf

<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName editor.soft84ya.shop

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/editor.soft84ya.shop/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/editor.soft84ya.shop/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf

    ProxyPreserveHost On
    AllowEncodedSlashes NoDecode

    ProxyPass / http://localhost:8079/ nocanon
    ProxyPassReverse / http://localhost:8079/

    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} =websocket [NC]
    RewriteRule /(.*) ws://localhost:8079/$1 [P,L]
    RewriteCond %{HTTP:Upgrade} !=websocket [NC]
    RewriteRule /(.*) http://localhost:8079/$1 [P,L]

</VirtualHost>
</IfModule>

🔧 3️⃣ Enable Required Apache Modules
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_wstunnel
sudo a2enmod rewrite
sudo a2enmod ssl

📜 4️⃣ Enable Site
sudo a2ensite editor.soft84ya.shop.conf
sudo a2ensite editor.soft84ya.shop-le-ssl.conf
sudo systemctl reload apache2

🔐 5️⃣ Generate SSL Certificate
sudo certbot --apache -d editor.soft84ya.shop

🌍 Access URL
https://editor.soft84ya.shop


Login using the password defined in docker-compose.editor.yml.

