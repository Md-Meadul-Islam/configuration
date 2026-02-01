# NGINX + PM2 Deployment Reference

Reusable NGINX configurations for common project types  
(React, Node, Next.js, WebSocket, PM2)

---

## Global Notes

- **PM2** → process manager (Node / static server)
- **NGINX** → reverse proxy, SSL, caching
- **Certbot** manages SSL blocks
- Always keep **NGINX port ↔ PM2 port consistent**

---

# Git – Store Credentials Permanently

git config --global credential.helper store

## 1. React (Static Build – NGINX Only) ⭐ Recommended

# Build

npm run build

# NGINX

server {
server_name domain.com www.domain.com;
root /var/www/react-app/build;
index index.html;

    location / {
        try_files $uri /index.html;
    }

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

}

## 2. React (PM2 Static Server + NGINX Proxy)

# PM2

pm2 serve ./build 3017 --spa --name projectName

# NGINX

server {
server_name domain.com www.domain.com;

    location / {
        include proxy_params;
        proxy_pass http://localhost:3017;
    }

}

## 3. Node.js + Express (API Server)

# PM2

pm2 start server.js --name api-server

# NGINX

server {
server_name api.domain.com www.api.domain.com;

    location / {
        proxy_pass http://localhost:4000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

}

## 4. Node.js + Express + WebSocket (Socket.IO / WS)

# PM2

pm2 start server.js --name socket-server

# NGINX (WebSocket Enabled)

server {
server_name socket.domain.com www.socket.domain.com;

    location / {
        proxy_pass http://localhost:4000;
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

}

## 5. Next.js (No WebSocket)

# PM2

pm2 start npm --name next-app -- start

# NGINX

server {
server_name domain.com www.domain.com;

    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Next.js static files with long cache
    location /_next/static/ {
        proxy_pass http://localhost:3000;
        add_header Cache-Control "public, max-age=31536000, immutable";
    }
    # Next.js image optimization
    location /_next/image {
        proxy_pass http://localhost:3000;
        add_header Cache-Control "public, max-age=31536000, immutable";
    }
    # All other routes
    location / {
        proxy_pass http://localhost:3000;
    }

}

## 6. Next.js + WebSocket

# PM2

pm2 start npm --name next-ws -- start

# NGINX (WebSocket Ready)

server {
server_name domain.com www.domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

}

# Common Commands

nginx -t
systemctl reload nginx
pm2 list
pm2 logs
pm2 save
pm2 startup
