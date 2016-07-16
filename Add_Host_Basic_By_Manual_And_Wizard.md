# icinga2 cơ bản (Cấu hình trên CentOS 7)

### [1. Thêm host bằng Wizard] (#wizard)

- #### [Host Windows] (#w.win) 

- #### [Host Linux] (#w.lin)

### [2. Thêm host bằng Manual] (#manual)

<a name="wizard"></a>
### Thêm host muốn giám sát bằng Wizard
<a name="create-ticket"><a/>
#### 1. Cấu hình trên icinga2

Sử dụng Wizard bằng câu lệnh:

```
icinga2 node wizard
```
<img src="http://image.prntscr.com/image/d40e8e726db9435e85c6a9ffed4d9ac3.png">

Khởi động lại icinga2

```
systemctl restart icinga2
```
<a name="create-ticket"><a/>
Tạo Ticket cho host muốn giám sát:

```
icinga2 pki ticket --cn "Windows-7-Demo"
```

<img src="http://image.prntscr.com/image/1f5d3b2b69314194b682c6705a247908.png">


**Chú ý:** "Windows-7": là tên của host muốn giám sát, có thể đặt tự do nhưng khi cấu hình agent trên host đó phải điền đúng tên và ticket
<a name="w.win"></a>
#### 2.1 Cấu hình trên host (Windows 7)

##### 2.1.1 Tải Agent cho Windows

https://packages.icinga.org/windows/

##### 2.1.2 Cài đặt Agent

Bấm **Next** để chuyển sang bước tiếp theo

<img src="http://image.prntscr.com/image/d1b7bd5cd5a84683bf1c1fc09631cf08.png">

Bấm **I agree** để Đồng ý với các Lisence của icinga2

<img src="http://image.prntscr.com/image/510e0f572d25414cb00c3eceeb4ccfa0.png">

Bấm **Next** để tiếp tục

<img src="http://image.prntscr.com/image/510e0f572d25414cb00c3eceeb4ccfa0.png">

Bấm **Install** để cài đặt

<img src="http://image.prntscr.com/image/54917a17aa534da3a4df14e2eddfc3d7.png">

Chờ khoảng 1 phút để quá trình cài đặt diễn ra.

Bấm **Finish** để khởi động Agent

img src="http://image.prntscr.com/image/7a528858d5334d418df59e49c2221ec5.png">

Bấm vào **Reconfigure** để cấu hình

<img src="http://image.prntscr.com/image/24305faaada647468d6e58e7a8610f35.png">

Sau đó, icinga2 agent sẽ hiện ra 1 bảng để chúng ta điền các thông tin cấu hình:

<img src="http://image.prntscr.com/image/a45ccf561f3c49eb8ae4c5e5c9f606e5.png">

Làm tiếp tục như hình

<img src="http://image.prntscr.com/image/cd3e800aaa004d82be451d3130e50e77.png">

Bấm **Next** để tiếp tục

<img src="http://image.prntscr.com/image/edf9844830e04758bf872a4e6d565093.png">

Chờ một chút cho agent hoạt động và khởi chạy Service trên host. Bấm **Finish** để hoàn tất quá trình.

<img src="http://image.prntscr.com/image/e8b8215c79bb4e4a946caf95926c4d31.png">

#### 2. Giám sát host trên icinga2

Khởi động lại dịch vụ và update các node

```
systemctl restart icinga2
icinga2 node update-config
```
<img src="http://image.prntscr.com/image/ae04725c908644e09c99ccb0516c866c.png">

Liệt kê các node đang giám sát

```
icinga2 node list
```

<img src="http://image.prntscr.com/image/c787f0da70324d45a14112bc6356c8a3.png">

#### 2.1.3 Giám sát trên Web

Truy cập vào IP hoặc FQDN của icinga

`http://youricinga2.com/icingaweb2`

<img src="http://image.prntscr.com/image/ff41b50d159b4da780ed1166a4cb3ff4.png">

Click vào **Tab Overviews > Hosts**

<img src="http://image.prntscr.com/image/e33913525d3c4340b255399d8ac1c4ff.png">

Chọn host **Windows-7-Demo**

Chúng ta sẽ kiểm tra dịch vụ như DISK, LOAD, PROC,..

<img src="http://image.prntscr.com/image/0f479276da6c4d70ae4cb119cefcff1c.png">

<a name="w.lin"></a>
#### 2.2 Cấu hình trên host (Linux)

##### 2.2.1 Tải Agent cho Linux CentOS 6

Mở port trên host

http://prntscr.com/bt6r48

Thêm repo của icinga2 vào host

```
rpm -i https://packages.icinga.org/epel/6/release/noarch/icinga-rpm-release-6-1.el6.noarch.rpm
```

Cài đặt icinga2 và các plugin

```
yum install -y icinga2
yum install -y nagios-plugins-all
```

Kích hoạt tính năng Wizard của icinga2 và Khai báo các thông tin cấu hình

```
icinga2 node wizard
```

<img src="http://image.prntscr.com/image/ad33363ea26441efa2e644a736147191.png" />

Sau khi xác nhận thông tin, chúng ta chuyển sang Master icinga2 để tạo Ticket cho host

http://prntscr.com/btaeet

Quay trở lại host để paste Ticket, và cho phép nhận các thông tin từ Master (Server Icinga2)

<img src="http://image.prntscr.com/image/d692a934fcc1429380cfe84cc2ba08ab.png" >

Cuối cùng là khởi động lại icinga2 trên host bằng lệnh

```
service icinga2 restart
```

##### 2.2.2 Cấu hình trên Master (Server Icinga2)


Trên Master, chúng ta liệt kê các host bằng lệnh và cập nhật cấu hình

```
icinga2 node list
icinga2 node update-config
```

<img src="http://image.prntscr.com/image/f08408239aa94e15aee38583c8f47056.png" >

Reload lại các cấu hình trên Master

```
systemctl reload icinga2
```

Truy cập vào IcingaWeb2 bằng trình duyệt, chúng ta có thể thấy một host mới ở **Overview > Hosts**

<img src="http://image.prntscr.com/image/476e897cda394422b16901314c7c24bc.png"/>

<a name="manual"></a>
### Thêm host muốn giám sát bằng Manual

**Lưu ý:** Cách này icinga2 sẽ check các dịch vụ tương ứng với các port đang hoạt động trên host, vì thế không thể kiểm tra giám sát được hoàn toàn host như DISK, PROC, LOAD như ở Wizard.

Với cách cấu hình này, chúng ta chỉ cần thực hiện trên Master mà không phải "động chạm" đến host. Với điều kiện, chúng ta phải biết IP của host muốn giám sát.

Tạo thêm một file cấu hình mới cho host muốn giám sát.

```
vi /etc/icinga2/repository.d/WebServer.conf
```

Với nội dung như sau:

```
# Check service "ssh"
object Service "ssh" {
        host_name = "Web-Server-1"
        check_command = "ssh"
}
# Check service "http"
object Service "http" {
        host_name = "Web-Server-1"
        check_command = "http"
}
# Create a new host with name as "Web-Server-1"
object Host "Web-Server-1" {
        import "generic-host"
		address = "192.168.100.160"
         }
}
```

Lưu lại và reload để icinga2 nhận file cấu hình

```
systemctl reload icinga2
```

Vào Web để kiểm tra **Overview > Hosts**

<img src="http://image.prntscr.com/image/c5b1123c43814491bd808522f746c1f8.png" />



