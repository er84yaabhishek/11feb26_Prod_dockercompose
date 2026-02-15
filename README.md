# Code Server Setup (Docker + Apache + SSL)

This project runs **code-server (VS Code in Browser)** using Docker and exposes it securely using Apache Reverse Proxy and Let's Encrypt SSL.

**Subdomain:** editor.soft84ya.shop

---

## 1. Docker Configuration

**File:** `docker-compose.editor.yml`

```yaml
version: "3.8"

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
```

Start container:

```bash
docker-compose -f docker-compose.editor.yml up -d --build
```

---

## 2. Apache HTTP Configuration

**File:** `/etc/apache2/sites-available/editor.soft84ya.shop.conf`

```apache
<VirtualHost *:80>
    ServerName editor.soft84ya.shop
    Redirect permanent / https://editor.soft84ya.shop/
</VirtualHost>
```

---

## 3. Apache HTTPS + SSL Configuration

**File:** `/etc/apache2/sites-available/editor.soft84ya.shop-le-ssl.conf`

```apache
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
```

---

## 4. Enable Required Apache Modules

```bash
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_wstunnel
sudo a2enmod rewrite
sudo a2enmod ssl
```

---

## 5. Enable Site

```bash
sudo a2ensite editor.soft84ya.shop.conf
sudo a2ensite editor.soft84ya.shop-le-ssl.conf
sudo systemctl reload apache2
```

---

## 6. Generate SSL Certificate

```bash
sudo certbot --apache -d editor.soft84ya.shop
```

---

## Access

https://editor.soft84ya.shop

---

## Security Recommendation

- Change default passwords before production use
- Use strong credentials
- Keep Docker images updated
