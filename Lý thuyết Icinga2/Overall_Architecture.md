#Kiến trúc tổng thể của Icinga
<img src="http://i.imgur.com/whsMvep.png" />

- Icinga được viết trong C và có kiến trúc kiểu module với 1 core riêng biệt, giao diện người dùng và CSDL để người dùng có thể tích hợp các add-on và plug-in

## Icinga Core
### Icinga 1.x
- Thằng này dựa trên Nagios.
- Quản lý các giám sát tác vu, nhận các kiểm tra kết quả từ các plug-in.
- Sau đó kết nối những kết quả này tới IDODB (icinga Data Out Database) qua giao diện IDOMOD (Icinga Data Out Module) và daemon IDO2DB (Icinga Data Out to Database) over SSL mã hóa TCP sockets.
- Mặc dù 2 thằng trên được đóng gói thành IDOUtils ở trong Core, nhưng nó vẫn là 2 thành phiên riêng biệt, có thể được tách ra để phân phối dữ liệu và làm việc với nhiều server trong trường hợp Giám sát các hệ thống phân phối.
- Giao diện người dùng cũ của Icinga cũng được đóng gói trong Core.

### Icinga2
- Thằng này có thêm các tính năng mới.
- Quản lý các giám sát tác vụ, running checks, gửi thông báo về cảnh báo.
- Có các feature mặc định như "checker", "notification".Các feature có thể được enable theo yêu cầu.
- Tích hợp được với các phần mềm ghi log như logstash, graylog ...
- HỖ trợ hầu hết các distribution chính.
- CLI mạnh mẽ.
- Bao gồm các thư viện mẫu sâu rộng.
- Giao diện bên ngoài tương thích với Icinga1.x và chính giao diện người dùng của nó.
- Được xây dựng trên cluster stack bảo mật bởi chứng chỉ SSL x509 giúp cho việc cài đặt các giám sát phân phối dễ hơn.
- Cú pháp cấu hình cũng # so với Icinga1.x và Nagios/

## Icinga's User Interfaces
### Icinga Classic UI (Classic Web)
- Dựa trên Nagios và giữ lại format của nó.
- Thêm các tính năng mới như đánh số trang, ouput JSON và export CSV.
- Được tích hợp sẵn trong Core (Icinga1.x).
- Nhận dữ liệu từ cache và gửi câu lệnh qua pipe tới các file lệnh.

### Icinga Web (New Web)
- Dựa trên Agavi và PHP.
- Sử dụng Cronks (widget) => kéo thả để tùy chỉnh dashboard.
- Khác với thằng Classic, thằng này đứng riêng biệt.
- Nó kết nối tới core, CSDL và các add-on bên thành phần thứ 3.
- Nó có 3 lớp: lớp trừu tượng Doctrine (đầu vào/CSDL), lớp REST API (chạy các script bên ngoài) và lớp Command Control (thực thi các câu lệnh).
- Cả 2 thằng đều hiện thị thông tin trên host và service về: trạng thái, lịch sử, thông báo, biểu đồ trạng thái của hệ thống mạng trong thời gian thực.Đều hỗ trợ IPv4 + IPv6.

## Icinga Data Out Database
- Nơi lưu trữ dữ liệu về lịch sử giám sát để các add-on hoặc giao diện Web truy cập vào.
- Khác với Nagios, thằng này hỗ trợ thêm CSDL PostgreSQL và Oracle.
