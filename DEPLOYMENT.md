# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Vương Đức Thoại |
| Mã học viên | 2A202601770 |
| Repo | REPOSITORY_URL |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-chat-rw0y.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

| Biến | Trạng thái | Ghi chú |
|------|------------|---------|
| `PORT` | Đã set | Render tự cung cấp |
| `API_TOKEN` | Đã set | Secret trên Render Dashboard |
| `REDIS_URL` | Đã set | Render Key Value connection string |
| `BUCKET_CAPACITY` | Đã set | Giá trị 10 |
| `REFILL_PER_MINUTE` | Đã set | Giá trị 10 |
| `DAILY_BUDGET_USD` | Đã set | Giá trị 1.0 |
| `LOG_LEVEL` | Đã set | INFO |

## Kết Quả Chạy Thật

```text
GET /healthz
HTTP 200
{"status":"ok","service":"day12-chat-service","version":"1.0.0"}

GET /readyz
HTTP 200
{"status":"ready","redis":true}

POST /chat không có Authorization
HTTP 401
{"detail":"invalid or missing bearer token"}

POST /chat có Bearer token hợp lệ
HTTP 200
Response có reply, client_id, turns_before, usd_cost và usage.