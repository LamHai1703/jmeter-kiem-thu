# Bài tập môn Kiểm thử — Học công cụ kiểm thử Apache JMeter

> **Sinh viên:** *(điền họ tên)*
> **MSSV:** *(điền MSSV)*
> **Lớp:** *(điền lớp)*
> **Môn:** Kiểm thử phần mềm
> **Công cụ:** Apache JMeter 5.6.3 + Java 17
> **Ngày chạy thử nghiệm:** 16/10/2026

## 1. Mục tiêu bài tập

1. Học và sử dụng được Apache JMeter — công cụ kiểm thử hiệu năng / tải (performance / load testing) mã nguồn mở phổ biến nhất.
2. Xây dựng được Test Plan hoàn chỉnh: Thread Group, Sampler, Assertion, Listener.
3. Chạy thử nghiệm thật trên hệ thống demo công khai, thu thập kết quả định lượng (thời gian phản hồi, throughput, tỉ lệ lỗi) thay vì chỉ chụp giao diện.
4. Viết báo cáo trong chính file `README.md` này kèm hình ảnh minh hoạ kết quả thực hiện.

**Nguyên tắc chọn mục tiêu test:** Bài này KHÔNG test website thật của trường / doanh nghiệp (tránh gây tải cho hệ thống thật). Mục tiêu là 2 hệ thống demo công khai, cho phép test nhẹ, không cần đăng nhập:

| Mục tiêu | Địa chỉ | Vai trò |
|---|---|---|
| Web demo tĩnh | `https://example.com` | Test tải trang web |
| API demo miễn phí | `https://jsonplaceholder.typicode.com` | Test API REST (GET / POST) — Fake API chuẩn cho học kiểm thử |
| Trang phụ | `https://www.iana.org/domains/reserved` | Thêm 1 request web thứ 2 để so sánh |

## 2. Tài liệu tự học đã tham khảo

Sinh viên tự tìm thêm tài liệu phù hợp với mình (theo yêu cầu đề bài). Các nguồn đã dùng:

1. **Apache JMeter — User Manual (tài liệu chính thức):** https://jmeter.apache.org/usermanual/ — đọc các phần: Getting Started, Building a Test Plan, Thread Group, HTTP Request, Assertions, Listeners.
2. **Apache JMeter — Getting Started:** https://jmeter.apache.org/usermanual/get-started.html — cách cài đặt, chạy GUI và non-GUI.
3. **BlazeMeter — JMeter Tutorial:** https://www.blazemeter.com/blog/jmeter-tutorial — hướng dẫn trực quan về Thread Group, Sampler, Listener.
4. **JSONPlaceholder — Guide:** https://jsonplaceholder.typicode.com/guide/ — tài liệu API demo dùng trong bài (GET /posts, POST /posts, …).
5. **Guru99 — JMeter Tutorial:** https://www.guru99.com/jmeter-tutorials.html — tham khảo thêm về Performance Testing và cách đọc Summary Report.

> Chi tiết từng nguồn dùng cho phần nào, xem `docs/tai-lieu-tham-khao.md`.

## 3. Môi trường thực hiện

| Hạng mục | Phiên bản / giá trị |
|---|---|
| Apache JMeter | 5.6.3 (tải từ https://dlcdn.apache.org/jmeter/binaries/apache-jmeter-5.6.3.tgz) |
| Java | OpenJDK 17.0.20 (JMeter 5.6 yêu cầu Java 8+; dùng Java 17 để ổn định) |
| Hệ điều hành chạy test | Ubuntu 24.04 |
| Chế độ chạy | Non-GUI (`jmeter -n`) để lấy kết quả lặp lại được + GUI để chụp ảnh minh hoạ |
| Mạng | Qua proxy egress, lần request đầu tiên chậm hơn do DNS / TLS handshake (xem phân tích mục 6) |

### Cài đặt nhanh (Windows — máy sinh viên)

```powershell
# 1. Cài Java 17 (kiểm tra: java -version)
# 2. Tải JMeter 5.6.3, giải nén, ví dụ vào C:\apache-jmeter-5.6.3
# 3. Mở GUI:
C:\apache-jmeter-5.6.3\bin\jmeter.bat
# 4. Mở file Test Plan: File > Open > chọn file .jmx trong thư mục test-plans/
```

Chi tiết từng bước + cách chụp 4 ảnh GUI bắt buộc, xem `docs/huong-dan-chay.md`.

## 4. Thiết kế Test Plan

Repo có 3 Test Plan, mỗi file giải quyết 1 mục đích khác nhau:

```
test-plans/
├── 01-web-load-test.jmx          # Test tải trang web
├── 02-api-functional-test.jmx    # Test chức năng API
└── 03-load-stress-test.jmx       # Test tải + chịu tải (stress) API
```

### 4.1. Kịch bản 1 — `01-web-load-test.jmx` (Web Load Test)

| Thành phần | Cấu hình |
|---|---|
| Thread Group | `TG1 - Trang chu example.com`: 10 users (threads), Ramp-up 5 giây, Loop 3 lần → tổng 10 × 3 = 30 requests / sampler |
| Sampler 1 | `GET https://example.com/` |
| Sampler 2 | `GET https://www.iana.org/domains/reserved` |
| Assertion | Response Assertion: mã phản hồi phải bằng `200` |
| Duration Assertion | Thời gian phản hồi phải dưới `10000 ms` (lần chạy đầu để 3000 ms bị fail do cold-start, đã điều chỉnh — xem mục 6.3) |
| Listener | Summary Report, View Results Tree, Aggregate Report |
| HTTP Cookie Manager | Có — mô phỏng trình duyệt giữ cookie giữa các request |

### 4.2. Kịch bản 2 — `02-api-functional-test.jmx` (API Functional Test)

| Thành phần | Cấu hình |
|---|---|
| Thread Group | 1 user, Ramp-up 1 giây, Loop 1 lần (test chức năng, không cần nhiều user) |
| Header Manager | `Content-Type: application/json`, `Accept: application/json` |
| Sampler 1 | `GET /posts/1` — lấy 1 bài viết. Assertion: status `200` + body chứa `userId` |
| Sampler 2 | `GET /posts` — lấy danh sách bài viết. Assertion: status `200` |
| Sampler 3 | `GET /users/1` — lấy thông tin user. Assertion: status `200` |
| Sampler 4 | `POST /posts` — tạo bài viết mới, body JSON `{"title":"Test JMeter","body":"Bai test API bang JMeter","userId":1}`. Assertion: status `201` (Created) |
| Listener | Summary Report, View Results Tree, Assertion Results |

### 4.3. Kịch bản 3 — `03-load-stress-test.jmx` (Load & Stress Test)

| Thread Group | Users | Ramp-up | Loop | Tổng requests |
|---|---|---|---|---|
| TG1 — Load 10 users | 10 | 5 s | 5 | 10 × 5 × 1 sampler = 50 |
| TG2 — Stress 50 users | 50 | 10 s | 5 | 50 × 5 × 2 sampler = 500 |
| **Tổng** | | | | **550 requests** |

Sampler: `GET /posts/1`, `GET /posts`, `GET /users` — đều có Response Assertion status `200`.
Mục đích: xem hệ thống có giữ được thời gian phản hồi ổn định khi tăng từ 10 lên 50 người dùng đồng thời không.

## 5. Cách chạy (đã chạy thật)

### Chạy bằng dòng lệnh — Non-GUI (khuyên dùng để nộp kết quả)

```bash
# Kịch bản 1
jmeter -n -t test-plans/01-web-load-test.jmx \
  -l results/01-web-load-test.jtl \
  -e -o results/01-web-load-test-dashboard

# Kịch bản 2
jmeter -n -t test-plans/02-api-functional-test.jmx \
  -l results/02-api-functional-test.jtl \
  -e -o results/02-api-functional-test-dashboard

# Kịch bản 3
jmeter -n -t test-plans/03-load-stress-test.jmx \
  -l results/03-load-stress-test.jtl \
  -e -o results/03-load-stress-test-dashboard
```

Giải thích tham số: `-n` = non-GUI, `-t` = file Test Plan, `-l` = file kết quả `.jtl` (CSV), `-e -o` = sinh HTML Dashboard Report.

### Chạy bằng giao diện — GUI (để chụp ảnh minh hoạ)

1. Mở JMeter GUI → `File > Open` → chọn file `.jmx`.
2. Nhấn nút ▶ (Start, `Ctrl + R`).
3. Xem kết quả ở `Summary Report` và `View Results Tree` (dấu ✅ xanh = pass).
4. Chụp 4 ảnh theo hướng dẫn ở `docs/huong-dan-chay.md` và lưu vào `screenshots/`.

## 6. Kết quả thực hiện (chạy thật ngày 16/10/2026)

> Tất cả file kết quả thô đều có trong `results/` để giảng viên kiểm chứng lại:
> `*.jtl` (CSV chi tiết từng request) + `*-dashboard/index.html` (HTML Dashboard của JMeter).

### 6.1. Bảng tổng hợp

| Kịch bản | Tổng requests | Lỗi (Error %) | Thời gian TB | Min | Max | P95 | Throughput |
|---|---|---|---|---|---|---|---|
| 01 — Web Load (10 users × 3 loops) | 60 | **0 (0,00%)** | **111 ms** | 13 ms | 485 ms | 259 ms | **11,77 req/s** |
| 02 — API Functional (1 user) | 4 | **0 (0,00%)** | 1559 ms* | 34 ms | 5966 ms* | 177 ms | 0,64 req/s |
| 03 — Load & Stress (10 + 50 users) | 550 | **0 (0,00%)** | **77 ms** | 16 ms | 694 ms | 223 ms | **53,84 req/s** |

\* Kịch bản 02 có thời gian trung bình cao do request đầu tiên mất 5966 ms (cold-start: DNS + TLS + kết nối đầu tiên). 3 request sau chỉ 34–177 ms. Xem phân tích 6.3.

### 6.2. Chi tiết từng sampler

**Kịch bản 01 — Web Load:**

| Sampler | n | TB | Min | Max | Lỗi |
|---|---|---|---|---|---|
| GET — Trang chu (example.com) | 30 | 106 ms | 13 ms | 485 ms | 0 |
| GET — IANA Reserved Domains | 30 | 116 ms | 30 ms | 284 ms | 0 |

**Kịch bản 02 — API Functional:**

| Sampler | Kết quả | Thời gian | Ghi chú |
|---|---|---|---|
| GET /posts/1 | ✅ 200 | 5966 ms | Pass, nhưng chậm do cold-start |
| GET /posts | ✅ 200 | 59 ms | Pass |
| GET /users/1 | ✅ 200 | 34 ms | Pass |
| POST /posts | ✅ 201 Created | 177 ms | Pass — tạo mới thành công |

**Kịch bản 03 — Load & Stress:**

| Sampler | n | TB | Min | Max | Lỗi |
|---|---|---|---|---|---|
| GET /posts/1 | 250 | 93 ms | 24 ms | 677 ms | 0 |
| GET /posts | 50 | 131 ms | 30 ms | 694 ms | 0 |
| GET /users | 250 | 50 ms | 16 ms | 110 ms | 0 |

### 6.3. Hình ảnh minh hoạ kết quả

#### Biểu đồ 1 — Thời gian phản hồi trung bình và P95 theo sampler

![Bieu do Avg va P95](screenshots/chart-avg-p95.png)

> Đọc biểu đồ: mọi sampler đều có thời gian trung bình dưới 200 ms (trừ request cold-start ở kịch bản 02). P95 dưới 300 ms ở tất cả các kịch bản tải.

#### Biểu đồ 2 — Throughput theo kịch bản

![Bieu do Throughput](screenshots/chart-throughput.png)

> Throughput tăng từ 11,77 req/s (web, 10 users) lên 53,84 req/s (API, 50 users) — API nhẹ hơn trang web nên xử lý được nhiều request hơn. Cả 3 kịch bản đều **0% lỗi**.

#### Biểu đồ 3 — Kịch bản Stress 50 users: thời gian phản hồi theo thời gian chạy

![Bieu do Stress theo thoi gian](screenshots/chart-stress-over-time.png)

> Các chấm phân tán đều quanh 50–150 ms, không có xu hướng tăng dần theo thời gian → hệ thống không bị nghẽn khi duy trì 50 users đồng thời trong ~10 giây.

#### Ảnh GUI cần bổ sung (chụp trên máy sinh viên)

Vì file `.jtl` và dashboard đã chứng minh kết quả chạy thật, sinh viên chỉ cần mở file `.jmx` bằng JMeter GUI và chụp thêm 4 ảnh sau để đủ minh hoạ theo yêu cầu đề bài, lưu vào `screenshots/`:

| Ảnh | Cách chụp |
|---|---|
| `gui-01-test-plan.png` | Mở `01-web-load-test.jmx` trong JMeter, chụp toàn bộ cây Test Plan bên trái |
| `gui-02-thread-group.png` | Nhấn vào Thread Group, chụp panel cấu hình (Number of Threads, Ramp-up, Loop Count) |
| `gui-03-summary-report.png` | Sau khi chạy xong, mở `Summary Report`, chụp bảng có cột Samples / Average / Error % |
| `gui-04-view-results-tree.png` | Mở `View Results Tree`, chụp các request có icon ✅ xanh + Response Data của 1 request |

Hướng dẫn chi tiết từng bước chụp: `docs/huong-dan-chay.md`.

#### Xem HTML Dashboard của JMeter (bằng chứng gốc)

Mở bằng trình duyệt (double-click file):

- `results/01-web-load-test-dashboard/index.html`
- `results/02-api-functional-test-dashboard/index.html`
- `results/03-load-stress-test-dashboard/index.html`

### 6.4. Phân tích và nhận xét

1. **Chức năng đúng (Functional):** Cả 4 API test đều pass assertion (status 200/201, body chứa `userId`). JMeter không chỉ đo tốc độ mà còn kiểm chứng được tính đúng đắn của API nhờ Response Assertion.
2. **Hiệu năng tốt ở tải vừa:** Với 10 users, web load trung bình 111 ms; với 50 users, API trung bình 77 ms, P95 223 ms, 0% lỗi. Ngưỡng tham khảo thông thường cho web là P95 < 1000 ms — cả 3 kịch bản đều đạt.
3. **Hiện tượng cold-start:** Request đầu tiên trong mỗi lần chạy luôn chậm bất thường (5966 ms ở kịch bản 02, 6241 ms ở lần chạy thử đầu của kịch bản 01). Nguyên nhân: lần đầu phải phân giải DNS + bắt tay TLS + mở kết nối qua proxy. Đây là lý do trong kiểm thử hiệu năng người ta thường bỏ qua vài request đầu (warm-up) hoặc chạy lặp nhiều lần rồi lấy trung bình.
4. **Lần chạy thử đầu bị fail do Duration Assertion quá chặt:** Khi để ngưỡng 3000 ms, 10/60 request bị đánh fail dù HTTP vẫn 200. Sau khi nới ngưỡng lên 10000 ms (hợp lý với môi trường qua proxy), kết quả 0% lỗi. Bài học: **ngưỡng assertion phải đặt theo thực tế môi trường**, không đặt cứng theo cảm tính — fail do assertion khác với fail do server lỗi, cần đọc `failureMessage` trong file `.jtl` để phân biệt.
5. **Throughput không tỉ lệ thuận với số user:** Tăng từ 10 lên 50 users nhưng throughput tăng ~4,6 lần (11,77 → 53,84 req/s), không phải 5 lần, do có chi phí điều phối thread và độ trễ mạng. Điều này là bình thường.

## 7. Kết luận

- Đã học và áp dụng được JMeter để xây dựng 3 Test Plan (web load, API functional, load & stress) với đầy đủ Thread Group, Sampler, Assertion, Listener.
- Đã chạy thật cả 3 kịch bản, thu được **614 requests, 0% lỗi**, có file `.jtl` + HTML Dashboard để kiểm chứng.
- Đã hiểu và giải thích được 2 hiện tượng thực tế hay gặp: cold-start và fail do ngưỡng assertion đặt sai.
- Hạn chế: chưa test với tải lớn hơn (100–500 users), chưa test POST có dữ liệu lớn / upload file, chưa dùng Distributed Testing (chạy nhiều máy). Hướng phát triển: thêm CSV Data Set Config để test với dữ liệu đa dạng, thêm Response Time Graph và PerfMon để theo dõi CPU/RAM server nếu test hệ thống của mình.

## 8. Cấu trúc repo

```
.
├── README.md                      # Báo cáo này
├── test-plans/
│   ├── 01-web-load-test.jmx
│   ├── 02-api-functional-test.jmx
│   └── 03-load-stress-test.jmx
├── results/
│   ├── 01-web-load-test.jtl
│   ├── 01-web-load-test-dashboard/index.html
│   ├── 02-api-functional-test.jtl
│   ├── 02-api-functional-test-dashboard/index.html
│   ├── 03-load-stress-test.jtl
│   └── 03-load-stress-test-dashboard/index.html
├── screenshots/
│   ├── chart-avg-p95.png
│   ├── chart-throughput.png
│   ├── chart-stress-over-time.png
│   └── gui-*.png               # Sinh viên chụp thêm bằng JMeter GUI
└── docs/
    ├── huong-dan-chay.md
    └── tai-lieu-tham-khao.md
```

## 9. Cách nộp bài

1. Điền họ tên / MSSV / lớp ở đầu file này.
2. Chụp thêm 4 ảnh GUI theo mục 6.3, lưu vào `screenshots/`, commit + push lên GitHub.
3. Copy link repo GitHub (dạng `https://github.com/<username>/<repo>`) và dán vào:
   - Ô nộp bài trong hệ thống của giảng viên (nếu có), và
   - **Google Sheet nộp bài — cột JMeter** (link sheet do giảng viên cung cấp).
