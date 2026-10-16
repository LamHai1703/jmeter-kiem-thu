# Tài liệu tham khảo (sinh viên tự tìm thêm theo yêu cầu đề bài)

| # | Nguồn | Link | Dùng cho phần nào |
|---|---|---|---|
| 1 | Apache JMeter — User Manual (tài liệu chính thức) | https://jmeter.apache.org/usermanual/ | Tổng quan, cấu trúc Test Plan, các thành phần |
| 2 | Apache JMeter — Getting Started | https://jmeter.apache.org/usermanual/get-started.html | Cài đặt, chạy GUI / non-GUI, tham số dòng lệnh |
| 3 | Apache JMeter — Building a Test Plan | https://jmeter.apache.org/usermanual/build-test-plan.html | Cách tạo Thread Group, Sampler, Listener |
| 4 | Apache JMeter — HTML Dashboard Report | https://jmeter.apache.org/usermanual/generating-dashboard.html | Cách sinh và đọc HTML Dashboard (`-e -o`) |
| 5 | BlazeMeter — JMeter Tutorial | https://www.blazemeter.com/blog/jmeter-tutorial | Hướng dẫn trực quan, ví dụ Thread Group / Assertion |
| 6 | Guru99 — JMeter Tutorials | https://www.guru99.com/jmeter-tutorials.html | Khái niệm Performance Testing, cách đọc Summary Report |
| 7 | JSONPlaceholder — Guide | https://jsonplaceholder.typicode.com/guide/ | Tài liệu API demo dùng trong kịch bản 02 và 03 |
| 8 | JSONPlaceholder — Trang chủ | https://jsonplaceholder.typicode.com/ | Danh sách endpoints mẫu |

## Thuật ngữ cần nhớ

- **Thread Group**: nhóm người dùng ảo. `Number of Threads` = số user đồng thời, `Ramp-up` = thời gian để đủ số user đó, `Loop Count` = mỗi user lặp lại kịch bản mấy lần.
- **Sampler**: 1 request (ví dụ HTTP Request). Mỗi sampler = 1 lần gọi tới server.
- **Assertion**: điều kiện kiểm tra kết quả. Ví dụ: status phải bằng 200, body phải chứa `userId`.
- **Listener**: nơi xem kết quả (Summary Report, View Results Tree, Aggregate Report).
- **Throughput**: số request xử lý được mỗi giây (req/s) — càng cao càng tốt.
- **P95**: phân vị 95 — 95% request có thời gian phản hồi ≤ giá trị này. Dùng P95 thay vì trung bình để đánh giá trải nghiệm người dùng thực tế.
- **Error %**: tỉ lệ request fail (do HTTP lỗi hoặc do Assertion fail).
