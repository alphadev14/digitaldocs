# ⚙️ Docker cơ bản phần 2

## 🎯 Mục tiêu

- Biết viết Dockerfile
- Biết build image
- Biết run container
- Hiểu các lệnh cơ bản trong Dockerfile

---

# 🧱 1. Dockerfile là gì?

Dockerfile là file chứa các bước để build ra một Docker Image.

👉 Hiểu đơn giản: Dockerfile = script build môi trường + app

---

# 🧠 2. Flow hoạt động

> Dockerfile → Build → Image → Run → Container

---

# 🔥 3. Các lệnh cơ bản trong Dockerfile

## 🧱 FROM

FROM node:18  
→ Chọn base image, bắt buộc phải có

## ⚙️ WORKDIR

WORKDIR /app  
→ Set thư mục làm việc trong container

## 📂 COPY

COPY package.json .  
→ Copy file từ máy local vào container

## 🔧 RUN

RUN npm install  
→ Chạy command khi build image

## 🚀 CMD

CMD ["npm", "start"]  
→ Command chạy khi container start

## 🌐 EXPOSE

EXPOSE 3000  
→ Khai báo port app sử dụng

---

# 🧪 4. Thực hành NodeJS

## index.js

```javascipt
const express = require('express');
const app = express();

app.get('/', (req, res) => {
res.send('Hello Docker!');
});

app.listen(3000, () => {
console.log('Server running');
});

```

---

```dockerfile
## Dockerfile

FROM node:18
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]

```

---

# ⚡ 5. Build Image

```dockerfile
docker build -t my-app .

```

---

# ▶️ 6. Run Container

```dockerfile
docker run -p 3000:3000 my-app
```

→ http://localhost:3000

---

# 💥 7. Debug

> 📋 Xem container

```docker
docker ps
```

> 🛑 Stop container

```docker
docker stop <container_id>
```

> 🧹 Xóa container

```docker
 docker rm <container_id>
```

---

# 🧠 8. Best Practices

- Dùng layer cache hợp lý
- Tránh COPY . . trước khi install
- Dùng .dockerignore
