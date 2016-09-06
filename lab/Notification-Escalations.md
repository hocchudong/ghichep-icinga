## Cảnh báo leo thang

Ở phần này chúng ta có thể hiểu nôm na rằng. Khi một host/service thay đổi trạng thái mail cảnh báo sẽ gửi tới một người hoặc một nhóm thứ nhất đã được đặt. Nếu quá khoảng thời gian chúng ta đặt cho người/nhóm thứ nhất, icinga2 sẽ gửi thông báo tới nhóm thứ 2. Khi host/service hoạt động trở lại, mail thông báo trạng thái UP sẽ được gửi tới cả 2 người/nhóm được gán.
### Thông tin host giám sát

```
OS: CentOS 6
eth0: 192.168.100.196
Service: http
```
### Cấu hình như sau:

#### Khai báo một host có tên `Web-Server-1`

```
vi /etc/icinga2/conf.d/hosts.conf
```

```
object Host "Web-Server-1" {
	import "generic-host"
	address = "192.168.100.196"
	check_command = "http"
}
```
#### Thêm user và group

Thêm các user và group mới vào trong file cấu hình:

```
vi \etc\icinga2\conf.d\users.conf
```

```
object User "hoangdh" {
	import "generic-user"
	display_name = "HoàngĐH"
	email = "hoangdh@example.com""
	groups = ["anninhxom"]
}
object User "lamkt" {
	import "generic-user"
	display_name = "LâmKT"
	email = "lamkt@example.com""
	groups = ["anninhxom"]
}
object User "congtt" {
  import "generic-user"
  display_name = "CongTO"
  groups = [ "anninhthon" ]
  email = "conto@example.com"
}
object UserGroup "anninhxom" {
	display_name = "An Ninh Xóm"
}
object UserGroup "anninhthon" {
	display_name = "An Ninh Thôn"
}
```

Lưu lại và tiếp tục cấu hình cảnh báo leo thang trong file:

```
vi \etc\icinga2\conf.d\notifications.conf
```

Nếu giám sát host, chúng ta thay **Service** **bằng Host**

**Lưu ý:** Thời gian **end** của nhóm 1 phải bằng thời gian **begin** của nhóm 2

```
apply Notification "Nhom-support-1" to Service {
		import "mail-service-notification"
		user_groups = ["anninhxom"]
		times = {
			begin = 1m
			end = 5m
		}
		assign where host.address == "192.168.100.196"
}
	apply Notification "Nhom-support-2" to Service {
		import "mail-service-notification"
		user_groups = ["anninhthon"]
		times = {
			begin = 5m
			end = 10m
		}
		assign where host.address == "192.168.100.196"
}
```

Lưu file và khởi động lại dịch vụ icinga2

```
systemctl restart icinga2
```

Kiểm tra trên Web Icinga2: **Overview > Services**

<img src="http://image.prntscr.com/image/97ddc7354bfa4f2698dc9766c7e1db42.png" />


Để kiểm tra, chúng ta tạm ngưng dịch vụ `http` trên host `Web-Server-1` đi.

Sau khoảng thời gian chưa đến 1 phút, icinga2 đã gửi đến **Nhom-support-1**:

<img src="http://image.prntscr.com/image/c325a58c9a8348c598b36f1b0024b1e9.png"/>

Xem lại thông tin gửi mail trên icinga2 **Reporting > Alert Summary**

<img src="http://image.prntscr.com/image/9c0b02863d8745788636d0b0d96e22ab.png" />

Chúng ta thấy sau khoảng thời gian ~5p, service không OK thì sẽ gửi đến **Nhom-support-2**. Và sau khi service up, sẽ có mail gửi đến cả 2 nhóm.