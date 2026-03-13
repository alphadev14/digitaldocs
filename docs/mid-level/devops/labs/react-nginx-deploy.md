# 🚀 Lab: Deploy ReactJS Frontend trên Ubuntu Server với Nginx

Ghi lại các bước thực hiện **deploy một dự án ReactJS lên
Ubuntu Server và chạy bằng Nginx**.

## Mục tiêu

- Cài NodeJS
- Build React project
- Cài đặt và cấu hình Nginx
- Deploy React build lên server
- Truy cập website bằng IP server

---

# 1. Chuẩn bị môi trường

Server sử dụng:

- Ubuntu 20.04 / 22.04
- Có quyền sudo
- Có kết nối internet

Update server:

```bash
sudo apt update
sudo apt upgrade -y
```

---

# 2. Cài NodeJS (version mới)

Ubuntu mặc định cài Node khá cũ nên dùng **NodeSource**.

### Thêm repository NodeSource

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
```

### Cài NodeJS

```bash
sudo apt install nodejs -y
```

### Kiểm tra

```bash
node -v
npm -v
```

---

# 3. Clone React Project lên Server

```bash
git clone https://github.com/your-repo/react-project.git
cd react-project
```

---

# 4. Cài Dependencies

```bash
npm install
```

---

# 5. Build React Project

```bash
npm run build
```

Sau khi build sẽ có thư mục:

    build/
     ├── index.html
     ├── static/

Đây là **static files để Nginx serve**.

---

# 6. Cài đặt Nginx

```bash
sudo apt install nginx -y
```

Kiểm tra trạng thái:

```bash
systemctl status nginx
```

---

# 7. Tạo thư mục chứa website

```bash
sudo mkdir -p /var/www/react-app
```

---

# 8. Copy React Build vào Nginx

```bash
sudo cp -r build/* /var/www/react-app
```

Kiểm tra:

```bash
ls /var/www/react-app
```

Kết quả:

    index.html
    static/

---

# 9. Cấu hình Nginx

Tạo file config:

```bash
sudo nano /etc/nginx/sites-available/react-app
```

Nội dung:

```nginx
server {
    listen 80;
    server_name SERVER_IP || DOMAIN_NAME;

    root /var/www/react-app;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```

Dòng quan trọng:

    try_files $uri /index.html;

Giúp React Router không bị lỗi **404 khi refresh trang**.

---

# 10. Enable Website

```bash
sudo ln -s /etc/nginx/sites-available/react-app /etc/nginx/sites-enabled/
```

---

# 11. Kiểm tra cấu hình Nginx

```bash
sudo nginx -t
```

---

# 12. Restart Nginx

```bash
sudo systemctl restart nginx
```

---

# 13. Mở Firewall (nếu cần)

```bash
sudo ufw allow 'Nginx Full'
```

hoặc

```bash
sudo ufw allow 80
```

---

# 14. Truy cập Website

Mở trình duyệt:

    http://SERVER_IP

Ví dụ:

    http://192.168.229.102

Nếu thấy React app chạy là deploy thành công 🎉

---

# 15. Cấu trúc Server Sau Khi Deploy

    /var/www/
       └── react-app
            ├── index.html
            ├── static/

Nginx config:

    /etc/nginx/sites-available/react-app

---

# 16. Flow Deploy React + Nginx

    React Source Code
           ↓
    npm install
           ↓
    npm run build
           ↓
    copy build → /var/www/react-app
           ↓
    nginx serve static files
           ↓
    Browser truy cập

---

# 17. Lưu ý quan trọng

1.  Nginx chỉ serve **static files**
2.  Sau khi build xong, server **không cần chạy NodeJS runtime**
3.  Nếu refresh bị lỗi 404 → kiểm tra dòng:

```{=html}
<!-- -->
```

    try_files $uri /index.html;

---

# 18. Dev Mode (chỉ để test)

```bash
npm run dev -- --host
```

Truy cập:

    http://SERVER_IP:5173

⚠️ Chỉ dùng cho **development**, không dùng production.

---

# 19. Kiến trúc hệ thống

    Browser
       ↓
    Nginx (port 80)
       ↓
    React Static Files
