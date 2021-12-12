# 										NMAP

###### eg1：使用 nmap 扫描一台服务器默认情况下，Nmap 会扫描 1000 个最有可能开放的 TCP 端口。

> ```
> nmap 192.168.1.63
> ```

###### eg2： 扫描一台机器，查看它打开的端口及详细信息。

> ```
> 参数说明：
> 		-v 			#表示显示冗余信息，在扫描过程中显示扫描的细节，从而让用户了解当前的扫描状态。
> 		nmap -v 192.168.1.63 	#查看以下相关信息。
> ```

###### eg3：扫描一个范围： 端口 1-65535

> ```
>  nmap -p 1-65535 192.168.1.63 		#生产环境下，我们只需要开启正在提供服务的端口，其他端口都关闭。
> ```

> ​	关闭不需要开的服务有两种方法：
> ​			情景 1：你认识这个服务，直接关服务
>
> ```
> systemctl stop rpcbind
> ```
>
> ​			情景 2：不认识这个服务，查看哪个进程使用了这个端口并找出进程的路径，然后 kill 进程，删除文件，接下来以 22 端口为							例，操作思路如下：
>
> ```
> lsof -i :22 #查看 22 端口正在被哪个进程使用
> COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME sshd 1089 root 3u IPv4 21779 0t0 TCP *:ssh (LISTEN)
> ```
>
> ​							通过 ps 命令查找对应的进程文件：						
>
> ```
> ps -axu | grep 1089
> root 1089 0.0 0.1 105996 3744 ? Ss 10:52 0:00 /usr/sbin/sshd -D 		#看到进程的文件的路是/usr/sbin/sshd 
> ```
>
> 如果没有看到此命令的具体执行路径，说明此木马进程可以在 bash 终端下直接执行，通过 which 和 rpm -qf 来查看命令的来源，	
>
> ```
> which vim
> /usr/bin/vim 
> 
> 解决：kill -9 1089
> ```
>
>
> 总结：这个思路主要用于找出黑客监听的后门端口和木马存放的路径。

###### eg4： 扫描一台机器：查看此服务器开放的端口号和操作系统类型。

> ```
>  nmap -sS -O www.xuegod.cn
> 		参数说明：
> 			-O： 显示出操作系统的类型。 每一种操作系统都有一个指纹。
> 			-sS：半开扫描(half-open)
> ```

###### eg5：扫描一个网段中所有机器是什么类型的操作系统。

> ```
>  nmap -sS -O 192.168.1.0/24 
> ```

###### eg6：查找一些有特点的 IP 地址中，开启 80 端口的服务器。

> ```
>  nmap -v -p 80 192.168.1.62-67
> ```

eg7：如何更隐藏的去扫描，频繁扫描会被屏蔽或者锁定 IP 地址。

> ```
> （1）随机扫描
> 		nmap -v --randomize-hosts -p 80 192.168.1.62-69
> （2）随机扫描+延时扫描 ，默认单位秒
> 		nmap -v --randomize-hosts --scan-delay 3000ms -p 80 192.168.1.62-69
> 		
> 		--randomize_hosts # 随机扫描，对目标主机的顺序随机划分
> 		--scan-delay #延时扫描，单位秒，调整探针之间的延迟
> ```

###### eg8：使用通配符指定 IP 地址

> ```
> nmap -v --randomize-hosts --scan-delay 30 -p 80 1.*.2.3-8
> ```

###### eg9：Connect 扫描

> ```
>  nmap -sT 192.168.1.63 -p 80 		#这种扫描方式和 SYN 扫描很像，只是这种扫描方式完成了 TCP 的三次握手。
> ```

###### eg10：UDP 扫描

> ```
>  nmap -sU 192.168.1.63
> ```

> ```
> 端口状态解析：
> 			open：从目标端口得到任意的 UDP 应答
> 			open|filtered：如果目标主机没有给出应答
> 			closed：ICMP 端口无法抵达错误
> 			filtered：ICMP 无法抵达错误
> ```

###### eg10：报文分段扫描	

> ```
>  nmap -f -v 192.168.1.63
> ```

###### eg11：使用诱饵主机隐蔽扫描

> ```
> （1）随机 3 个诱饵
> 		nmap -D RND:3 192.168.1.63
> （2）使用自己 IP 作为诱饵
> 		nmap -D ME 192.168.1.63
> （3）指定单个 IP：192.168.1.14 作为诱饵
> 		nmap -D 192.168.1.14 192.168.1.63
> （4）指定多个 IP 作为诱饵对 192.168.1.63 进行探测
> 		nmap -D 192.168.1.14,192.168.1.18 192.168.1.63
> ```

###### eg12：伪造源端口为 8888 对目标进行扫描

> ```
> 	nmap --source-port 8888 101.200.128.35
> 	nmap -g 8888 101.200.128.35
> ```

###### eg13：从互联网上随机选择 10 台主机扫描是否运行 Web 服务器（开放 80 端口）

> ```
> nmap -v -iR 10 -p 80
> ```

###### eg14：将所有主机视为联机，跳过主机发现，这种方式可以穿透防火墙，避免被防火墙发现

> ```
>  nmap -Pn 101.200.128.35
> ```

> ```
> -sA 扫描端口版本 
> -F 快速扫描
> -T4 加快执行速度
> -A 操作系统及版本探测	-A 完全扫描，对操作系统和软件版本号进行检测，并对目标进行 traceroute 路由探测。
> 					  -O 参数仅识别目标操作系统，并不做软件版本检测和路由探测。
> -v 显示详细的输出
> -sS TCP SYN 扫描
> -sU UDP 扫描
> -p 指定端口扫描范围
> -traceroute 显示本机到目标的路由跃点。
> --version-light 设定侦测等级为 2。
> ```
>

------



















