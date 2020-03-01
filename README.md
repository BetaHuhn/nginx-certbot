# nginx-nodejs-config-file
A nginx configuration file for Node.js Apps

## Certbot command
`sudo certbot certonly --nginx -d DOMAIN`

## Nginx config file
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

  # certs sent to the client in SERVER HELLO are concatenated in ssl_certificate
  ssl_certificate /etc/letsencrypt/live/*DOMAIN*/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/*DOMAIN*/privkey.pem;
  ssl_session_timeout 1d;
  ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
  ssl_session_tickets off;

  # curl https://ssl-config.mozilla.org/ffdhe2048.txt > /path/to/dhparam.pem
  ssl_dhparam /etc/letsencrypt/dhparam.pem;

  # intermediate configuration
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
  ssl_prefer_server_ciphers on;

  # HSTS (ngx_http_headers_module is required) (63072000 seconds)
  add_header Strict-Transport-Security "max-age=63072000" always;

  # OCSP stapling
  ssl_stapling on;
  ssl_stapling_verify on;

  # verify chain of trust of OCSP response using Root CA and Intermediate certs
  ssl_trusted_certificate /etc/letsencrypt/live/*DOMAIN*/chain.pem;

  # replace with the IP address of your resolver
  resolver 8.8.8.8;

  error_page 502 /public/error/maintenance/index.html;
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

  location /public/error/maintenance/index.html{

  }
}
```
