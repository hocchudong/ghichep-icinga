# Tổng quan Icinga
## Lịch sử hình thành
- Tháng 5 năm 2009, 1 nhóm các nhà phát triển từ cộng đồng Nagios đã tự đứng ra phát triển 1 sản phẩm mới có tên Icinga khi họ thấy Nagios phát triển chậm.
- Năm đầu: họ phát hành các version tách lẻ (Core, API, Web) và có 10000 downloads.
- Năm 2: họ phát hành bản core và web thống nhất và ổn định.Hỗ trợ thêm IPv4/IPv6 dual-stack, tối ưu kết nối tới csdl, làm mới giao diện người dùng, tích hợp cộng đồng add-on # (PNP4Nagios ... )
- Năm 3: tăng lên 100000 download, tích hợp thành phần API -> Web, cải thiện báo cáo SLA, mở rộng thiết bị Icinga (Debian, OpenSuSe, Centos)
- 6/2014: Công bố bản chính thức đầu tiên của icinga2.
- Icinga phát triển dựa trên Nagios do đó nó có những đặc trưng có sẵn + SLA chính xác hơn + tạo kết nối CSDL cho PostgreSQL và Oracle + monitor dự phòng,
- Icinga vẫn duy trì cấu hình + plugin tương thích với Nagios để hỗ trợ chuyển từ Nagios -> Icinga

## Icinga là gì?
- Icinga là 1 hệ thống giám sát check hosts và các services mà bạn chỉ định hoặc thông báo cho bạn khi có lỗi gì đó và khi chúng được phục hồi.
- Nó có thể chạy trên nhiều Linux distributions (bao gồm Fedora, Ubuntu, openSUSE) cũng như các Unix platforms khác (bao gồm Solaris và HP).
- Được viết bằng C.
- Hệ thống có thể giám sát bất kì cái gì có kết nối mạng.
- Các đặc trưng của Icinga gồm:
<ul>
	<li>Giám sát các dịch vụ mạng (SMTP, POP3, HTTP, NNTP, PING ...)</li>
	<li>Giám sát tài nguyên của host (CPU load, disk usage ...)</li>
	<li>Thiết kế các plugin đơn giản cho phép người dùng dễ dàng phát triền cách check services theo ý của họ.</li>
	<li>Check các service 1 cách song song.</li>
	<li>Có thể định nghĩa hệ thống mạng các host bằng cách sử dụng các host "parent", cho phép bảo vệ và phân biệt giữa các host khi nó bị down và các host đó sẽ không bị động đến.</li>
	<li>Gửi thông báo khi service hoặc host gặp vấn đề hoặc được giải quyết vấn đề (qua email,sms hoặc tùy người dùng)</li>
	<li>Có thể định nghĩa cách xử lý sự kiện trong khi service đang chạy để có thể giải quyết vấn đề chủ động.</li>
	<li>Tự động quay vòng file log.</li>
	<li>Hỗ trợ cho việc thực hiện giám sát các host dư thừa.</li>
	<li>Giao diện web cổ điển để xem trạng thái mạng,lịch sử thông báo và vấn đề, file log...</li>
	<li>Giao diện web mới dựa trên Icinga Core, IDOUtils, API sử dụng GUI Web 2.0 mới mẻ hơn có thể show trạng thái hiện tại, thông tin lịch sử, sử dụng cronks và bộ lọc, tạo thông báo với hỗ trọ đa ngôn ngữ.</li>
</ul>


