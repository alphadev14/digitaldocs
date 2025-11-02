# React Router – Routing & Navigation

## 🎯 Mục tiêu học tập

- Hiểu cách React Router hoạt động.
- Tạo ứng dụng nhiều trang với nested routes.
- Sử dụng `useNavigate`, `useParams`, `Outlet`.
- Tạo **Protected Route** (route bảo vệ người dùng chưa đăng nhập).
- Tối ưu hiệu năng với **lazy loading** routes.

---

## 1️⃣ Cài đặt

```bash
npm install react-router-dom
```

---

## 2️⃣ Cấu trúc cơ bản của React Router

```jsx
import { BrowserRouter as Router, Routes, Route } from "react-router-dom";
import Home from "./pages/Home";
import About from "./pages/About";
import NotFound from "./pages/NotFound";

function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </Router>
  );
}

export default App;
```

✅ `path="*"` dùng để xử lý route không tồn tại (404 page).

---

## 3️⃣ Nested Routes (Lồng route con)

```jsx
import { Routes, Route, Outlet } from "react-router-dom";

function Dashboard() {
  return (
    <div>
      <h2>Dashboard</h2>
      <Outlet />
    </div>
  );
}

function Users() {
  return <div>User Management</div>;
}

function Settings() {
  return <div>Settings</div>;
}

function App() {
  return (
    <Routes>
      <Route path="dashboard" element={<Dashboard />}>
        <Route path="users" element={<Users />} />
        <Route path="settings" element={<Settings />} />
      </Route>
    </Routes>
  );
}
```

---

## 4️⃣ useNavigate – Điều hướng bằng code

```jsx
import { useNavigate } from "react-router-dom";

function Login() {
  const navigate = useNavigate();

  const handleLogin = () => {
    navigate("/dashboard");
  };

  return <button onClick={handleLogin}>Đăng nhập</button>;
}
```

---

## 5️⃣ useParams – Lấy tham số từ URL

```jsx
import { useParams } from "react-router-dom";

function ProductDetail() {
  const { id } = useParams();
  return <h3>Chi tiết sản phẩm ID: {id}</h3>;
}
```

---

## 6️⃣ Protected Route (Route bảo vệ)

```jsx
import { Navigate } from "react-router-dom";

function ProtectedRoute({ children }) {
  const isAuthenticated = localStorage.getItem("token");
  return isAuthenticated ? children : <Navigate to="/login" />;
}
```

**Sử dụng:**

```jsx
<Route
  path="/dashboard"
  element={
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  }
/>
```

---

## 7️⃣ Lazy Loading Routes

```jsx
import React, { lazy, Suspense } from "react";
import { Routes, Route } from "react-router-dom";

const Home = lazy(() => import("./pages/Home"));
const About = lazy(() => import("./pages/About"));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </Suspense>
  );
}
```

---

## 8️⃣ Layout Route (Chung layout cho nhiều trang)

```jsx
function Layout() {
  return (
    <div>
      <header>Header</header>
      <Outlet />
      <footer>Footer</footer>
    </div>
  );
}

<Route path="/" element={<Layout />}>
  <Route index element={<Home />} />
  <Route path="about" element={<About />} />
</Route>;
```

---

## 9️⃣ Navigate component (Redirect)

```jsx
import { Navigate } from "react-router-dom";

<Route path="/old-home" element={<Navigate to="/" replace />} />;
```

---

## 🔟 Tổng kết

| Kiến thức chính     | Ý nghĩa                         |
| ------------------- | ------------------------------- |
| `BrowserRouter`     | Bọc toàn bộ ứng dụng            |
| `Routes` & `Route`  | Khai báo route                  |
| `Outlet`            | Render route con                |
| `useNavigate`       | Điều hướng bằng code            |
| `useParams`         | Lấy param động từ URL           |
| `ProtectedRoute`    | Bảo vệ route cần đăng nhập      |
| `lazy` + `Suspense` | Tải component khi cần để tối ưu |

---
