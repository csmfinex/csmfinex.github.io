# Google 常用语法说明

```
site			#指定域名		
inurl URL 		#中存在的关键字页面	
intext 			#网页内容里面的关键字
Filetype 		#指定文件类型
intitle 		#网页标题中的关键字  intitle:index.of .bash_history
intitle 		#表示标题	
link 			#返回你所有的指定域名链接
info 			#查找指定站点信息
cache 			#搜索 Google 里的内容缓
```


	eg1 ：
		inurl（拆开来，就是 in url ）	   #它的作用是限定在 url 中搜索。
		inurl:qq.txt					#可以看到有很多存在 QQ 账号密码信息的页面
		inurl:admin_login.asp			#查找后台登录页；可以看到全部都是后台登录的页面。
	
	eg2 ：      
		intitle:index.of .bash_history
		
		index.of :表示包含index.of字段，出现该字段表示网站目录是对我们开放的、我们可以查看到网站目录下的所有文件信息。
		.bash_history :表示我们要筛选的文件名称，也可以替换成其他的敏感信息文件，该文件记录了用户的历史命令记录；   
				##http://www.lamardesigngroup.com/homedir/##
		 my.cnf					#查找 mysql 的配置文件
		 config_global.php		#查找 discuz 论坛中存储 mysql 密码的配置文件
	
	eg3	：
		cache:xuegod.cn
		
		cache 返回的结果是被搜索引擎收录时的页面，比如一些页面被删除了，我们通过 cache 还是可以访问。
		我们看到 Google 缓存的日期为 6 月 3 日，他也会提示我们当前页面可能已经更改，所以我们查看一些被删除的内容是完全可能的
	
	eg4	：
		Kali filetype:torrent
		
		Kali 		#是我们要搜索的关键字，至于同学们关心什么奇怪的内容老师就不知道了。手动斜眼笑。
		filetype 	#指定文件类型
		torrent 	#文件类型名称，torrent 是种子文件，可以填写任意扩展名
		我们可以找到很多 Kali 相关的种子文件，当然这些就比较老了，不建议下载使用，使用官方版本即可。
	
	eg5	:
		apache site:bbs.xuegod.cn
		
		apache 			#是我们搜索的关键字
		site 			#可以查询网站的收录情况
		bbs.xuegod.cn   #目标网站：学神官方论坛
		使用场景，我们通常在一些网站中找到一些有用的信息是非常麻烦的 ，因为站内的搜索功能并不是那么好用，所以我们使用该方式可以快速的查找到自己想要的信息。（可以看到搜索到的内容都包含了apache.）

```
eg6	：
	intext:user.sql intitle:index.of	#组合使用技巧
	intext:user.sql 					#查询包含 user.sql 用户数据库信息的页面
	intitle:index.of 					#表示网站目录是开放状态
	-------我们可以看到有很多页面都包含了敏感信息。-------
	http://western-blotting.com/oc_v1.5.2.1_database_dump/	#这应该是一个备份数据库的地方
```

	eg7	：
		s3 site:amazonaws.com filetype:xls password
		
		s3 						#关键字，亚马逊的一种服务器类型。
		site:amazonaws.com 		#目标是亚马逊云平台。
		filetype:xls password 	#文件类型 xls 包含密码的文件。
		------我们点开一些链接会自动下载 xls 的表格文件文件中包含了用户名密码信息。
		------这里给大家提个醒，不要下载什么文件都直接打开，一定要开启杀毒软件，原因是黑客会利用我们这种行为，伪造一个包含信息的页	------面，我们打开页面后下载的文件就有可能是包含病毒的文件。

> 总结：	关键字 site:目标网站 filetype：文件格式    
> 				intitle:index.of 表对外开放的网站目录 后可以+要查询的文件
> 				cache:目标网站 			查看目标的缓存
> 				intext：xxx 				   查找包含xxx的页面
> 				inurl:文件名称 		   	在URL中查询该类文件