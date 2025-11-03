# ⚡ React Performance Optimization

## 🎯 Mục tiêu

Hiểu và áp dụng các kỹ thuật tối ưu hiệu suất trong React

- Giảm số lần re-render không cần thiết
- Cải thiện tốc độ tải trang (load time).
- Tăng trải nghiệm người dùng.

---

## 1️⃣ Hiểu rõ cơ chế re-render trong React

React render lại component khi:

- **Prop** hoặc **State** thay đổi.
- **Context** thay đổi.
- **Parent component** render lại.

> ✅ Mục tiêu: tránh những re-render không cần thiết.

---

## 2️⃣ Sử dụng React.memo() – Tối ưu component con

```jsx
import React from "react";

const Child = React.memo(({ name }) => {
  console.log("Child render:", name);
  return <div>{name}</div>;
});

export default Child;
```

`React.memo()`giúp component chỉ render lại khi props thay đổi, tránh render thừa.

📌 Lưu ý: Nếu props là object/array/function — cần useCallback hoặc useMemo để giữ tham chiếu.

---

## 3️⃣ useCallback() – Ghi nhớ hàm callback

```jsx
import React, { useState, useCallback } from "react";
import Child from "./Child";

export default function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    console.log("Clicked!");
  }, []);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>Tăng</button>
      <Child onClick={handleClick} />
    </>
  );
}
```

- Nếu không dùng `useCallback`, mỗi lần parent render, `onClick` là hàm mới - Child render lại.
- `useCallback` giúp giữ cùng một tham chiếu hàm - tránh render lại.

---

## 4️⃣ useMemo() – Ghi nhớ giá trị tính toán nặng

```jsx
import React, { useMemo } from "react";

function ExpensiveCalculation({ num }) {
  const result = useMemo(() => {
    console.log("Calculating...");
    return num ** 2;
  }, [num]);

  return <p>Kết quả: {result}</p>;
}
```

Dùng khi:

- Tính toán phức tạp hoặc dữ liệu lớn.
- Tránh chạy lại logic nặng không cần thiết.

---

## 5️⃣ code Splitting - Chia nhỏ bundle

React hỗ trợ **dynamic import** để tải code chỉ khi cần.

```jsx
import React, { Suspense, lazy } from "react";

const About = lazy(() => import("./About"));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <About />
    </Suspense>
  );
}
```

✅ Giúp giảm bundle size ban đầu → tải nhanh hơn.
📦 Áp dụng cho:

- Trang hoặc module ít dùng.
- Component load route

---

## 6️⃣ Lazy Loading hình ảnh, component

Lazy load image:

```jsx
<img src="placeholder.jpg" loading="lazy" alt="demo" />
```

**Lazy load component:**

Kết hợp `React.lazy` và `Suspense` như phần trên.

---

## 7️⃣ Virtualization – Hiển thị danh sách lớn

Dùng thư viện như:

- react-window
- react-virtualized

```jsx
import { FixedSizeList as List } from "react-window";

function MyList({ items }) {
  return (
    <List height={400} itemCount={items.length} itemSize={35} width={300}>
      {({ index, style }) => <div style={style}>{items[index]}</div>}
    </List>
  );
}
```

✅ Chỉ render item hiển thị trên màn hình → siêu nhanh.

---

## 8️⃣ Profiling – Đo lường hiệu suất

**🧭 Dùng React Profiler (DevTools)**

- Mở tab Profiler trong React DevTools
- Ghi lại quá trình render - xem component nào tốn thời gian.
- Phát hiện "render thừa".

**🧰 Dùng Performance tab (Chrome)**

- Kiểm tra FPS, CPU, memory leak, repaint.

---

## 9️⃣ Tránh tạo object/array inline trong JSX

❌ Sai:

```jsx
<Child data={{ name: "Tam" }} />
```

✅ Đúng:

```jsx
const data = useMemo(() => ({ name: "Tam" }), []);
<Child data={data} />;
```

> Vì { name: "Tam" } tạo object mới mỗi lần render → Child render lại.

---

## 🔟 Tối ưu build & deploy

🧱 **Sử dụng production build:**

```bash
npm run build
```

🧹 **Bật tree-shaking:**
Webpack, Vite, CRA đều hỗ trợ → loại bỏ code không dùng.

🚀 **Dùng CDN hoặc caching:**

- Cache ảnh, JS, CSS.
- Dùng HTTP/2 hoặc CDN để giảm độ trễ.

---

✅** Checklist Tối ưu Hiệu Suất**

| Mục tiêu            | Kỹ thuật                         |
| ------------------- | -------------------------------- |
| Giảm re-render      | React.memo, useCallback, useMemo |
| Tối ưu load ban đầu | Code splitting, lazy loading     |
| Danh sách lớn       | react-window, react-virtualized  |
| Theo dõi hiệu suất  | React Profiler, Chrome DevTools  |
| Giảm bundle size    | Tree-shaking, CDN, compression   |

💡 **Kết luận**

> Hiệu suất tốt không chỉ đến từ tối ưu code mà còn từ thiết kế component hợp lý và quản lý state thông minh.
