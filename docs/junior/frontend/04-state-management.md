# 🧠 State Management trong React

## 🎯 Mục tiêu

Hiểu cách quản lý và chia sẻ state trong ứng dụng React, từ cơ bản đến nâng cao, sử dụng Context API, Redux và Zustand.

---

## 1️⃣ Giới thiệu về State Management

Trong React, **state** là dữ liệu quyết định giao diện được hiển thị.  
Khi ứng dụng mở rộng, việc chia sẻ state giữa nhiều component trở nên phức tạp — đây là lúc cần **state management**.

### 🔹 Ba cấp độ quản lý state:

1. **Local State** – sử dụng `useState`, `useReducer`.
2. **Shared State** – chia sẻ qua Context API.
3. **Global State** – sử dụng thư viện quản lý state chuyên dụng (Redux, Zustand).

---

## 2️⃣ Redux – State Management Truyền Thống và Mạnh Mẽ

### 💡 Giới thiệu

Redux là thư viện giúp quản lý state tập trung (Centralized Store).  
Dữ liệu chỉ có thể được thay đổi thông qua **actions** và **reducers**, đảm bảo tính nhất quán.

### 🔧 Cấu trúc Redux cơ bản

```
src/
├── store/
│   ├── index.js
│   ├── actions.js
│   ├── reducers.js
│   └── slice.js (với Redux Toolkit)
```

### 🧩 Nguyên tắc hoạt động

1. Component gửi **action** →
2. **Reducer** xử lý action và cập nhật state →
3. **Store** phát (dispatch) state mới đến tất cả component.

### ⚙️ Cài đặt Redux Toolkit

```bash
npm install @reduxjs/toolkit react-redux
```

### ✍️ Ví dụ cơ bản

```jsx
// store.js
import { configureStore, createSlice } from "@reduxjs/toolkit";

const counterSlice = createSlice({
  name: "counter",
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
  },
});

export const { increment, decrement } = counterSlice.actions;
export const store = configureStore({
  reducer: { counter: counterSlice.reducer },
});
```

```jsx
// App.js
import React from "react";
import { Provider, useDispatch, useSelector } from "react-redux";
import { store, increment, decrement } from "./store";

function Counter() {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
    </div>
  );
}

export default function App() {
  return (
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}
```

### ⚡ Ưu điểm

- Dễ debug, dễ kiểm soát luồng dữ liệu.
- Redux DevTools hỗ trợ mạnh mẽ.
- Phù hợp với dự án lớn, nhiều logic state.

### ⚠️ Nhược điểm

- Boilerplate code nhiều (đã cải thiện với Redux Toolkit).
- Hơi nặng cho ứng dụng nhỏ.

---

## 3️⃣ Zustand – State Management Nhẹ, Hiện Đại

### 💡 Giới thiệu

**Zustand** là thư viện quản lý state nhẹ và đơn giản hơn Redux, được phát triển bởi team của React Three Fiber.

### ⚙️ Cài đặt

```bash
npm install zustand
```

### ✍️ Ví dụ

```jsx
import { create } from "zustand";

const useCounterStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

function Counter() {
  const { count, increment, decrement } = useCounterStore();
  return (
    <div>
      <h2>{count}</h2>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}

export default Counter;
```

### ⚡ Ưu điểm

- Cực kỳ nhẹ và dễ dùng.
- Không cần boilerplate (actions, reducers).
- Tích hợp tốt với TypeScript.
- Hỗ trợ server-side rendering (Next.js).

### ⚠️ Nhược điểm

- Không có DevTools mạnh như Redux (nhưng có plugin riêng).
- Khó kiểm soát nếu state quá phức tạp.

---

## 4️⃣ So sánh Redux và Zustand

| Tiêu chí            | Redux Toolkit            | Zustand                            |
| ------------------- | ------------------------ | ---------------------------------- |
| Cấu trúc            | Cứng nhắc, có quy ước rõ | Linh hoạt, tùy theo lập trình viên |
| Dễ dùng             | Trung bình               | Dễ                                 |
| Performance         | Rất tốt                  | Rất tốt                            |
| Debug tools         | Có sẵn (Redux DevTools)  | Có (nhưng giới hạn)                |
| Tích hợp TypeScript | Tốt                      | Tốt                                |
| Dự án phù hợp       | Lớn, nhiều module        | Nhỏ & trung bình                   |

---

## 5️⃣ Khi nào dùng cái nào?

- **Redux**: khi ứng dụng lớn, nhiều state phức tạp, cần log, undo, hoặc history.
- **Zustand**: khi ứng dụng nhỏ hoặc trung bình, cần tốc độ và code gọn gàng.
- **Context API**: khi chỉ cần chia sẻ state đơn giản.

---

## ✅ Kết luận

Hiểu và chọn đúng công cụ quản lý state là chìa khóa để giữ cho ứng dụng React **ổn định, dễ bảo trì và mở rộng**.

> 🔖 Tip: Nếu bạn chỉ có vài component cần chia sẻ state — dùng Context.  
> Nếu dự án phát triển lớn — chuyển sang Redux hoặc Zustand.
