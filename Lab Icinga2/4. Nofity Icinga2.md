###I. Notify trong Icinga2

**Icinga2 Notify** sử dụng 2 template cảnh báo trong /etc/icinga2/scripts.

####1. Điều kiện : 
 - Tính năng notify trên Icinga2 được bật
 - Có mail server để gửi mail cảnh báo
 
####2. Các bước cấu hình gửi mail cảnh báo

<li>Step 1 : Mở tính năng notify trên Icinga2</li>

	- icinga2 feature list 				 (list các feature đã enable)
	- icinga2 feature enable notification  (mở tính năng notify)
![icinga](/images/icinga-05.png)
	
<li>Step2 : Cài đặt mailutils và sSMTP</li>

```sh
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install mailutils ssmtp -y
```

<li>Step2: Cấu hình sSMTP /etc/ssmtp/ssmtp.conf</li>
![icinga](/images/icinga-09.png)

Gửi test mail :  
```sh
echo "Hello receiver" | mail -s "Test" manhdinh1994@gmail.com
```
Mở Inbox Email và kiểm cha mail Test

<li>Step 3 : Tạo user gửi mail và thêm user gửi mail vào object Host</li>
![icinga](/images/icinga-06.png)
![icinga](/images/icinga-07.png)

<li>Step4 : Thực hiện gửi mail khi Host có vấn đề</li>

Ta tắt host đang giám sát, host sẽ chuyển sang trạng thái **DOWN**, mail cảnh báo sẽ được gửi đi
![icinga](/images/icinga-08.png)
![icinga](/images/icinga-10.png)
