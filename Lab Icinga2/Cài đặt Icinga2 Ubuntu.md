###I. Cai đặt Icinga2 trên Ubuntu
OS : Ubuntu 14.04 x 64
Model : ALL-in-one
Thành phần : Icinga2 Server, IcingaWeb2, IDO database sủ dụng MySQL.

###II. Cài đặt

####Step 1 : Cài đặt Icinga2 Server

```sh
add-apt-repository ppa:formorer/icinga
apt-get update
apt-get install icinga2 -y
```
![icinga](/images/i11.png)

####Step 2 : Cài đặt và cấu hình Database IDO MYSQL

```sh
apt-get install mysql-server mysql-client -y
apt-get install icinga2-ido-mysql -y
```
Chú ý khai báo password cho mysql và IDO database, nếu chọn config kiểu cấu hình tự động, IDO database tự động config cấu hình với Mysql.

![icinga](/images/i30.png)
![icinga](/images/i31.png)

Bật feature ido-mysql cho Icinga2

```sh
icinga2 feature enable ido-mysql command 
service icinga2 restart
```

####Step 3 : Cài đặt Icinagweb2

Add user của webserver vào group icingamd để có quyền sử dụng command cho icinga2

```sh
usermod -a -G nagios www-data
```

Cài đặt Icingaweb2

```sh
wget -O - http://packages.icinga.org/icinga.key | apt-key add -
add-apt-repository 'deb http://packages.icinga.org/ubuntu icinga-trusty main'
apt-get update
apt-get install icingaweb2 -y
```
- Tạo token xác thực : 
```sh
icingacli setup token create
```

Câu lệnh để xem token :

```sh
icingacli setup token show
```

Cài đặt các gói php :

```sh
apt-get -y install php5 php5-cgi libapache2-mod-php5 php5-common php-pear php5-json php5-gd php5-imagick php5-pgsql php5-intl php5-ldap php5-mysql 
```

Tạo file config cho web server

```sh
icingacli setup config webserver apache --document-root /usr/share/icingaweb2/public
```

Thay đổi date.timezone php trong thư mục /etc/php5/apache2/php.ini 
![icinga](/images/i29.png)

Phân quyền user và group
```sh
addgroup --system icingaweb2
usermod -a -G icingaweb2 www-data
service apache2 restart
icingacli setup config directory
```

Tạo thư mục monitor :

```sh
mkdir /etc/icingaweb2/modules/monitoring
chmod 777 /etc/icingaweb2/modules/monitoring
```

Truy cập vào broswer với địa chỉ : http://ip/icingaweb2/setup

 - Nhập Token
 
![icinga](/images/i12.png)

 - Chọn các Module để cài đặt
 
![icinga](/images/i13.png)

 - Cài đặt các gói php cần thiết cho Icinga2

![icinga](/images/i14.png)

 - Chọn kiểu xác thực cho Icinga2

![icinga](/images/i15.png)

 - Tạo database cho IcingWeb2

![icinga](/images/i16.png)

![icinga](/images/i17.png)

![icinga](/images/i18.png)

 - Tạo user và password đăng nhập IcingaWeb2

![icinga](/images/i19.png)

 - Cấu hình logging khi cài đặt

![icinga](/images/i20.png)

 - Review các cấu hình cơ bản

![icinga](/images/i21.png)

 - Thực hiện cấu hình cho module Monitor

![icinga](/images/i22.png)

 - Chọn Monitoring backend 

![icinga](/images/i23.png)

 - Lấy thông tin về database của ido-mysql

![icinga](/images/i24.png)

 - Nhập Lấy thông tin database từ file ido-mysql.conf

![icinga](/images/i25.png)

![icinga](/images/i26.png)

![icinga](/images/i27.png)

 - Review lại các thông tin về module Monitor

![icinga](/images/i28.png)

