# 																					DDOS


	##开启http服务##
	centos7 				#安装 httpd 服务进行测试。
	yum install -y httpd
	systemctl start httpd
	http://192.168.1.63/	#浏览器访问 


	##对服务器的压力测试##
	hping3 -c 1000 -d 120 -S -w 64 -p 80 --flood --rand-source 192.168.1.63
			-c 1000 		# 发送的数据包的数量。
			-d 120 			# 发送到目标机器的每个数据包的大小。单位是字节
			-S 				#只发送 SYN 数据包。
			-w 64 			#TCP 窗口大小。
			-p 80 			#目的地端口(80 是 WEB 端口)。你在这里可以使用任何端口。
			--flood 		#尽可能快地发送数据包，不需要考虑显示入站回复。洪水攻击模式。
			--rand-source 	#使用随机性的源头IP地址。这里的伪造的IP地址，只是在局域中伪造。通过路由器后，还会还原成真实的IP 地址


	1）通过优化缓解 DDOS 攻击，优化 Linux 内核参数提升 SYN：上传 Centos7 内核优化脚本，脚本优化了 Centos 内核参数
		net.ipv4.tcp_syncookies = 1 			
				#打开 SYN cookies 功能，该功能可以尽量把过多的 SYN 请求缓存起来。
		net.ipv4.tcp_max_syn_backlog = 262144	
				#定义 backlog 队列容能容纳的最大半连接数，Centos 系统中该值默认为 256，是远远不够的。
		net.ipv4.tcp_synack_retries = 1		
				#表示收到SYN后发送SYN+ACK的重传次数，默认为 5 表示重试 5 次，而且 每次重试的间隔会翻倍。
				#1、2、4、8、16 秒，最后一次重试后等待32秒，若仍然没有收到ACK，才会关闭连接故共需要等待63秒。
				#日常中这个值推荐设置为 2 秒，目前网络环境延迟没有以前那么高所以 2 秒可以满足日常使用，攻击比较多的时						#候可适当为 1 秒。
		net.core.somaxconn = 65535		
				#上面所提到的 backlog 队列实际上受限于系统级的队列上线，通过修改 		  			
		net.core.somaxconn 
				#来提高系统队列上限。
	/*以上是对SYN的一些优化参数，实际上脚本中有很多针对系统以及TCP协议的优化，目的都是提高服务器的响应能力。修改HTTPD服务的队列长 	度，每个应用会有自己默认的队列长度，应用的队列长度和系统队列长度取最小值。所以修改服务配置。默认HTTPD服务队列长度是511.
			ss -lnt
		echo "ListenBacklog 2048" >> /etc/httpd/conf.modules.d/00-mpm.conf
			   	#添加 HTTPD 服务队列长度配置，不要给的太高。系统最大值 65535，由 somaxconn 参数指定。
		systemctl restart httpd		#重启 HTTPD 服务	
			ss -lnt

```
netstat -an | grep SYN | wc -l		#查看 SYN 半连接数量
```

	2）通过 iptables 封禁攻击者 ip 缓解 DDOS 攻击。
		iptables 是 centos 自带的防火墙工具，可以通过添加防火墙规则限制同一 ip 地址对我们发起过多的请求。
		iptables 四个表(raw,mangle,nat,filter);5 个链接(INPUT,OUTPUT,FORWARD,PREROUTING,POSTROUTING)
		iptables命令总结：iptables [-t 要操作的表] <操作命令> [要操作的链] [规则号码] [匹配条件] [-j 匹配到以后的动作]
	
		操作命令
			-A 添加规则
			-I num 插入，把当前规则插入为第几条
			-D num 删除，明确指定删除第几条规则
			-P 设置默认策略的
			-F 清空规则链的
		查看命令
			-[vnx]L
			-L 列出规则
			-n 以数字格式显示 ip 和 port，需要配合-L 选项使用
			-v 显示信息，以详细信息显示
	
		实例：
			允许访问 TCP/80 端口
				 iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
					-m： module_name
					-p： protocol
					iptables -p tcp : 表示使用 TCP 协议
					iptables -m tcp：表示使用TCP模块的扩展功能（tcp 扩展模块提供了 --dport, --tcp-flags, --sync 等功能）
					--dport 80 目标端口 80 
					-j ACCEPT 放行流量
			拒绝大于 15 个连接的 IP 访问。
				iptables -I INPUT -p tcp --dport 80 -m connlimit --connlimitabove 15 -j REJECT
					-m connlimit --connlimit-above 15
	
				connlimit 功能：	限制每个客户端 IP 的并发连接数，即每个 IP 同时连接到一个服务器个数。
						限制内网用户的网络使用，对服务器而言则可以限制每个 IP 发起的连接数。
				connlimit 参数：
					--connlimit-above n ＃限制为多少个
					--connlimit-mask n ＃这组主机的掩码,默认是 connlimit-mask 32 ,即每个 IP.
					REJECT 动作会返回一个拒绝(终止)数据包(TCP FIN或UDP-ICMP-PORT-UNREACHABLE)，明确的拒绝对方的连接动作。


​	