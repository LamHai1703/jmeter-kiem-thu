# Hướng dẫn chạy và chụp ảnh minh hoạ

## A. Cài đặt

### Windows (máy sinh viên thường dùng)

1. Cài **Java 17** (hoặc Java 21):
   - Tải tại https://adoptium.net/ → chọn Temurin 17 → cài đặt.
   - Kiểm tra: mở Command Prompt / PowerShell, gõ `java -version` → phải hiện `17.x` trở lên.
2. Tải **Apache JMeter 5.6.3**:
   - https://dlcdn.apache.org/jmeter/binaries/apache-jmeter-5.6.3.zip
   - Giải nén, ví dụ vào `C:\apache-jmeter-5.6.3`.
3. Mở JMeter: chạy `C:\apache-jmeter-5.6.3\bin\jmeter.bat` (đợi vài giây để GUI hiện lên).

### macOS / Linux

```bash
java -version   # cần Java 17+
# Tải và giải nén apache-jmeter-5.6.3.tgz rồi chạy:
./bin/jmeter
```

## B. Chạy bằng GUI (để chụp ảnh)

1. `File > Open` → chọn `test-plans/01-web-load-test.jmx` (làm tương tự cho 02, 03).
2. Nhấn nút ▶ **Start** trên thanh công cụ (hoặc `Run > Start`, phím tắt `Ctrl + R`).
3. Đợi chạy xong (góc phải dưới hiện `0 / 10` → về 0, icon không còn xoay).
4. Nhấn vào từng Listener để xem kết quả:
   - **Summary Report**: bảng tổng hợp Samples / Average / Min / Max / Error %.
   - **View Results Tree**: danh sách từng request, icon xanh = pass, đỏ = fail. Nhấn vào 1 request → xem tab `Sampler result` + `Response data`.

## C. Chụp 4 ảnh GUI bắt buộc

Lưu vào thư mục `screenshots/` với đúng tên sau:

| Tên file | Cách chụp |
|---|---|
| `gui-01-test-plan.png` | Chụp toàn bộ cửa sổ JMeter khi vừa mở file `.jmx`, thấy rõ cây Test Plan bên trái (Test Plan → Thread Group → Sampler → Listener). |
| `gui-02-thread-group.png` | Nhấn vào Thread Group, chụp panel bên phải thấy rõ `Number of Threads`, `Ramp-up period`, `Loop Count`. |
| `gui-03-summary-report.png` | Sau khi chạy xong, nhấn vào `Summary Report`, chụp bảng kết quả (phải thấy cột Error % = 0.00%). |
| `gui-04-view-results-tree.png` | Nhấn vào `View Results Tree`, chụp danh sách request có icon xanh + phần Response Data của 1 request. |

Mẹo: dùng `Win + Shift + S` (Windows) hoặc `Cmd + Shift + 4` (macOS) để chụp vùng chọn.

## D. Chạy bằng dòng lệnh (Non-GUI) — để lấy file .jtl + HTML Dashboard

```bash
# Từ thư mục gốc của repo:
jmeter -n -t test-plans/01-web-load-test.jmx -l results/01-web-load-test.jtl -e -o results/01-web-load-test-dashboard
jmeter -n -t test-plans/02-api-functional-test.jmx -l results/02-api-functional-test.jtl -e -o results/02-api-functional-test-dashboard
jmeter -n -t test-plans/03-load-stress-test.jmx -l results/03-load-stress-test.jtl -e -o results/03-load-stress-test-dashboard
```

Lưu ý:
- Nếu thư mục `results/*-dashboard` đã tồn tại, JMeter sẽ báo lỗi. Xoá thư mục cũ hoặc đổi tên output trước khi chạy lại.
- Trên Windows, lệnh là `jmeter.bat` thay vì `jmeter`.

## E. Xem kết quả

- Mở file `results/<ten>-dashboard/index.html` bằng trình duyệt để xem Dashboard của JMeter.
- Mở file `results/<ten>.jtl` bằng Excel / VS Code để xem chi tiết từng request (cột `success`, `failureMessage` cho biết request nào fail và vì sao).
