# Context API — Quản lý dữ liệu toàn cục trong React

**🎯 Mục tiêu:**

- Hiểu cách **chia sẻ dữ liệu giữa nhiều component** trong React **mà không cần truyền props qua nhiều cấp (props drilling)**.
- Nắm vững cách tạo, cung cấp, và sử dụng **React Context API**.

---

## 1️⃣ Vấn đề: Props Drilling là gì?

**Props drilling** xảy ra khi bạn cần truyền dữ liệu qua nhiều cấp component, ngay cả khi một số component trung gian **không cần dùng dữ liệu đó**.

Ví dụ:

```jsx
function App() {
  const user = { name: "Tam" };
  return <Layout user={user} />;
}

function Layout({ user }) {
  return <Header user={user} />;
}

function Header({ user }) {
  return <UserInfo user={user} />;
}

function UserInfo({ user }) {
  return <p>Xin chào, {user.name}!</p>;
}
```

🧩 Vấn đề: `user` phải được truyền qua `Layout`, `Header` chỉ để đến `UserInfo`.

---

## 2️⃣ Giải pháp: Context API

**Context** cho phép bạn tạo một _“kho dữ liệu toàn cục”_ mà bất kỳ component nào trong cây có thể truy cập, **mà không cần props drilling**.

---

## 3️⃣ Tạo và sử dụng Context

### 🔹 Bước 1: Tạo Context

```jsx
import { createContext } from "react";

export const UserContext = createContext(null);
```

### 🔹 Bước 2: Tạo Provider

Provider bao bọc các component con, cung cấp dữ liệu toàn cục.

```jsx
import { useState } from "react";
import { UserContext } from "./UserContext";
import Dashboard from "./Dashboard";

function App() {
  const [user, setUser] = useState({ name: "Tam", role: "Admin" });

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Dashboard />
    </UserContext.Provider>
  );
}
```

### 🔹 Bước 3: Sử dụng dữ liệu trong component con

```jsx
import { useContext } from "react";
import { UserContext } from "./UserContext";

function UserProfile() {
  const { user } = useContext(UserContext);
  return <h2>Xin chào, {user.name}</h2>;
}
```

---

## 4️⃣ Cập nhật dữ liệu trong Context

```jsx
function UserProfile() {
  const { user, setUser } = useContext(UserContext);

  const changeName = () => setUser({ ...user, name: "Minh" });

  return (
    <div>
      <h2>Xin chào, {user.name}</h2>
      <button onClick={changeName}>Đổi tên</button>
    </div>
  );
}
```

> ✅ Tất cả component dùng Context đều cập nhật theo.

---

## 5️⃣ Tách riêng Provider cho gọn code

```jsx
import { createContext, useState } from "react";

export const UserContext = createContext();

export function UserProvider({ children }) {
  const [user, setUser] = useState({ name: "Tam", role: "Admin" });
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
}
```

Sử dụng trong `App.js`:

```jsx
import { UserProvider } from "./UserProvider";
import Dashboard from "./Dashboard";

function App() {
  return (
    <UserProvider>
      <Dashboard />
    </UserProvider>
  );
}
```

---

## 6️⃣ Khi nào nên dùng Context API?

| Tình huống                | Có nên dùng Context không? |
| ------------------------- | -------------------------- |
| Truyền theme (dark/light) | ✅ Có                      |
| Ngôn ngữ / i18n           | ✅ Có                      |
| Dữ liệu đăng nhập user    | ✅ Có                      |
| Danh sách sản phẩm nhỏ    | ✅ Có                      |
| Dữ liệu phức tạp, cache   | ❌ Dùng Redux hoặc Zustand |

---

## 7️⃣ useContext vs. Redux/Zustand

| Tiêu chí       | Context API                             | Redux / Zustand   |
| -------------- | --------------------------------------- | ----------------- |
| Cấu hình       | Đơn giản, có sẵn trong React            | Cần thêm thư viện |
| Phù hợp        | Ứng dụng nhỏ / trung bình               | Ứng dụng lớn      |
| Performance    | Re-render nhiều nếu không tách Provider | Tối ưu hơn        |
| Learning curve | Dễ học                                  | Phức tạp hơn      |

---

## 8️⃣ Tối ưu Context để tránh re-render

- Tách Context theo chức năng (UserContext, ThemeContext...).
- Dùng **useMemo** và **useCallback** để ổn định giá trị.
- Không bao `Provider` quá rộng.

```jsx
const value = useMemo(() => ({ user, setUser }), [user]);
<UserContext.Provider value={value}>...</UserContext.Provider>;
```

---

## 🧠 Tổng kết

| Kiến thức          | Mục tiêu                                 |
| ------------------ | ---------------------------------------- |
| **Props Drilling** | Hiểu vấn đề truyền dữ liệu qua nhiều cấp |
| **Context API**    | Tạo state toàn cục                       |
| **useContext**     | Truy cập dữ liệu từ bất kỳ component     |
| **Update Context** | Cho phép thay đổi dữ liệu toàn cục       |
| **Tối ưu Context** | Tránh re-render và chia nhỏ Provider     |
