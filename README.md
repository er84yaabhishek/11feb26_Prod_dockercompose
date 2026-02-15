docker-compose.editor.yml
---
services:
  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - PASSWORD=12345678
      - SUDO_PASSWORD=12345678
      - DEFAULT_WORKSPACE=/config/workspace
      - PWA_APPNAME=code-server
    volumes:
      - ./app_Staging:/config/workspace
    ports:
      - "8079:8443"
    restart: unless-stopped
=====================================================================
editor.soft84ya.shop-le-ssl.conf
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName editor.soft84ya.shop

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/editor.soft84ya.shop/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/editor.soft84ya.shop/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf
    
 # Preserve the original host header
    ProxyPreserveHost On
    AllowEncodedSlashes NoDecode

    # Standard HTTP Reverse Proxy
    ProxyPass / http://localhost:8079/ nocanon
    ProxyPassReverse / http://localhost:8079/

    # WebSocket Reverse Proxy
    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} =websocket [NC]
    RewriteRule /(.*) ws://localhost:8079/$1 [P,L]
    RewriteCond %{HTTP:Upgrade} !=websocket [NC]
    RewriteRule /(.*) http://localhost:8079/$1 [P,L]

</VirtualHost>
</IfModule>
=========================================================================
editor.soft84ya.shop.conf
<VirtualHost *:80>
    ServerName editor.soft84ya.shop

    Redirect permanent / https://editor.soft84ya.shop/
</VirtualHost>



































    


