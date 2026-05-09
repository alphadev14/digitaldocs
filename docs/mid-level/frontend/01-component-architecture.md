# Component Architecture

## Mục tiêu

Xây dựng component dễ đọc, dễ tái sử dụng và không bị phình to theo thời gian.

## Cần nắm

- Presentational component vs container component.
- Composition.
- Controlled vs uncontrolled component.
- Custom hook.
- Feature-based folder structure.
- Design system component.

## Checklist

- Component có trách nhiệm rõ ràng.
- UI logic và business/data logic được tách hợp lý.
- Props không quá nhiều hoặc quá mơ hồ.
- Component lặp lại được đưa vào shared component.
- Component theo feature nằm gần nơi sử dụng.

## Bài thực hành

- Tách trang product list thành filter, table, pagination, empty state.
- Viết custom hook `useProductFilters`.
- Tạo shared `ConfirmDialog` dùng lại ở nhiều feature.

