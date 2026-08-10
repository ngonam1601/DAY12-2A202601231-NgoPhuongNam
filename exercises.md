# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng hướng dẫn bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Ngô Phương Nam  Mã học viên: 2A202601231

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy ứng dụng lên Production nhưng người vận hành quên cấu hình biến môi trường `AGENT_API_KEY`. Nếu để giá trị mặc định `"changeme"`, ứng dụng vẫn khởi động bình thường và kẻ tấn công hoặc người lạ có thể lợi dụng khóa mặc định này để gọi API miễn phí, làm kiệt huệ tài nguyên và tăng chi phí LLM ngoài kiểm soát mà không ai phát hiện ra. Ngược lại, việc không để giá trị mặc định giúp app "chết sớm" (Fail Fast) ngay lúc khởi động, giúp hệ thống giám sát cảnh báo tức thì để khắc phục trước khi ứng dụng nhận traffic.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log JSON thu được:
> `{"event": "ask_completed", "level": "info", "timestamp": "2026-08-10T05:33:16.123456+00:00", "user_id": "sv01", "tokens_in": 249, "tokens_out": 47, "cost_usd": 0.00006555}`
>
> Hai việc làm được với dòng log JSON:
> 1. Hệ thống giám sát tự động (Datadog/Elasticsearch) có thể parse các trường để lọc/tìm kiếm chính xác theo `user_id`, `level` hoặc cảnh báo khi `cost_usd` vượt ngưỡng mà không cần viết regex phức tạp.
> 2. Dễ dàng tính toán tổng hợp dữ liệu thời gian thực như tổng chi phí token, mức độ tiêu thụ của từng user để vẽ biểu đồ giám sát hệ thống.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản                 | Dung lượng |
| -------------------- | ------------ |
| 1 stage (bản đầu) | 446 MB       |
| Multi-stage          | 63.7 MB      |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần dung lượng chênh lệch chủ yếu đến từ base image đầy đủ `python:3.11` và các thành phần không cần thiết cho môi trường production. Multi-stage sử dụng `python:3.11-slim` ở production stage và chỉ copy các dependency cần thiết, nên image nhỏ hơn đáng kể.

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi sửa `app/main.py`, các layer cài đặt dependency như `COPY requirements.txt` và `RUN pip install` vẫn được dùng lại từ cache vì file `requirements.txt` không đổi. Chỉ có layer `COPY . .` và các bước sau mới phải chạy lại. Nếu đặt `COPY . .` lên trước `RUN pip install`, mỗi lần sửa code thì layer `COPY . .` bị thay đổi sẽ làm mất cache của tất cả các lệnh sau, buộc Docker phải chạy lại `RUN pip install` để tải lại toàn bộ thư viện từ đầu, khiến thời gian build rất lâu.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Kẻ tấn công khai thác lỗ hổng Remote Code Execution (RCE) trong code Python để thực thi lệnh hệ thống. Vì container chạy quyền root, tiến trình Python có quyền tối cao bên trong container. Kẻ tấn công khai thác lỗi container breakout để chiếm quyền root trên máy host. Lệnh `USER nonroot` cắt đứt chuỗi tấn công ngay từ đầu bằng cách tước quyền root của tiến trình container, khiến kẻ tấn công khi RCE thành công chỉ thu được quyền của user thường, không thể thao tác các file nhạy cảm hay leo leo quyền lên máy host.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng có thể gửi tối đa **20 request** trong 2 giây liên tiếp. Cách đạt được: gửi 10 request ở giây 10:00:59 (cuối phút thứ nhất) và gửi tiếp 10 request ở giây 10:01:00 (đầu phút thứ hai). Thuật toán đếm theo phút đồng hồ xem đây là 2 phút riêng biệt nên cho qua cả 20 request, mặc dù thực tế 20 request này xảy ra trong khoảng thời gian chỉ 2 giây.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Khác biệt: Rate limit giới hạn *số lượng request* trong khoảng thời gian ngắn (tần suất). Cost guard giới hạn *tổng số tiền/token* tiêu tốn trong khoảng thời gian dài (ngân sách).
> - Rate limit cho qua nhưng Cost guard chặn: User gửi 2 request/phút (dưới hạn mức 10 req/phút), nhưng mỗi request chứa prompt khổng lồ tiêu tốn hết ngân sách tháng. Cost guard sẽ chặn request thứ 2.
> - Cost guard cho qua nhưng Rate limit chặn: User mới bắt đầu tháng chưa tiêu tiền (ngân sách còn nguyên), nhưng gửi 15 request liên tục trong 3 giây. Rate limit sẽ chặn từ request thứ 11.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> 1. Redis mất kết nối 30s $\rightarrow$ Endpoint trả về kesalahan 503.
> 2. Orchestrator tưởng cả 3 container agent bị chết (do liveness probe `/health` thất bại).
> 3. Orchestrator tiến hành ngắt và restart cả 3 container agent.
> 4. Trong khi restart, Redis vẫn chưa lên lại $\rightarrow$ container tiếp tục báo hỏng $\rightarrow$ rơi vào vòng lặp restart liên tục (CrashLoopBackOff), tiêu tốn tài nguyên vô ích.
> 5. Khi Redis phục hồi, cả cụm container bị quá tải do quá trình restart hàng loạt thay vì chỉ đơn giản là tạm ngưng nhận traffic và tự phục hồi khi Redis hoạt động trở lại.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Nếu lưu trong dict Python, các request tiếp theo được Load Balancer chia ngẫu nhiên cho 3 container khác nhau. Mỗi container chỉ giữ lịch sử của riêng nó, khiến giá trị `history_length` thay đổi nhảy vọt thất thường (ví dụ: req 1 vào A $\rightarrow$ 0, req 2 vào B $\rightarrow$ 0, req 3 vào A $\rightarrow$ 2...). Ngược lại khi lưu ở Redis chung, cả 3 container cùng truy cập 1 bộ nhớ nên `history_length` luôn tăng dần liên tục và đồng nhất (0, 2, 4, 6, 8...).

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Thông báo lỗi: `Error response from daemon: Bind for 0.0.0.0:8000 failed: port is already allocated`.
> Cách tìm nguyên nhân: Kiểm tra danh sách container bằng `docker ps` và `netstat -ano`, phát hiện cổng 8000 đã bị chiếm dụng bởi container cũ hoặc tiến trình Uvicorn ngầm.
> Cách sửa: Chạy `docker compose down` để giải phóng cổng 8000, kiểm tra file cấu hình `.env` để chuyển `REDIS_URL` chuẩn xác rồi khởi động lại.
