Icinga2 notify sử dụng 2 template cảnh báo trong /etc/icinga2/scripts.
Điều kiện : 
	- Mở tính năng notify trên Icinga2, dùng mailutils và sSMTP.
Step 1 : Mở tính năng notify trên Icinga2

icinga2 feature list 				 (list các feature đã enable)
icinga2 feature enable notification  (mở tính năng notify)

Step2 : Cài đặt mailutils và sSMTP
```sh
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install mailutils ssmtp -y
```
Cấu hình sSMTP /etc/ssmtp/ssmtp.conf 

```sh
#
# Config file for sSMTP sendmail
#
# The person who gets all mail for userids < 1000
# Make this empty to disable rewriting.
root=postmaster
 
# The place where the mail goes. The actual machine name is required no
# MX records are consulted. Commonly mailhosts are named mail.domain.com
AuthUser=@gmail.com
AuthPass=Your-Gmail-Password
mailhub=smtp.gmail.com:587
UseSTARTTLS=YES
 
# Where will the mail seem to come from?
rewriteDomain=Icinga2.Host
 
# The full hostname
hostname=Icinga2
 
# Are users allowed to set their own From: address?
# YES - Allow the user to specify their own From: address
# NO - Use the system generated From: address
FromLineOverride=YES
```
Gửi test mail :  echo "Hello receiver" | mail -s "Test" someone@2342424422.com

