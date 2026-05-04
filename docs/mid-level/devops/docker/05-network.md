# 📦 Làm việc với Container

## 🎯 Mục tiêu

- Quản lý container
- Biết debug container
- Hiểu workflow xử lý lỗi khi deploy

---

# 🧠 1. Tư duy quan trọng

> Container = 1 process đang chạy

- Container chết → app chết
- Không có system như VM
- Log = stdout

---

# 📋 2. Xem container

## Container đang chạy

```
docker ps
```

## Tất cả container

```

docker ps -a
```

→ Bao gồm running / stopped / exited

---

# 🧱 3. Xem image

```

docker images
```

---

# 🛑 4. Stop & Remove

```
docker stop <container_id>
docker rm <container_id>
docker rm -f <container_id>
```

---

# 📜 5. Logs

```
docker logs <container_id>
docker logs -f <container_id>
```

→ Debug chính bằng logs

---

# 🧪 6. Exec vào container

```
docker exec -it <container_id> bash
docker exec -it <container_id> sh
```

---

# 💥 7. Debug workflow

## Case 1: Container crash

```

docker ps -a
docker logs <id>
docker exec -it <id> bash
```

## Case 2: Không connect DB

```

docker exec -it backend bash
ping postgres
curl postgres:5432

```

---

# 🧠 8. Best Practices

- Chạy foreground process
- Log ra stdout
- Không SSH container
- One container = one service
