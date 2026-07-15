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

```bash
git config --global credential.helper store
```

# 1. React (Static Build – NGINX Only) ⭐ Recommended

### Build

```bash
npm run build
```

### NGINX
Save with Ctrl+O, Enter, then Ctrl+X.

```nginx
server {
    server_name domain.com www.domain.com;
    root /var/www/react-app/build;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

}
```

# 2. React (PM2 Static Server + NGINX Proxy)

### PM2

```bash
pm2 serve ./build 3017 --spa --name projectName
```

### NGINX

```nginx
server {
    server_name domain.com www.domain.com;
    location / {
        include proxy_params;
        proxy_pass http://localhost:3017;
    }
}
```

# 3. Node.js + Express (API Server)

### PM2

```bash
pm2 start server.js --name api-server
pm2 start server.js --name api-server -- run prod
```
### runs with ecosystem.config.js
```javascript
module.exports = {
  apps: [
    {
      name: "api_server_name",
      script: "server.js",
      env: {
        NODE_ENV: "production",
        PORT: 9000,
      },
      description: "",
    },
  ],
};
```
```bash
pm2 start ecosystem.config.js
```

### NGINX

```nginx
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
```

# 4. Node.js + Express + WebSocket (Socket.IO / WS)

### PM2

```bash
pm2 start server.js --name socket-server
```

### NGINX (WebSocket Enabled)

```nginx
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
```

# 5. Next.js (No WebSocket)

### PM2

```bash
pm2 start npm --name next-app -- start
pm2 start npm --name next-app -- start -- -p 3010
```

### NGINX

```nginx
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
```

# 6. Next.js + WebSocket

### PM2

```bash
pm2 start npm --name next-ws -- start
pm2 start npm --name next-ws -- start -- -p 3010
```

### NGINX (WebSocket Ready)

```nginx
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
```

# Common Commands

```bash
nginx -t
systemctl reload nginx
pm2 list
pm2 logs
pm2 save
pm2 startup
pm2 start ecosystem.config.js --only food_delivery_burgerbros
pm2 start ecosystem.config.js --only food_delivery_grilllab,food_delivery_burgerbros
```

# Certbot Commands

```bash
sudo certbot --nginx -d domain.com -d www.domain.com
```

# Directory/Files Commands

## Delete a file
```bash
rm /path/to/your/file
```
## Delete a directory- Empty
```bash
rmdir /path/to/directory
```
## Delete a directory- including all files and subfolders
```bash
rm -rf folder_name
```


