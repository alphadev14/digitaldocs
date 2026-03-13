# 🚀 Lab 02: Deploy NextJS Frontend trên Ubuntu Server với Nginx + PM2 (Ubuntu)

**Server sử dụng**:

- Node.js -- chạy ứng dụng Next.js
- PM2 -- quản lý process Node chạy nền
- Nginx -- reverse proxy và expose domain ra internet

---

# Architecture

```bash
Internet
↓
Nginx (Reverse Proxy)
↓
Next.js App (PM2)
↓
Node.js (localhost:3000)
```

---

# 1. Update Server

```bash
sudo apt update sudo apt upgrade -y
```

---

# 2. Install Node.js

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x \| sudo -E bash -\
sudo apt install -y nodejs

# Check version:

node -v\
npm -v
```

---

# 3. Install PM2

```bash
sudo npm install -g pm2

# Check:

pm2 -v
```

---

# 4. Go to Next.js project

Example:

```bash
cd \~/tools/next-blog
```

---

# 5. Install dependencies

```bash
npm install
```

---

# 6. Build project

```bash
npm run build
```

---

# 7. Run Next.js with PM2

```bash
pm2 start npm --name next-blog -- start
# Check:
pm2 list
```

---

# 8. Test application

Next.js runs at:

```bash
http://localhost:3000
```

Test:

```bash
curl http://localhost:3000
```

---

# 9. Install Nginx

```bash
sudo apt install nginx -y
# Check:
nginx -v
```

---

# 10. Create Nginx config

```bash
sudo nano /etc/nginx/sites-available/dev.tampv.pro.com

Example config:

server { listen 80; server_name dev.tampv.pro.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;

        proxy_cache_bypass $http_upgrade;
    }

}

```

---

# 11. Enable site

```bash
sudo ln -s /etc/nginx/sites-available/dev.tampv.pro.com
/etc/nginx/sites-enabled/
```

---

# 12. Test Nginx config

```bash
sudo nginx -t

Expected:

syntax is ok\
test is successful
```

---

# 13. Reload Nginx

```bash
sudo systemctl reload nginx
```

---

# 14. Access website

```bash
http://dev.tampv.pro.com

Nginx will proxy request to:

localhost:3000
```

---

# 15. PM2 commands

```bash
Show processes:

pm2 list

Logs:

pm2 logs next-blog

Restart:

pm2 restart next-blog

Stop:

pm2 stop next-blog
```

---

# 16. Auto start after reboot

```bash
pm2 startup\
pm2 save

Follow the command printed by pm2.
```

---

# 17. Deploy workflow

```bash
git pull\
npm install\
npm run build\
pm2 restart next-blog
```

---

# 18. Debug 502 Bad Gateway

```bash
Common checks:

pm2 list\
lsof -i :3000\
pm2 logs next-blog

```
