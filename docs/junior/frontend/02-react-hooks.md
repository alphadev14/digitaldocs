# ⚙️ React Hooks

**🎯 Mục tiêu:** Hiểu và sử dụng thành thạo các hooks cơ bản và nâng cao trong React: `useState`, `useEffect`, `useMemo`, `useCallback`, và **Custom Hooks**.

---

## 1️⃣ Giới thiệu Hooks

**Hooks** là cơ chế giúp Function Component có thể sử dụng **state**, **lifecycle**, và các logic phức tạp mà trước đây chỉ có trong Class Component.

Hooks bắt đầu bằng từ khóa **`use`** và chỉ được dùng **trực tiếp bên trong function component** hoặc **custom hook khác**.

---

## 2️⃣ useState – Quản lý trạng thái

### ✅ Cú pháp:

```jsx
const [state, setState] = useState(initialValue);
```

### 📘 Ví dụ:

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Tăng</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

### 💡 Lưu ý:

- Mỗi lần gọi `setState` → component sẽ **re-render**.
- Nếu state phụ thuộc vào giá trị trước đó, nên dùng callback:
  ```jsx
  setCount((prev) => prev + 1);
  ```

---

## 3️⃣ useEffect – Xử lý side effects

Dùng để chạy code **bên ngoài luồng render** (gọi API, thao tác DOM, setTimeout, lắng nghe sự kiện, ...).

### ✅ Cú pháp:

```jsx
useEffect(() => {
  // logic
  return () => {
    // cleanup (optional)
  };
}, [dependencies]);
```

### 📘 Ví dụ:

```jsx
import { useState, useEffect } from "react";

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => setSeconds((s) => s + 1), 1000);
    return () => clearInterval(interval); // cleanup khi unmount
  }, []);

  return <p>Đã trôi qua: {seconds} giây</p>;
}
```

### 💡 Lưu ý:

- `[]` → chạy 1 lần khi component mount.
- `[deps]` → chạy lại khi deps thay đổi.
- Không có dependency → chạy sau mỗi lần render.

---

## 4️⃣ useMemo – Ghi nhớ giá trị tính toán

Dùng để **tối ưu hiệu năng** bằng cách ghi nhớ kết quả của phép tính tốn kém.

### ✅ Cú pháp:

```jsx
const memoizedValue = useMemo(() => computeValue(a, b), [a, b]);
```

- useMemo chỉ tính lại giá trị khi các dependency [a, b] thay đổi.
- Nếu dependencies không đổi → React trả lại giá trị cũ đã cache.

### 📘 Ví dụ:

```jsx
import { useMemo, useState } from "react";

function FibonacciCalculator() {
  const [num, setNum] = useState(10);
  const [count, setCount] = useState(0);

  const fib = (n) => {
    console.log("Đang tính Fibonacci...");
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
  };

  // Không có useMemo → fib() sẽ chạy lại MỖI LẦN render
  const result = useMemo(() => fib(num), [num]);

  return (
    <div>
      <p>
        Kết quả Fibonacci({num}) = {result}
      </p>
      <button onClick={() => setNum(num + 1)}>Tăng num</button>
      <button onClick={() => setCount(count + 1)}>Re-render ({count})</button>
    </div>
  );
}
```

➡️ Khi bạn bấm “Re-render”, nhờ useMemo, hàm fib() không chạy lại — vì num không thay đổi.

⚠️ Sai lầm phổ biến:

- Dùng useMemo mọi nơi: điều này không cần thiết và có thể làm code phức tạp hơn.
- Chỉ dùng khi:
  - Tính toán tốn CPU.
  - Hoặc giá trị được truyền xuống component con có thể gây render lại không cần thiết.

### 💡 Khi nào dùng:

- Khi logic tính toán phức tạp.
- Khi cần tránh tính lại trong mỗi lần render không cần thiết.

---

## 5️⃣ useCallback – Ghi nhớ hàm (function)

Giúp tránh tạo **hàm mới** trong mỗi lần render, đặc biệt hữu ích khi truyền callback xuống **component con** để tránh render lại không cần thiết.

### ✅ Cú pháp:

```jsx
const memoizedFn = useCallback(() => {
  /* logic */
}, [dependencies]);
```

### 📘 Ví dụ:

```jsx
import { useState, useCallback } from "react";

function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    setCount((c) => c + 1);
  }, []);

  return <Child onClick={handleClick} />;
}

const Child = React.memo(({ onClick }) => {
  console.log("Render Child");
  return <button onClick={onClick}>Tăng</button>;
});
```

### 💡 useMemo vs useCallback:

| Hook          | Dùng cho            | Trả về              |
| ------------- | ------------------- | ------------------- |
| `useMemo`     | Ghi nhớ **giá trị** | Giá trị tính toán   |
| `useCallback` | Ghi nhớ **hàm**     | Một hàm đã memoized |

---

## 6️⃣ Custom Hooks – Tạo hook riêng

Khi có logic muốn **tái sử dụng**, bạn có thể trích nó ra thành custom hook.

### 📘 Ví dụ: Hook theo dõi kích thước cửa sổ

```jsx
import { useState, useEffect } from "react";

function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () =>
      setSize({ width: window.innerWidth, height: window.innerHeight });
    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return size;
}
```

Dùng trong component:

```jsx
function App() {
  const { width, height } = useWindowSize();
  return (
    <p>
      Kích thước: {width} x {height}
    </p>
  );
}
```

---

## 7️⃣ Tổng kết

| Hook            | Mục đích                  | Ghi nhớ                |
| --------------- | ------------------------- | ---------------------- |
| **useState**    | Lưu trữ trạng thái        | Cập nhật gây re-render |
| **useEffect**   | Chạy side effects         | Có thể cleanup         |
| **useMemo**     | Ghi nhớ kết quả tính toán | Tối ưu hiệu năng       |
| **useCallback** | Ghi nhớ function          | Tránh render lại       |
| **Custom Hook** | Tái sử dụng logic         | Bắt đầu bằng `use`     |

---
