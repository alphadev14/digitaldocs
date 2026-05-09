# State Management

## Mục tiêu

Biết phân loại state để tránh đưa mọi thứ vào global store và làm app khó maintain.

## Cần nắm

- Local UI state.
- Form state.
- URL state.
- Global client state.
- Server state.
- Derived state.
- Context API, Zustand, Redux Toolkit, React Query.

## Checklist

- State chỉ đặt global khi nhiều nơi thật sự cần.
- Filter/search/pagination nên cân nhắc đưa lên URL.
- Server data nên dùng cache layer như React Query/SWR.
- Không duplicate state nếu có thể derive từ state khác.
- Form phức tạp nên dùng form library phù hợp.

## Bài thực hành

- Làm trang search product lưu filter trên URL.
- Tách form state khỏi server state.
- Refactor một Context quá lớn thành nhiều context nhỏ hoặc store theo feature.

