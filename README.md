# nginx-certbot
A nginx configuration file to use with Let's Encrypt HTTPS certificates.

## Certbot
### Install Certbot

Add repository:

`sudo add-apt-repository ppa:certbot/certbot`

Update package list:

`sudo apt update`

Install certbot nginx package:

`sudo apt-get install python-certbot-nginx`

### Certbot commands

Generate certificate for one or more domains:

`sudo certbot certonly --nginx -d domain.com -d www.domain.com`

Generate wildcard certificate:

`sudo certbot certonly --manual -d *.domain.com -d domain.com --preferred-challenges dns`

> Note: You will need to add two TXT records in your DNS for the acme-challenge

Renew all certificates:

`sudo certbot renew`

> Note: Only works for none wildcard certificates

## Firewall

View current settings:

`sudo ufw status`

Run:

`sudo ufw allow 'Nginx Full'`

and

`sudo ufw delete allow 'Nginx HTTP'`

## Nginx

### Config file

```
server {
  listen 80;
  root *PATH TO ROOT FOLDER*;
  server_name *DOMAIN*;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  client_max_body_size 5000M;
  root *PATH TO ROOT FOLDER*;
  server_name *DOMAIN*;

  ssl_certificate /etc/letsencrypt/live/*DOMAIN*/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/*DOMAIN*/privkey.pem;
  ssl_session_timeout 1d;
  ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
  ssl_session_tickets off;

  ssl_dhparam /etc/letsencrypt/dhparam.pem;

  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
  ssl_prefer_server_ciphers on;

  add_header Strict-Transport-Security "max-age=63072000" always;

  ssl_stapling on;
  ssl_stapling_verify on;

  ssl_trusted_certificate /etc/letsencrypt/live/*DOMAIN*/chain.pem;

  resolver 8.8.8.8;

  location / {
    proxy_pass http://localhost:*PORT NUMBER*;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';  
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_cache_bypass $http_upgrade;
    proxy_intercept_errors on;
    proxy_pass_request_headers on;
  }
}
```

### Commands

Verify config:

`sudo nginx -t`

Restart nginx:

`sudo service restart nginx`
