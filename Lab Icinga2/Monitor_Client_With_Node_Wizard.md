# Các Mode cấu hình Client với Node Wizard
## Mục lục
[1.Giới thiệu và chú thích] (#1)

[2.Mode 1: Cấu hình trên local clients] (#2)

[3.Mode 2: Cấu hình trên Master áp dụng cho clients sử dụng brigde] (#3)

<a name="1"></a>
### 1.Giới thiệu và chú thích
- Như ta đã biết ở bài viết [Thêm host bằng Wizard] (https://github.com/hocchudong/ghichep-icinga/blob/master/Lab%20Icinga2/2.%20Icinga2%20Client.md) thì để monitor 1 host nào đấy chúng ta sẽ dùng wizard.
- Nhưng khi add node wizard, bạn mới chỉ monitor được các service cơ bản như: CPU, disk, ping, ssh ...
- Bài viết sau đây tôi sẽ tùy chỉnh để có thể monitor được thêm các service khác như: Memory, Interface ...
- Từ đó bạn có thể tùy ý thêm các service khác mà bạn muốn monitor trên clients.
- Có 3 mode cấu hình client chính khi dùng wizard:
<ul>
	<li>Cấu hình trên local client, sau đó gửi đến master.</li>
	<li>Client được coi như brigde để thực thi các command mà không có bất kì cấu hình local nào.</li>
	<li>Client nhận hoàn toàn cấu hình từ master - đồng bộ cấu hình giữa các cluster. (đây là phần nâng cao nên sẽ đề cập trong bài viết sau)</li>
</ul>
- Mô hình lab của tôi như sau:
- Icinga2 master:
<ul>
	<li>OS: CentOS 7</li>
	<li>IP address: 192.168.10.10</li>
	<li>hostname: icinga2.lamtlu.com</li>
</ul>
- Icinga2 Linux client
<ul>
	<li>OS: CentOS 6.7</li>
	<li>IP address: 192.168.10.11</li>
	<li>hostname: dns1.lamtlu.com</li>
</ul>
- Icinga2 Windows client
<ul>
	<li>OS: Windows 7</li>
	<li>IP address: 192.168.10.100</li>
	<li>hostname: KTLam-PC</li>
</ul>

<ul>
	<li></li>
	<li></li>
</ul>
<a name="2"></a>
### 2. Mode 1: Clients with Local Configuration.
- Không có sự khác nhau trong cú pháp cấu hình trên clients với các cấu hình icinga2 master.
- Các quy ước như sau:
<ul>
	<li>Hostname trong cấu hình object host mặc định nên để giống với Common Name khi setup SSL</li>
	<li>Thêm service mới và các check-command tại local client.</li>
</ul>
- Các cấu hình sẽ được chuyển tới master
#### 2.1. Window client
- Trước tiên tôi install icinga trên disk D:
- Khi install icinga trên window, sẽ có các file thực thi có sẵn lưu tại đường dẫn *D:\ICINGA2\sbin* để check các dịch vụ cơ bản trên window như: check_disk, check_load, check_memory, check_network ...
- Sau đó ta vào file service theo đường dẫn *D:\ICINGA2\etc\icinga2\conf.d\service.conf* để add thêm các dịch vụ muốn monitor.Ta xem qua các service có sẵn trong file.
```sh
apply Service "ping4" {
  import "generic-service"

  check_command = "ping4-windows"

  assign where host.address
}

apply Service "ping6" {
  import "generic-service"

  check_command = "ping6-windows"

  assign where host.address6
}

apply Service for (disk => config in host.vars.disks) {
  import "generic-service"

  check_command = "disk-windows"

  vars += config
}

apply Service "icinga" {
  import "generic-service"

  check_command = "icinga"

  assign where host.name == NodeName
}

apply Service "load" {
  import "generic-service"

  check_command = "load-windows"

  /* Used by the ScheduledDowntime apply rule in `downtimes.conf`. */
  vars.backup_downtime = "02:00-03:00"

  assign where host.name == NodeName
}

apply Service "procs" {
  import "generic-service"

  check_command = "procs-windows"

  assign where host.name == NodeName
}

apply Service "swap" {
  import "generic-service"

  check_command = "swap-windows"

  assign where host.name == NodeName
}

apply Service "users" {
  import "generic-service"

  check_command = "users-windows"

  assign where host.name == NodeName
}
```

- Ở đây ta có thể thấy chưa có service cơ bản như memory.
- Trước khi add service, ta cùng xem qua command check_memory có sẵn ở file *D:\ICINGA2\share\icinga2\include\command-plugins-windows*
```sh
object CheckCommand "memory-windows" {
	import "plugin-check-command"
	
	command = [ PluginDir + "/check_memory.exe" ]
	
	arguments = {
		"-w" = {
			value = "$memory_win_warn$"
			description = "Warning Threshold"
		}
		"-c" = {
			value = "$memory_win_crit$"
			description = "Critical Threshold"
		}
		"-u" = {
			value = "$memory_win_unit$"
			description = "Use this unit to display memory"
		}
	}
	vars.memory_win_unit = "mb" 
	//The default
}
```

- Giải thích các tham số: 
<ul>
	<li>-w: ngưỡng cảnh báo khi dung lượng free của memory đạt đến value được chỉ định, nếu bạn muốn tính theo %, thì thêm % vào sau value.</li>
	<li>-c: ngưỡng nguy hiểm.</li>
	<li>-u: đơn vị để hiển thị  memory, mặc định là mb.đơn vị # : gb, kb.</li>
</ul>
- Nếu bạn muốn đặt ngưỡng cảnh báo và ngưỡng nguy hiểm, thì add thêm các tham số sau vào CheckCommand "memory-windows" trên
```sh
vars.memory_win_warn = "10%"
vars.memory_win_crit = "1%"
```

- Lưu ý: giá trị của crit > warn.
- Tiếp theo add doan ham sau vao file D:\ICINGA2\etc\conf.d\service.conf để monitor memory.
```sh
	apply Service "memory" {
		import "generic-service"
		check_command = "memory-windows"
		assign where host.name == NodeName
		}
```
	
- Sau đó chạy [node wizard] (https://github.com/hocchudong/ghichep-icinga/blob/master/Lab%20Icinga2/2.%20Icinga2%20Client.md)
- Kiểm tra các node đã nhận hay chưa: `icinga2 node list`
<img src="http://image.prntscr.com/image/036da201cbde405c95406e7cb3e932f0.png" />

- Sau đó update các node và restart icinga2.
```sh
icinga2 node update-config
systemctl restart icinga2
```

- Vào trên giao diện web

<img src="http://image.prntscr.com/image/8a21ed7962c34bbfb004837752a489a3.png" />

#### 2.2. Linux client
- Install icinga
- Tương tự như đối với window, nhưng trên linux khi install icinga2 cũng như nagios-plugins-alls, không có sẵn file plugin để check memory.Do đó bạn phải tải về và để chung đường dẫn với các plugin trước đấy tại */usr/lib64/nagios/plugins*
- Tôi đã tải file plugin "check_mem.pl" tại đường dẫn  *https://exchange.nagios.org/directory/Plugins/Operating-Systems/Solaris/check_mem-2Epl/details*
- Trước khi add service memory, ta xem qua file *CheckCommand "mem"* tại đường dẫn */usr/share/icinga2/include*/plugins-contrib.d/operating-system.conf* mà service sẽ gọi đến:
```sh
object CheckCommand "mem" {
	import "plugin-check-command"

	command = [ PluginContribDir + "/check_mem.pl" ]

	arguments = {
		"-u" = {
			set_if = "$mem_used$"
			description = "Check USED memory"
		}
		"-f" = {
			set_if = "$mem_free$"
			description = "Check FREE memory"
		}
		"-C" = {
			set_if = "$mem_cache$"
			description = "Count OS caches as FREE memory"
		}
		"-w" = {
			value = "$mem_warning$"
			description = "Percent free/used when to warn"
		}
		"-c" = {
			value = "$mem_critical$"
			description = "Percent free/used when critical"
		}
	}

	vars.mem_used = false
	vars.mem_free = false
	vars.mem_cache = false
}
```

- Sau đó chúng ta set *const PluginContribDir = "/usr/lib64/nagios/plugins"* ở trong file */etc/icinga2/constants.conf* để nó nhận plugin mà ta đã thêm vào trước dấy.
- Nếu bạn muốn đặt ngưỡng cảnh báo và ngưỡng nguy hiểm, thì add thêm các tham số sau vào CheckCommand "mem" trên
```sh
vars.mem_warning = 70
vars.mem_critical = 90
```

- Tiếp theo add doan ham sau vao file /etc/conf.d/service.conf
	apply Service "memory" {
		import "generic-service"
		check_command = "mem"
		assign where host.name == NodeName
	}
	
- Tiep tuc vao file /etc/icinga2/icinga2.conf goi plugins-contrib: 
```sh
include <plugins-contrib>
```
- Sau đó chạy [node wizard] (https://github.com/hocchudong/ghichep-icinga/blob/master/Lab%20Icinga2/2.%20Icinga2%20Client.md)

- Kiểm tra các node đã nhận hay chưa: `icinga2 node list`

- Sau đó update các node và restart icinga2.
```sh
icinga2 node update-config
systemctl restart icinga2
```

- Vào trên giao diện web
<img src="http://prnt.sc/bxneid" />

<a name="3"></a>
### 3.Mode 2: Cấu hình trên Master áp dụng cho clients sử dụng brigde
- Ở mode này, client chỉ thực thi cac command được gửi từ master. Không có object host hay service nào được cấu hình local, chỉ các check command được cấu hình.

#### 3.1 Windows Client
##### Trên Client
- Install icinga disk D:
- Chạy luôn [node wizard] (https://github.com/hocchudong/ghichep-icinga/blob/master/Lab%20Icinga2/2.%20Icinga2%20Client.md)

##### Trên Master
- Thêm các tham số sau vào file host */etc/icinga2/repository.d/KTLam-PC.conf sau khi node wizard.
```sh
	object Host "KTLam-PC" {
		import "satellite-host"
		check_command = "cluster-zone"
		vars.os = "Windows Server"
		vars.clients = "KTLam-PC"	// chỉ ra tham số clients để các service gọi đến.
	}
```

- Tiếp tục vào đường dẫn:/etc/icinga2/repository.d/hosts/KTLam-PC (nơi chứa các file service được sinh ra sau khi node wizard, các service này được khai báo giống như cách 1.).T tạo 1 file service mới "memory.conf" 
```sh
	apply Service "memory" {
	import "generic-service"
	check_command = "memory-windows" // ten command: /usr/share/icinga2/include/command-plugins-windows.conf
	command_endpoint = host.vars.clients // Chỉ ra các command này chạy đối với endpoint được add thêm vào, và tham số chi định là "vars.clients"
	assign where host.name == "KTLam-PC"
	}
```

- Tùy chỉnh các tham số trong CheckCommand "memory-windows"
```sh
object CheckCommand "memory-windows" {
	import "plugin-check-command"
	
	command = [ PluginDir + "/check_memory.exe" ]
	
	arguments = {
		"-w" = {
			value = "$memory_win_warn$"
			description = "Warning Threshold"
		}
		"-c" = {
			value = "$memory_win_crit$"
			description = "Critical Threshold"
		}
		"-u" = {
			value = "$memory_win_unit$"
			description = "Use this unit to display memory"
		}
	}
	vars.memory_win_unit = "mb" 
	//The default
	vars.memory_win_warn = "10%"
	vars.memory_win_crit = "1%"
}

- Kiểm tra trên file /etc/icinga2/icinga2.conf đã gọi windows-plugins chưa:`include <windows-plugins>`

- Restart lại dịch vụ.

#### 3.2.Linux client
##### Trên Client
- Tương tự như trên window, Install Icinga2 và sau đó chạy [node wizard] (https://github.com/hocchudong/ghichep-icinga/blob/master/Lab%20Icinga2/2.%20Icinga2%20Client.md)

##### Trên Master
- Thêm các tham số sau vào file host */etc/icinga2/repository.d/dns1.lamtlu.com.conf*
```sh
	object Host "dns1.lamtlu.com" {
		import "satellite-host"
		check_command = "cluster-zone"
		vars.os = "Linux Servers"
		vars.clients = "dns1.lamtlu.com"
	}
```

- Tiếp tục vào đường dẫn:/etc/icinga2/repository.d/hosts/dns1.lamtlu.com (nơi chứa các file service được sinh ra sau khi node wizard, các service này được khai báo giống như cách 1.).T tạo 1 file service mới "memory.conf"
```sh
	apply Service "memory" {
		import "generic-service"
		check_command = "mem"	
		command_endpoint = host.vars.clients
		assign where host.name == "dns1.lamtlu.com"
	}
```

- Bạn cũng phải nhớ để file plugin *check_mem.pl* ở cùng đường dẫn các plugins # tại */usr/lib64/nagios/plugins*
- Sau đó vào file /usr/share/icinga2/include/plugins-contrib.d/operating-system.conf để chỉnh lại các tham số command "mem"
```sh
	object CheckCommand "mem" {
		import "plugin-check-command"
	
		command = [ PluginContribDir + "/check_mem.pl" ]

		arguments = {
			"-u" = {
				set_if = "$mem_used$"
				description = "Check USED memory"
			}
			"-f" = {
				set_if = "$mem_free$"
				description = "Check FREE memory"
			}
			"-C" = {
				set_if = "$mem_cache$"
				description = "Count OS caches as FREE memory"
			}
			"-w" = {
				value = "$mem_warning$"
				description = "Percent free/used when to warn"
			}
			"-c" = {
				value = "$mem_critical$"
				description = "Percent free/used when critical"
			}
		}

		vars.mem_used = false
		vars.mem_free = false
		vars.mem_cache = false
		vars.mem_warning = 70
		vars.mem_critical = 90
	}
```

- Tiep tuc vao file /etc/icinga2/icinga2.conf goi plugins-contrib: 
	include <plugins-contrib>
- Restart lại dịch vụ.
