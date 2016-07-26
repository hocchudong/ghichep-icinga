## Monitor MySQL trong icinga2

Thông tin host MySQL

```
OS: CentOS 7
IP: 192.168.100.199
Service: MySQL or MariaDB
```
### Tạo user kiểm tra cho icinga2

Ở phần này chúng ta cần tạo một USER có quyền USAGE trên host DB (MySQL hoặc MariaDB).

```
GRANT usage ON *.* TO 'checker'@'%' IDENTIFIED BY 'password';
```

- `checker` : Tên của user dùng cho icinga2 có thể truy xuất vào DB
- `password` : Mật khẩu của `checker` (Tùy chọn)
- `%` : Anyhost - User này có thể đăng nhập từ bất kỳ host nào.

Như vậy là ta đã hoàn thành công việc ở host DB.

### Thêm plugin `check_mysql_health` vào thư mục plugin của icinga2

Tải plugin ở đây: http://abc.com

Sau khi tải về, ta phân quyền cho nó bằng lệnh

```
chmod 755 check_mysql_health
```

**Chú ý**: Plugin này làm việc cần có `perl` và `mysql-client`. Nếu chưa có, cài đặt chúng bằng lệnh sau:

```
yum install perl mysql-client -y
```

**Chú ý thứ 2**: là HOST DB phải bind-address:

Sửa file: `vi /etc/my.cnf` và thêm ở `[mysqld]`

```
[mysqld]
bind-address = 0.0.0.0
```

Ở một số distro, đường dẫn có thể bị thay đổi, vui lòng Google để tìm hiểu!

Chúng ta có thể kiểm tra plugin có hoạt động hay không bằng lệnh sau:

```
./check_mysql_health --hostname 192.168.100.199 --username checker --password password --mode connection-time
```

Kết quả trả về có dạng, cho biết plugin hoạt động ổn định:

```
OK - 0.03 seconds to connect as checker | connection_time=0.0343s;1;5
```

### Cấu hình icinga2

Đầu tiên, chúng ta thêm một `command` mới với nội dung:

```
vi \etc\icinga2\conf.d\commands.conf
```

```
# MySQL Health
object CheckCommand "mysql-health" {
	import "plugin-check-command"
	command = [ PluginDir + "/check_mysql_health"]
	arguments ={
	"--hostname" = "$hostname$"
	"--username" = "$username$"
	"--password" = "$password$"
	"--mode"	 = "$mode$"
	"--units"    = "$units$"
	}
}
```

Sau đó, chúng ta thêm một `host` khai báo thông tin về máy chủ DB.

Mở file `/etc/icinga2/conf.d/hosts.conf`

```
 object Host "DB-1" {
	import "generic-host"
	address = "192.168.100.199"
}
```

Lưu lại và tiếp tục thêm các dịch vụ cần check về máy chủ DB.

Mở file `/etc/icinga2/conf.d/services.conf`, và thêm:

```
apply Service "check-mysql-health" {
	import "generic-service"
	check_command = "mysql-health"
	vars.hostname = host.address
	vars.username = "checker"
	vars.password = "123"
	vars.mode = "connection-time" // Mode kiểm tra thời gian kết nối đến DB
	assign where host.name == "DB-1"
}

apply Service "check-mysql-threads-connected" {
	import "generic-service"
	check_command = "mysql-health"
	vars.hostname = host.address
	vars.username = "checker"
	vars.password = "123"
	vars.mode = "threads-connected" // Mode kiểm tra số USER kết nối đến DB
	vars.units = "%"
	assign where host.name == "DB-1"
}
```

Các mode khác xem tại đây: https://labs.consol.de/nagios/check_mysql_health/#command-line-parameters

Sau khi thêm các dịch vụ theo dõi DB mà bạn muốn, chúng ta lưu file `services.conf` và khởi động lại icinga2 để nó nhận những gì mình vừa cấu hình.

```
systemctl restart icinga2
```

Kiểm tra lại trên Web, **Overview > Host > MySQL-Health**

<img src="http://image.prntscr.com/image/570eccad1a2340e0b587e1489c95452c.png" />