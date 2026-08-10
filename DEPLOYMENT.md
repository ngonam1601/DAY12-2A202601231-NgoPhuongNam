# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Ngô Phương Nam |
| Mã học viên | 2A202601231 |
| Repo | https://github.com/ngonam1601/DAY12-2A202601231-NgoPhuongNam |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-j2cv.onrender.com |
| Platform | Render Blueprint |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set

Chỉ ghi tên biến và nguồn giá trị; không ghi secret.

| Biến | Trạng thái | Nguồn/ghi chú |
|------|------------|---------------|
| `PORT` | ✅ | Render tự gán |
| `AGENT_API_KEY` | ✅ | secret trong môi trường chạy, không lưu trong repo |
| `REDIS_URL` | ✅ | Render Key Value `day12-redis` |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

Để chạy test xác thực CP5 không bị skip, thêm `DEPLOY_API_KEY` vào `.env` local
với đúng giá trị `AGENT_API_KEY` đang đặt trên Render. Không commit giá trị này.

## Kết Quả Kiểm Tra Thực Tế

Service đã được deploy public trên Render và dùng Redis Key Value cùng Blueprint:

```text
GET /       -> redirect tới `/docs` (Swagger UI)
GET /health -> 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET /ready  -> 200 {"status":"ready","redis":true}
POST /ask không có X-API-Key -> 401
```

Các kết quả trên được kiểm tra ngày 2026-08-10 với URL public:
`https://day12-agent-j2cv.onrender.com`.

Swagger UI: `https://day12-agent-j2cv.onrender.com/docs`.

## Ảnh Chụp

- `screenshots/health.png` — kết quả kiểm tra endpoint `/health`.
- `screenshots/README.md` — hướng dẫn bổ sung ảnh dashboard.

## Ghi Chú Bảo Mật

Giá trị thật của `AGENT_API_KEY` và `REDIS_URL` không được ghi vào tài liệu hoặc
repository. Các secret được cấu hình trực tiếp trong Render Environment.
