## Phần 1. Phân tích

**Input:** tên miền phong-dieu-phoi.greenlog.vn
**Output:** địa chỉ ip của máy chủ

**Sơ đồ luồng dữ liệu dns:**

nam gõ tên miền → trình duyệt kiểm tra cache → hệ điều hành kiểm tra cache và file hosts → dns resolver của isp → root server (chỉ sang server .vn) → tld server .vn (chỉ sang server của greenlog.vn) → authoritative server của greenlog.vn (trả về ip) → resolver trả ip về trình duyệt → trình duyệt kết nối tới ip đó

**Phân biệt lỗi dns và lỗi 500:**

- lỗi dns (DNS_PROBE_FINISHED_NXDOMAIN): không tìm được ip của tên miền, request chưa hề gửi tới máy chủ, nên lỗi nằm ở phía client (gõ sai tên miền, dns lỗi)
- lỗi 500: đã tìm được ip, request đã tới máy chủ, nhưng máy chủ xử lý bị lỗi, nên lỗi nằm ở phía server

**Hậu quả khi gõ sai grenlog.vn:**

trong "danh bạ internet" không có tên grenlog.vn nên không có ip trả về, trình duyệt báo NXDOMAIN. máy chủ công ty không bị sập, nam nói sai.

## Phần 2. Triển khai

quy trình kiểm tra:

1. kiểm tra lại tên miền đã gõ đúng chưa
2. dùng lệnh `nslookup phong-dieu-phoi.greenlog.vn` xem có ra ip không
3. nếu không ra ip thì là lỗi dns, chưa có status code, request chưa tới server
4. nếu ra ip thì dùng `curl -I http://phong-dieu-phoi.greenlog.vn` hoặc mở f12 tab network xem status code
5. có status code trả về nghĩa là request đã chạm tới máy chủ

ý nghĩa mã: 2xx thành công, 3xx chuyển hướng, 4xx lỗi từ phía client, 5xx lỗi từ phía server. nếu không có mã nào hoặc bị timeout thì chưa tới được server.

## Phần 3. Kiểm thử

nam gõ đúng tên miền mà vẫn bị 404 thì dns đã ok, request đã tới máy chủ và máy chủ vẫn chạy bình thường. vấn đề là máy chủ không tìm thấy trang hoặc tài nguyên được yêu cầu. nguyên nhân có thể là sai đường dẫn url, trang bị xóa hoặc đổi tên, hoặc cấu hình web server sai.