# Frontend Performance

## Mục tiêu

Giảm render thừa, giảm bundle size và cải thiện cảm nhận tốc độ của người dùng.

## Cần nắm

- React render lifecycle.
- Memoization đúng chỗ.
- Code splitting.
- Lazy loading.
- Virtualization cho list lớn.
- Image optimization.
- Web Vitals.

## Checklist

- Không memo hóa bừa bãi.
- List lớn dùng virtualization.
- Route lớn được code split.
- Image có kích thước phù hợp.
- Component nặng được đo bằng React DevTools Profiler.
- Bundle được kiểm tra khi app tăng kích thước.

## Bài thực hành

- Profile một page render chậm.
- Tách route dashboard bằng lazy import.
- Virtualize table có 10.000 rows.

