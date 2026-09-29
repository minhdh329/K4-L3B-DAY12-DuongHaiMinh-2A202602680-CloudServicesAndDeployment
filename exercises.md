# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Dương Hải Minh  Mã học viên: 2A202602680

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> *Câu trả lời của bạn*
App không có cấu hình hợp lệ để xác thực /ask; nếu dùng changeme, app vẫn chạy với khóa đã biết và người khác có thể gọi endpoint đó.
---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> *Câu trả lời của bạn*
{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T07:26:26.329758+00:00", "user_id": "exercise-log-check", "tokens_in": 6, "tokens_out": 38, "cost_usd": 2.37e-05}
-Lọc các event theo user_id, event hoặc level để tìm hoạt động hay lỗi của một người dùng.
-Tổng hợp cost_usd và token theo người dùng hoặc khoảng thời gian để tìm nơi phát sinh chi phí.
---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.73 GB |
| Multi-stage | 310 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> *Câu trả lời của bạn*
Chênh lệch khoảng 1.42 GB. Phần lớn đến từ base python:3.11 đầy đủ ở bản one-stage; bản multi-stage dùng python:3.11-slim cho runtime và không đưa toàn bộ builder stage, cùng pip cache, vào image cuối.
---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> *Câu trả lời của bạn*
Khi sửa một ký tự trong file app/main.py, quá trình build sẽ diễn ra như sau:
Layer được dùng lại (từ cache): Các layer nằm trước lệnh COPY app ./app, bao gồm base image và bước cài đặt dependencies. Layer phải chạy lại: Layer COPY app sẽ bị build lại, kéo theo tất cả các lệnh (instruction) nằm sau nó cũng phải chạy lại. 
Nếu bạn đặt COPY . . lên trước RUN pip install:
Bất kỳ thay đổi nào trong source code cũng sẽ làm thay đổi layer COPY này.
Hệ quả là cache của lệnh RUN pip install sẽ bị mất hiệu lực, buộc hệ thống phải tải và cài đặt lại toàn bộ dependencies từ đầu dù bạn chỉ sửa một dòng code.
---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> *Câu trả lời của bạn*
Lỗ hổng ứng dụng cho phép kẻ tấn công thực thi lệnh bên trong container. Nếu để mặc định chạy bằng quyền root, chúng có toàn quyền kiểm soát container và có nguy cơ leo thang chiếm quyền máy host (nếu container có quyền đặc biệt hoặc mount files).
Lệnh USER app giới hạn quyền của tiến trình ngay từ lúc khởi động. Nhờ đó, dù kẻ tấn công có khai thác được lỗ hổng, chúng cũng chỉ có quyền của user thông thường, giúp chặn đứng khả năng phá hoại hệ thống và leo thang đặc quyền. 
---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> *Câu trả lời của bạn*
Có thể đạt 20 request trong khoảng 2 giây: gửi 10 request ngay trước mốc phút mới, rồi 10 request ngay sau khi bộ đếm reset ở giây 00. Hai lô thuộc hai phút đồng hồ khác nhau nên đều nằm trong hạn mức 10/phút, dù tổng cộng 20 request đến gần như liên tiếp. Sliding window tránh lỗ hổng này vì đếm mọi request trong 60 giây gần nhất.
---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> *Câu trả lời của bạn*
Rate limit giới hạn số request trong một khoảng thời gian
Cost guard giới hạn tổng chi phí của một user trong tháng.
Tình huống 1: user gửi ít request, chưa vượt rate limit, nhưng chi phí dự kiến sẽ làm ngân sách tháng vượt mức, nên cost guard chặn.
Tình huống 2: user còn ngân sách, nhưng gửi request thứ 11 trong một phút khi giới hạn là 10, nên rate limit chặn.
---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> *Câu trả lời của bạn*
Redis mất kết nối, cả 3 container đều không ping được Redis.
Nếu /health cũng kiểm tra Redis, cả 3 trả 503.
Orchestrator hiểu đó là lỗi liveness và restart cả 3 container.
Redis vẫn mất trong 30 giây, nên container mới cũng tiếp tục trả 503 và có thể bị restart lặp lại; cụm mất khả năng phục vụ dù process vẫn sống.
Tách endpoint sẽ tránh việc này: /health chỉ kiểm tra process, còn /ready trả 503 để load balancer tạm ngừng gửi traffic.
---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> *Câu trả lời của bạn*
Với Redis dùng chung, mỗi request đọc được lịch sử từ mọi container; history_length thường tăng 2 mỗi lượt vì mỗi lần hỏi lưu một tin nhắn user và một tin nhắn assistant.
Nếu mỗi container giữ lịch sử trong dict riêng, request được điều hướng sang container khác sẽ không thấy lịch sử của container trước. Vì vậy history_length có thể lúc tăng, lúc giảm hoặc trở về 0, tùy request rơi vào instance nào
---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> *Câu trả lời của bạn*
