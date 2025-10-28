# 🧩 React Foundation (Cơ bản)

**🎯 Mục tiêu:** Hiểu cách React hoạt động, component, props, state, và các hooks cơ bản (như `useState`, `useEffect`).

---

## 1️⃣ Giới thiệu React

**React** là thư viện JavaScript được phát triển bởi Facebook (Meta) dùng để xây dựng **UI (giao diện người dùng)**.  
React giúp chia nhỏ giao diện thành các **component** – các khối có thể tái sử dụng.

### Ưu điểm chính:

- Tư duy **Component-Based** (tái sử dụng code dễ dàng).
- **Virtual DOM** giúp cập nhật UI hiệu quả.
- Dễ mở rộng, tích hợp với framework khác.
- Hỗ trợ **Hooks** để quản lý state & logic.

---

## 2️⃣ JSX (JavaScript XML)

React dùng **JSX** để viết HTML trong JavaScript.  
Ví dụ:

```jsx
function Hello() {
  return <h1>Xin chào React!</h1>;
}
```

JSX cho phép nhúng biểu thức JavaScript bằng `{}`:

```jsx
const name = "MWG";
return <h2>Chào mừng đến với {name}</h2>;
```

---

## 3️⃣ Component trong React

### 🧱 Loại 1: **Function Component**

Là kiểu viết hiện đại – dùng **Hooks** để quản lý state và lifecycle.

```jsx
function Welcome(props) {
  return <h1>Xin chào, {props.name}</h1>;
}
```

Gọi component:

```jsx
<Welcome name="Phan" />
```

### 🧱 Loại 2: **Class Component** (ít dùng trong dự án mới)

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Xin chào, {this.props.name}</h1>;
  }
}
```

---

## 4️⃣ Props (Properties)

- **Props** là dữ liệu truyền từ component cha xuống component con.
- Props là **immutable** – không thể thay đổi trong component con.

```jsx
function UserCard({ name, age }) {
  return (
    <div>
      <h3>{name}</h3>
      <p>Tuổi: {age}</p>
    </div>
  );
}

// Dùng
<UserCard name="An" age={25} />;
```

---

## 5️⃣ State (Trạng thái)

**State** dùng để lưu trữ dữ liệu **thay đổi theo thời gian** bên trong component.  
Dùng hook `useState` để khai báo state.

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Bạn đã bấm {count} lần</p>
      <button onClick={() => setCount(count + 1)}>Bấm tôi</button>
    </div>
  );
}
```

---

## 6️⃣ Event Handling (Xử lý sự kiện)

React xử lý sự kiện tương tự HTML, nhưng dùng **camelCase**:

```jsx
<button onClick={handleClick}>Bấm</button>;

function handleClick() {
  alert("Đã bấm!");
}
```

Truyền tham số vào handler:

```jsx
<button onClick={() => handleClick("Xin chào!")}>Bấm</button>
```

---

## 7️⃣ Lifecycle cơ bản với `useEffect`

`useEffect` giúp chạy **side effect** (gọi API, thay đổi DOM, setInterval, v.v.)

### a. Chạy mỗi lần render

```jsx
useEffect(() => {
  console.log("Component render lại!");
});
```

### b. Chạy 1 lần khi component mount

```jsx
useEffect(() => {
  console.log("Chạy 1 lần khi load");
}, []);
```

### c. Chạy khi giá trị phụ thuộc thay đổi

```jsx
useEffect(() => {
  console.log("Count thay đổi");
}, [count]);
```

---

## 8️⃣ Tổng kết

| Khái niệm     | Mô tả ngắn                                |
| ------------- | ----------------------------------------- |
| **Component** | Khối UI độc lập có thể tái sử dụng        |
| **Props**     | Dữ liệu truyền từ cha xuống con           |
| **State**     | Dữ liệu nội bộ có thể thay đổi            |
| **JSX**       | Cú pháp kết hợp HTML và JS                |
| **useEffect** | Hook cho lifecycle (mount/update/unmount) |
