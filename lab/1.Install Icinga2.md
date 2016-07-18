# Các bước để cài đặt Icinga2 trên Centos7
## Mục lục 
[1.Giới thiệu và chú thích] (#1)

[2. Install Icinga2 ] (#2)

[3. Install icinga2-ido-mysql] (#3)

[4. Install and Configure icingaweb2] (#4)

[5. Tham Khảo] (#5)

<a name="1"></a>
### 1.Giới thiệu và chú thích
- Prepare LAB enviroment: VMWare.
- A Centos 7 minimal server system
- Trong bài viết này tôi sử dụng hostname ktlam, IP address:192.168.10.10/24.
- Database được sử dụng sẽ là mariadb và sẽ được cài đặt cùng với host.

<a name="1"></a>
### 1. Install Icinga2
- Trước khi bắt đầu vào cài đặt Icinga2, bạn phải cài đặt các gói httpd, PHP, MySQL, phpmyAdmin, php-ldap.
- Các tham số tôi set như sau: 
<ul>
	<li>ServerName(/etc/httpd/conf/httpd.conf): www.ktlam.com (</li>
	<li>date.timezone(/etc/php.ini): Asia/HCM</li>
</ul>
- Set selinux
```sh
Set selinux=permissive : sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config
```
- Bắt đầu install icinga2.
```sh
rpm -i https://packages.icinga.org/epel/7/release/noarch/icinga-rpm-release-7-1.el7.centos.noarch.rpm
yum -y install icinga2
systemctl enable icinga2
systemctl start icinga2
```

<a name="2"></a>
### 2. Install icinga2-ido-mysql
- Tiếp theo tạo database cho icinga2: `mysql -u root -p`
```sh
create database icinga;
grant select, insert, update, delete, drop, create view, index, execute on icinga.* to 'icinga'@'localhost' identified by 'pass4icinga';
flush privileges;
exit
```
- Install icinga2-ido-mysql *database cho icinga2*: `yum -y install icinga2-ido-mysql`

- Populate database
```sh
mysql -u root -p icinga < /usr/share/icinga2-ido-mysql/schema/mysql.sql
```
- chỉnh sửa ido-mysql.conf để icinga2 kết nối đến mariadb, phần password các bạn set giống với password đã đặt trong phần tạo database cho icinga2: `vi /etc/icinga2/features-available/ido-mysql.conf`
```sh
library "db_ido_mysql"

object IdoMysqlConnection "ido-mysql" {
  user = "icinga"
  password = "pass4icinga"
  host = "localhost"
  database = "icinga"
}
```
- Check các feature mà icinga2 có: `icinga2 feature list`
- enable các feature cần thiết:
```sh
icinga2 feature enable ido-mysql
icinga2 feature enable command
```

<a name="3"></a>
### 3. Install icingaweb2
- Tiếp đến cài đặt icingaweb2
```sh
yum -y install icingaweb2 icingacli`
chcon -R -t httpd_sys_rw_content_t /etc/icingaweb2/
systemctl restart httpd
```
- Sau đó bạn truy cập vào đường dẫn *http://ktlam.com/icingaweb2/setup* để tiến hành setup token và cấu hình icingaweb2.
- Setup token:
```sh
groupadd -r icingaweb2;
usermod -a -G icingaweb2 apache;
icingacli setup config directory --group icingaweb2;
icingacli setup token create;
```
- Sau khi có được token bạn nhập vào ô và ấn next để tiếp tục
<img src="http://image.prntscr.com/image/fc4594b6bc144ed995449bf8a6d2bcbe.png" />

- Trang tiếp theo giúp bạn chọn các module có sẵn của icingaweb2 : Doc, Monitoring, Translation. Mặc định module *Monitoring* sẽ được chọn.Sau đó bạn ấn next để tiếp tục.
- Trang sau sẽ show ra các thành phần cần có của icingaweb2, nếu thiếu cái nào thì bạn install nó thêm vào.
- Trang tiếp theo giúp bạn chọn cách xác thực khi truy cập icingaweb2.Chúng ta để mặc định là *Database* và next.
- Tiếp đến là cấu hình database resource, nơi lưu trữ users và user groups.Sau khi cài đặt các tham số bạn chọn *validate configuration*, nếu thành công bạn nhấn next.
<img src="http://image.prntscr.com/image/7d700777353b40d28cbdb3fca13b2c61.png" />

- Trang kế tiếp để ta cung cấp user MySQL dùng để xác thực như user root và password ta đã set khi cài đặt MySQL.	
<img src="http://image.prntscr.com/image/4afe4e71eee04445803d8dcfe9b8c16b.png" />

- Tiếp theo là phần xác thực backend, bạn để mặc định và next.
- Trang tiếp theo là để bạn config username, password login vào icingaweb2.
<img src="http://image.prntscr.com/image/635673421f2948df97b828d781000178.png" />

- Các trang tiếp theo bạn để mặc định và next.
- Đến trang "Monitoring IDO Resource". Bạn cài đặt như sau: set password giống như bạn set trong file /etc/icinga2/features-available/ido-mysql.conf.
<img src="http://image.prntscr.com/image/15100492f21d46ea9aa61fcd6591561f.png" />

- Cacs trang tiếp bạn để mặc định và finish, sau đó login vào icingaweb2 với username và password bạn đã set.
- Giao diện trang chủ sau khi đã login vào,ở đây báo lỗi thiếu plugin của nagios nên ko thể check các host và service được.
<img src="http://image.prntscr.com/image/0e36c9f240894a3780fa8ecbfc485f67.png" />

- Tiến hành cài đặt plugin: `yum -y install nagios-plugins-all`
- Sau khi cài đặt bạn restart lại icinga2 và vào lại trang chủ của icingaweb2.
<img src="http://image.prntscr.com/image/509e24986f004c77b9f31bedeedc5466.png" />

- Vậy là chúng ta đã cài đặt thành công !!!

<a name="4"></a>
### 4.Tham Khảo
- https://www.youtube.com/watch?v=Nz0OxWzJWiY
