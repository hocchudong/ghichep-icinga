## Cảnh báo leo thang

Ở phần này chúng ta có thể hiểu nôm na rằng. Khi một host/service thay đổi trạng thái mail cảnh báo sẽ gửi tới một người hoặc một nhóm thứ nhất đã được đặt. Nếu quá khoảng thời gian chúng ta đặt cho người/nhóm thứ nhất, icinga2 sẽ gửi thông báo tới nhóm thứ 2. Khi host/service hoạt động trở lại, mail thông báo trạng thái UP sẽ được gửi tới cả 2 người/nhóm được gán.
### Thông tin host giám sát

```
OS: CentOS 6
eth0: 192.168.100.196
Service: http, ssh
```
### Cấu hình như sau:

#### Thêm user và group

Thêm các user và group mới vào trong file cấu hình:

```
vi \etc\icinga2\conf.d\users.conf
```

```
object User "hoangdh" {
	import "generic-user"
	display_name = "HoàngĐH"
	email = "daohuyhoang87@gmail.com"
	groups = ["anninhxom"]
}
object User "lamkt" {
	import "generic-user"
	display_name = "LâmKT"
	email = "kieutunglam@gmail.com"
	groups = ["anninhxom"]
}
object User "congtt" {
  import "generic-user"
  display_name = "CongTO"
  groups = [ "anninhthon" ]
  email = "tcvn1985@gmail.com"
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

```
// Nếu giám sát host, chúng ta thay **Service** **bằng Host**
// Lưu ý: Thời gian **END** của nhóm 1 phải bằng thời gian **BEGIN** của nhóm 2
apply Notification "Nhom-support-1" to **Service** {
		import "mail-service-notification"
		user_groups = ["anninhxom"]
		times = {
			begin = 1m
			**end = 5m**
		}
		assign where host.address == "192.168.100.196"
}
	apply Notification "Nhom-support-2" to **Service** {
		import "mail-service-notification"
		user_groups = ["anninhthon"]
		times = {
			**begin = 5m**
			end = 10m
		}
		assign where host.address == "192.168.100.196"
}
```