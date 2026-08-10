# Phiếu Phản Ánh — K4 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay từng phần giữ chỗ bên dưới bằng câu trả lời của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Vương Đức Thoại    Mã học viên: 2A202601770

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi
khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc
"chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Ví dụ, khi deploy lên Render, nếu tôi quên cấu hình `API_TOKEN` mà chương
> trình dùng mặc định `"changeme"`, service vẫn khởi động và người khác có thể
> đoán token để gọi `/chat`. Khi `api_token` không có giá trị mặc định,
> Pydantic báo ValidationError ngay lúc khởi động. Deploy thất bại hiện rõ
> trong log, giúp tôi sửa cấu hình trước khi service nhận traffic và phát sinh
> chi phí.
---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng log tôi thu được:
>
> `{"event":"chat_completed","severity":"INFO","ts":"2026-08-10T09:00:00+00:00","client_id":"sv01","prompt_tokens":10,"completion_tokens":40,"usd_cost":0.0000255}`
>
> Với JSON log, tôi có thể lọc tất cả sự kiện `chat_completed` của một
> `client_id` cụ thể và cộng trường `usd_cost` để biết client nào tiêu nhiều
> tiền nhất. Tôi cũng có thể đếm log theo `severity`, tạo cảnh báo khi số lỗi
> tăng cao. Dòng `print("đã trả lời xong")` không có các trường có cấu trúc nên
> khó lọc, tổng hợp và cảnh báo tự động.
---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t chat:single .
docker build -t chat:multi .
docker images | grep chat
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ... MB |
| Multi-stage | ... MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 183MB |
| Multi-stage | 183 MB |

> Bản một stage lớn hơn vì dùng image `python:3.11` đầy đủ và giữ lại toàn bộ
> thành phần của môi trường build trong image chạy thật. Bản multi-stage dùng
> `python:3.11-slim`; stage builder cài dependency rồi chỉ copy kết quả
> `/install` sang runtime. Các thành phần chỉ cần lúc build không nằm trong
> image cuối, nên image nhỏ hơn đáng kể.
---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi tôi chỉ sửa `app/main.py`, các layer base image, `COPY requirements.txt`
> và `RUN pip install` vẫn được lấy từ cache vì requirements không đổi. Layer
> `COPY app ./app` và các layer đứng sau nó phải chạy lại; `COPY utils ./utils`
> vẫn có thể dùng cache nếu nội dung utils không đổi. Nếu đặt `COPY . .` trước
> `RUN pip install`, mọi thay đổi source đều làm layer COPY thay đổi và khiến
> Docker chạy lại pip install, dù requirements vẫn giữ nguyên. Build vì vậy
> chậm hơn và tải/cài thư viện lại không cần thiết.
---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu ứng dụng Python có lỗ hổng thực thi mã từ xa, kẻ tấn công có thể chạy
> lệnh với quyền của process trong container. Nếu process chạy root, chúng có
> quyền cao nhất bên trong container và có thêm cơ hội lợi dụng kernel,
> volume nhạy cảm hoặc cấu hình container sai để tác động tới host. Lệnh
> `USER appuser` khiến mã bị chiếm quyền chỉ chạy bằng user thường, không thể
> tùy ý sửa file hệ thống hoặc thực hiện nhiều thao tác đặc quyền. Nó không
> loại bỏ mọi khả năng container escape, nhưng giảm đáng kể phạm vi thiệt hại.
---

### Câu 6 — Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả **cùng
một** thông báo lỗi cho cả ba trường hợp (thiếu header, sai scheme, sai token)
thay vì nói rõ sai ở đâu cho người dùng dễ sửa?

> Header `WWW-Authenticate: Bearer` cho client biết endpoint yêu cầu cơ chế
> Bearer token, giúp client xử lý response 401 đúng chuẩn HTTP. Tôi dùng cùng
> thông báo cho thiếu header, sai scheme và sai token để không tiết lộ bước
> nào đã đúng. Nếu trả các thông báo khác nhau, kẻ tấn công có thể dùng chúng
> như manh mối để lần lượt xác định scheme và dò token.
---

### Câu 7 — Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi
liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn
`min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

> Với `capacity=10` và `refill_per_minute=10`, sau khi im lặng 10 phút client
> vẫn chỉ gửi liên tiếp được tối đa 10 request trước khi request tiếp theo bị
> 429. Sức chứa giới hạn lượng token tích lũy. Nếu bỏ `min(capacity, ...)`,
> sau 10 phút xô có thể tích thêm 100 token; từ trạng thái đầy nó có thể đạt
> khoảng 110 token và cho phép một đợt request quá lớn. Điều này làm mất ý
> nghĩa giới hạn burst của token bucket.
---

### Câu 8 — Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự
cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là
bao nhiêu và service tự hồi phục khi nào?

> Với hạn mức 30 USD mỗi tháng, sự cố bắt đầu lúc 2 giờ sáng có thể tiêu hết
> toàn bộ phần ngân sách tháng còn lại, tối đa 30 USD. Client sau đó có thể bị
> chặn tới đầu tháng sau nếu không có người can thiệp. Với hạn mức 1 USD mỗi
> ngày, thiệt hại trong ngày bị giới hạn ở khoảng 1 USD. Khi chuyển sang ngày
> UTC mới, key chi phí thay đổi và service tự có ngân sách mới. Hạn mức ngày
> vì vậy giới hạn thiệt hại của một sự cố tốt hơn và tự hồi phục sớm hơn.
---

### Câu 9 — /healthz khác /readyz (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp hai endpoint và cho health check kiểm tra Redis, khi Redis mất kết
> nối thì cả ba container cùng báo unhealthy. Orchestrator tưởng cả ba process
> đều hỏng nên restart chúng. Sau khi khởi động lại, Redis vẫn chưa hoạt động,
> các container tiếp tục báo unhealthy và bị restart lặp lại. Một sự cố Redis
> 30 giây vì vậy bị khuếch đại thành sự cố của toàn bộ cụm. Tách `/healthz`
> giúp process vẫn báo còn sống, còn `/readyz` trả 503 để load balancer tạm
> ngừng gửi traffic vào instance chưa phục vụ được.
---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi deploy lên Render, tôi mở Public URL và nhận `404 Not Found`. Khi kiểm
> tra bằng curl, response có header `x-render-routing: no-server`, cho thấy
> phản hồi đến từ router của Render chứ không phải FastAPI. Tôi mở deploy log
> và kiểm tra các dòng khởi động Uvicorn, health check và trạng thái service.
> Sau khi chạy deploy mới nhất và chờ log xuất hiện `Application startup
> complete` cùng `Your service is live`, `/healthz` và `/readyz` hoạt động.
> Tôi cũng nhận ra URL gốc `/` vẫn trả 404 vì ứng dụng không khai báo route
> này; các endpoint đúng phải là `/healthz`, `/readyz` và `/chat`.