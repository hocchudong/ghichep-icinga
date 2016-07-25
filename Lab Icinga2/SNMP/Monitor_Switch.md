# Cài đặt snmp-plugins và cấu hình monitor SNMP.
## Mục lục
[1.Giới thiệu và chú thích] (#1)

[2.Cài đặt snmp-plugins] (#2)

[3.Cấu hình monitor SNMP] (#3)

<a name="1"></a>
### 1.Giới thiệu và chú thích
- Icinga2 Core
<ul>
	<li>OS: CentOS7</li>
	<li>Ip: 192.168.100.10</li>
</ul>
- Switch cần monitor
<ul>
	<li>Model: Cisco 3550</li>
	<li>Ip: 192.168.100.220</li>
	<li>Interface: FastEthernet0/1-24</li>
	<li>Comunity: public</li>
</ul>

<a name="2"></a>
### 2.Cài đặt snmp-plugins
- Cài đặt các gói cần thiết:
```sh
yum -y install net-snmp net-snmp-utils net-snmp-perl perl-CPAN
```
- Chạy perl để install Net::SNMP:
```sh
perl -MCPAN -e shell // enter mặc định
cpan> install Net::SNMP
```
- Tải nagios-snmp-plugins về:
```sh
wget http://nagios.manubulon.com/nagios-snmp-plugins.1.1.1.tgz
```
- Giải nén:
```sh
tar -xzvf nagios-snmp-plugins.1.1.1.tgz
```
- Vào thư mục đã giải nén ra chạy file : ./install.sh.
- Khi được hỏi install các file plugin ".pl" để chạy snmp, bạn có thể tùy chọn đường dẫn.Tôi sẽ để chung với các plugins trước đấy: /usr/lib64/nagios/plugins

<a name="3"></a>
### 3.Cấu hình monitor SNMP
- Trước khi cấu hình monitor SNMP, set các view table cho các đối tượng cần monitor ở trong file /etc/snmp/snmpd.conf.
- Link về các view table: http://nagios.manubulon.com/index_info.html
- Bắt đầu thêm host,service và command để monitor switch, có thể khai báo các hàm vào các file host.conf, service.conf, command.conf có sẵn ở đường dẫn /etc/icinga2/conf.d. Nhưng để dễ quản lí và fix lỗi, tôi sẽ tạo riêng các file config host.conf, service.conf, command.conf tại đường dẫn :/etc/icinga2/repository.d/hosts/Switch.
#### 3.1.Khai báo host
- Khai báo host trong file host.conf
```sh
object host Switch {
	import "generic-host"
	address = "192.168.100.220"
	// Các tham số sau phục vụ cho việc khai báo các interface được monitor.Bạn có thể thêm các interface khác tùy ý với cú pháp tương tự.
	vars.int["f0/1"] = {
	int = "FastEthernet0/1"
	}
	vars.int["f0/2"] = {
	int = "FastEthernet0/2"
	// Chỉ ra os để tôi add vào GroupHost
	vars.os = "Switch"
	// Chỉ ra address cho 1 số service cần gọi đến.
	vars.address = "192.168.100.220"
}
```

#### 3.2.Khai báo service
##### a.Check-Load
- Bạn có thể vào đường dẫn /usr/lib64/nagios/plugins và chạy file *./check_snmp_load.pl -h* để xem các tham số cần có.
- Mẫu vd: http://nagios.manubulon.com/snmp_load.html
- Trước khi chạy plugins với icinga2, bạn có thể chạy plugins riêng với câu lệnh:
```sh
./check_snmp_load.pl -H 192.168.100.220 -C public -2 -T cisco -w 3,2,1 -c 3,3,2 -f
```

- Khai báo service vào trong file service.conf
```sh
object Service "snmp-load" {
    host_name           = "Switch"
    // Set the type of load check to use.
    vars.snmp_load_type = "cisco"
    // Set the Load Average warning threshold.
    vars.snmp_warn      = "3,2,1" // 1min: 2*(Number of Cores)+1, 5min: (Number of Cores)+1, 15min: (Number of Cores)
    // Set the Load Average critical threshold.
    vars.snmp_crit      = "3,3,2" // 1min: 3*(Number of Cores), 5min: 2*(Number of Cores)+1, 15min: (Number of Cores)+1
    check_command       = "snmp-load"
	vars.community = "public"
}
```

##### b.Check-Memory
- Tương tự chạy file *./check_snmp_mem.pl -h* để xem các tham số cần có.
- Mẫu vd: http://nagios.manubulon.com/snmp_mem.html
- Tương tự check plugins riêng:
```sh
./check_snmp_mem.pl -H 192.168.100.220 -C public -2 -w 99 -c 100 -I -f
```

- Khai báo tiếp service vào trong file service.conf
```sh
apply Service "switch-memory" {
	import "generic-service"
	// Set the Memory warning for Ram and swap Respectively.
    // Uses percents.
    check_command  = "check_memory"
	// Gọi đến address của switch.
	assign where host.vars.address
}
```

##### c.Check-Interface
- Tương tự chạy file *./check_snmp_int.pl -h* để xem các tham số cần có.
- Mẫu vd: http://nagios.manubulon.com/snmp_int.html
- Check plugins riêng:
```sh
./check_snmp_int.pl -H 192.168.100.220 -C public -n "Fast.*0.1[1234]" 
```

- Khai báo tiếp service vào trong file service.conf
```sh
apply Service "Switch-interface: " for (int => config in host.vars.int){
	import "generic-service"
	check_command = "check_interface"
	vars += config
	assign where host.vars.int
}
```

#### 3.3.Khai báo command
##### a.Check-Memory
- Khai báo command trong file command.conf
```sh
object CheckCommand "check_memory" {
	import "plugin-check-command"
	command = [ PluginDir + "/check_snmp_mem.pl", "-2", "-f", "-I"]
	arguments = {
		"-H" = "$address$"
		"-C" = "$community$"
		"-w" = "99"
		"-c" = "100"
	}
	vars.community = "public"
}
```

##### b.check_interface
- Khai báo tiếp command trong file command.conf
```sh
object CheckCommand "check_interface" {
	import "plugin-check-command"
	command = [ PluginDir + "/check_snmp_int.pl", "-2", "-r", "-f", "-k", "-Y"] 

	arguments = {
	"-H" = "$address$"
	"-C" = "$community$"
	"-n" = "$int$"
	"-w" = "90,90"
	"-c" = "99,99"
	}
	vars.community = "public"
}
```

- Cuối cùng reload lại icinga2 và enjoy.
