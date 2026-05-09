# Data Fetching & Cache

## Mục tiêu

Fetch data ổn định hơn với loading, error, retry, cache và revalidation.

## Cần nắm

- Loading state.
- Error state.
- Empty state.
- Retry.
- Cache key.
- Stale time.
- Refetch.
- Optimistic update.

## Checklist

- Mỗi API call có loading/error/empty UI.
- Cache key phản ánh đủ filter/pagination.
- Không fetch trùng lặp không cần thiết.
- Mutation invalidate đúng query liên quan.
- Có skeleton hoặc loading phù hợp.

## Bài thực hành

- Dùng React Query để fetch product list.
- Thêm mutation update product và invalidate list/detail.
- Thử optimistic update cho action toggle active.

