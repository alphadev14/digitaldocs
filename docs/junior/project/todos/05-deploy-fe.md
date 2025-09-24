# 🚀 Deploy Frontend React/Vite lên Vercel

## 1. Chuẩn bị Project

- Frontend dùng **Vite + React**.
- Đảm bảo trong `package.json` có các scripts sau:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

---

## 2. Đăng ký và Kết nối với Vercel

1. Truy cập [https://vercel.com](https://vercel.com).
2. Đăng nhập bằng tài khoản **GitHub / GitLab / Bitbucket**.
3. Chọn **New Project** → Import repo FE của bạn.

---

## 3. Cấu hình Project trên Vercel

- **Chọn thư mục FE** (nếu repo có cả FE và BE).
- **Framework Preset**: chọn `Vite`.
- **Build Command**:

```bash
npm run build
```

- **Output Directory**:

```bash
dist
```

> Vì Vite build ra thư mục `dist/`.

---

## 4. Cấu hình Environment Variables

Vào **Project Settings → Environment Variables**:

- Thêm biến kết nối API, ví dụ:

```bash
VITE_API_URL=https://your-backend.onrender.com
```

⚠️ **Lưu ý**:

- Vite chỉ nhận biến bắt đầu bằng **`VITE_`**.
- Sau khi thêm biến → **Redeploy** để FE nhận giá trị mới.

---

## 5. Deploy

- Nhấn **Deploy**.
- Vercel sẽ build → nếu thành công sẽ có URL dạng:

```
https://your-fe-project.vercel.app
```

---

✅ Vậy là FE React/Vite đã deploy thành công lên Vercel 🎉
