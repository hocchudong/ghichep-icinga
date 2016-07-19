## I. Giám sát cơ bản

###1 Host và Service
Icinga2 có thể được dùng để giám sát khả năng sẵn sàng của các host và service. Các host và service có thể là :

	-	Network service (HTTP, SMTP, SNMP, SSH...)
	-	Printer
	-	Switch/Router
	-	Cảm biến nhiệt độ
	-	Các service local hoặc có thể truy cập qua mạng

Các object về host cung cấp 1 cơ chế tới nhóm dịc vụ đang chạy trên cùng 1 thiết bị vật lý.

Ví dụ, muốn giám sát 1 số service của 1 host, ta khai báo thêm trong file /etc/icinga2/conf.d/hosts.conf : 
```sh
object Host "manh-graphite" {
  address = "172.16.69.236"
  check_command = "hostalive"
}

object Service "ping4" {
  host_name = "manh-graphite"
  check_command = "ping4"
}

object Service "http" {
  host_name = "manh-graphite"
  check_command = "http"
}
```
Ví dụ trên sẽ tạo 1 object **Host**, thuộc tính **address** sẽ được dùng để xác định rằng địa chỉ nào sẽ được gắn với object 
**Host** này. Việc kiểm tra host sử dụng check command là **hostalive**

2 object được tạo sau đó là **ping4** và **http** thuộc về host có trong **host_name**.

###2 Các trạng thái của Host và service
####2.1 Các trạng thái của Host
 - UP : Host khả dụng
 - DOWN : Host không khả dụng
 
####2.2 Các trạng thái của Service

 - OK : Service đang hoạt động tốt
 - WARNING : Service đang có 1 số vấn đề nhưng vẫn trong tình trạng làm việc.
 - CRITICAL : Service đang trong trạng thái nguy hiểm
 - UNKNOW : Không thể kiểm tra trạng thái của Service
 
####2.3 Trạng thái Hard và Soft 
<ul>
<li>Khi có vấn đề xảy ra với host/service, Icinga2 sẽ không lập tức gửi cảnh báo đi ngay, mà sẽ tiến hành kiểm tra lại trước 
khi gửi cảnh báo, nhằm tránh việc những cảnh báo không quan trọng được gửi đi liên tục.</li>
<li>Trong thời gian kiểm tra này, object rơi vào trạng thái **SOFT**</li>
<li>Sau khi tất cả re-check được tiến hành, và object được kiểm tra vẫn trong trạng thái non-OK, thì host/service sẽ chuyển 
sang trạng thái **HARD** và cảnh báo được gửi đi.
![icinga](/images/icinga-01.png)
<li>Để thay đổi thời gian và số lần kiểm tra, hãy tìm đến /etc/icinga2/conf.d/templates.conf
![icinga](/images/icinga-02.png)

###3. Template
Icinga2 cung cấp sẵn các template chứa các thông số cơ bản cho host, service, hay việc gửi cảnh báo. Ta cũng có thể tự tạo
 template để tùy chỉnh theo nhu cầu (như việc thay đổi thời gian và số lần re-check ở ví dụ trên), và import template mới
 vào các object.
![icinga](/images/icinga-03.png)
![icinga](/images/icinga-04.png)