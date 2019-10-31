## CentOS8下Rsync的安装与配置 ##

###1. Server端

####1.1安装rsync

	dnf install rsync -y

rsync的配置文件默认不存在，所以需要手动创建。Rsync有三个配置文件：

	rsyncd.conf：主配置文件
	rsyncd.secrets：密码文件
	rsyncd.motd：服务器信息文件

####1.2 创建主配置文件

**说明：配置文件中的#注释要去掉，否则设置开机启动会失败**

	vim /etc/rsyncd.conf

	# 常见全局参数：
	lock file = /var/run/rsync.lock  #设置锁文件
	log file = /var/log/rsyncd.log   #设置日志文件
	pid file = /var/run/rsyncd.pid   #设置进程号文件
	motd file = /etc/rsyncd.motd     #设置服务器信息提示文件
	
	port = 873                       #监听端口
	uid = root                       #数据传输时使用的用户ID，默认nobody
	gid = root                       #数据传输时使用的组ID，默认nobody
	read only = no                   #是否只读
	write only = no
	list = no                        #是否允许查看模块信息
	max connections = 10             #并发连接数，0表示无限制。超出并发连接数时若再访问，会收到稍后重试的消息
	use chroot = yes                 #是否开启chroot，若设为yes，rsync会首先进行chroot设置，将根映射到模块中path指定目录
	                                 #需要root权限，在同步符号连接资料时仅同步文件名，不同步文件内容
	hosts allow=202.119.11.100    #设置允许访问服务器的主机
	hosts deny=*
	
	#transfer logging = yes          #开启Rsync数据传输日志功能
	#log format = %t %a %m %f %b     #设置日志格式，默认log格式为："%o %h [%a] %m (%u) %f %l"
	# 默认log格式表达的是："操作类型 远程主机名[远程IP地址] 模块名 (认证的用户名) 文件名 文件长度字符数"
	#syslog facility =  local3       #指定rsync发送日志消息给syslog时的消息级别，默认值是daemon
	#timeout = 600                   #可以覆盖客户指定的 IP 超时时间。确保rsync服务器不会永远等待一个崩溃的客户端。
	                                 #超时单位为秒钟，0表示没有超时定义，这也是默认值。
	                                 #对于匿名rsync服务器来说，一个理想的数字是600
	
	# 模块配置
	[dns100]
	comment = 202.119.11.100       #添加注释
	path = /home/www/html/dns100     #同步文件或目录路径
	auth users = root               #允许连接的用户，可以是系统中不存在的
	secrets file = /etc/rsyncd.secrets   #设置密码验证文件，该文件的权限要求为只读（600），该文件仅在设置了auth users才有效
	ignore errors                   #忽略一些IO错误
	# exclude =                      #可以指定不同步的目录或文件，使用相对路径
	
	[dns100deny]
	comment = 202.119.11.100
	path = /home/www/html/dns100deny
	auth users = root             
	secrets file = /etc/rsyncd.secrets  
	ignore errors        
	# exclude =        
	
	[dns101]
	comment = 202.119.11.101     
	path = /home/www/html/dns101         
	auth users = root             
	secrets file = /etc/rsyncd.secrets  
	ignore errors                    
	# exclude =                    
	
	[dns101deny]
	comment = 202.119.11.101
	path = /home/www/html/dns101deny
	auth users = root             
	secrets file = /etc/rsyncd.secrets  
	ignore errors                    
	# exclude =        
	
	[nginx80]
	comment = 202.119.11.80
	path = /home/www/html/nginx80
	auth users = root             
	secrets file = /etc/rsyncd.secrets  
	ignore errors                    
	# exclude =    


####1.3 创建密码文件

	useradd rsync_user && echo "rsync_password" | passwd rsync_user --stdin  创建用户和密码
	echo "rsync_user:rsync_password" >> /etc/rsyncd.secrets    添加用户到密码文件

这里直接使用系统root的账号:

	echo "root:Cpu" >> /etc/rsyncd.secrets 
	chmod 600 /etc/rsyncd.secrets    修改密码文件权限
	more /etc/rsyncd.secrets        查看密码文件

####1.4 创建服务器信息文件【可选】

	echo "Welcome to Rsync" >> /etc/rsyncd.motd

####1.5 创建相应目录

	mkdir -p /home/www/html/dns100
	mkdir -p /home/www/html/dns100deny
	mkdir -p /home/www/html/dns101
	mkdir -p /home/www/html/dns101deny
	mkdir -p /home/www/html/nginx80

####1.6 防火墙开放873端口
	
	firewall-cmd --add-port=873/tcp --permanent
	firewall-cmd --add-port=873/udp --permanent
	firewall-cmd --reload

####1.7 永久改变 SELinux 状态：

	vi  /etc/sysconfig/selinux 
	将SELINUX=enforcing
	改为SELINUX=disabled
	：wq
	重启

####1.8 启动和停止

选择rsync服务器启动方式

	rsync服务器负载比较高，则使用独立启动模式
	rsync服务器负载较低，使用xinetd运行方式

**独立启动**

	/usr/bin/rsync --daemon  --config=/etc/rsyncd.conf
	#--config用于指定rsyncd.conf的位置,如果在/etc下可以不写

**停止**

	kill $(cat /var/run/rsyncd.pid)
	ps aux | grep rsync

###2. client端

####2.1安装rsync

	dnf install rsync -y

####2.2 创建密码文件

	echo "Cpu" >> /etc/rsyncd.secrets 
	chmod 600 /etc/rsyncd.secrets 

####2.3 客户端同步

	rsync -vzrtopg --progress --password-file=/etc/rsyncd.secrets  /var/named/chroot/var/log/query.log root@202.119.191.86::dns100
	rsync -vzrtopg --progress --password-file=/etc/rsyncd.secrets  /var/named/chroot/var/log/default.log root@202.119.191.86::dns100deny

####2.4 crontab创建定时任务

	crontab -e
	
	*/5 * * * *  rsync -vzrtopg --progress --password-file=/etc/rsyncd.secrets  /var/named/chroot/var/log/query.log root@202.119.11.86::dns100
	*/5 * * * *  rsync -vzrtopg --progress --password-file=/etc/rsyncd.secrets  /var/named/chroot/var/log/default.log root@202.119.11.86::dns100deny


**查看**
	crontab -l


###3.服务器端设置开机自启动

####3.1编辑服务

	[root@~]# vim /usr/lib/systemd/system/rsyncd.service

	[Unit]
	Description=Rsync service
	ConditionPathExists=/etc/rsyncd.conf
	After=network.target remote-fs.target nss-lookup.target
	[Service]
	Type=forking
	ExecStart=/usr/bin/rsync --daemon --config=/etc/rsyncd.conf 
	ExecStop=kill $(cat /var/run/rsyncd.pid)
	[Install]
	WantedBy=multi-user.target


####3.2设置开机自启动

	[root@ ~]# systemctl enable rsyncd.service

####3.3查看设置状态

	[root@oldboy ~]# systemctl status rsyncd.service

	3.4 使用systemctl启动rsync并查看状态

	[root@ ~]# systemctl start rsyncd.service
	[root@ ~]# systemctl status rsyncd.service

####3.5使用lsof检查真实服务状态

	[root@oldboy ~]# lsof -i :873 #<==需要yum install lsof 

####3.6使用systemctl停止rsync并查看状态

	[root@oldboy ~]# systemctl stop rsyncd.service
	[root@oldboy ~]# lsof -i :873

###4.参数说明

**rsync有六种不同的工作模式：**

	　　1. rsync [OPTION]... SRC [SRC]... DEST
	　　拷贝本地文件；当SRC和DES路径信息都不包含有单个冒号":"分隔符时就启动这种工作模式。
	　　2. rsync [OPTION]... SRC [SRC]... [USER@]HOST:DEST
	　　用一个远程shell程序（如rsh、ssh）来实现将本地机器的内容拷贝到远程机器。当DST路径地址包含单个冒号":"分隔符时启动该模式。
	　　3. rsync [OPTION]... [USER@]HOST:SRC DEST
	　　使用一个远程shell程序（如rsh、ssh）来实现将远程机器的内容拷贝到本地机器。当SRC地址路径包含单个冒号":"分隔符时启动该模式。 
	　　4. rsync [OPTION]... [USER@]HOST::SRC [DEST]
	  从远程rsync服务器中拷贝文件到本地机。当SRC路径信息包含"::"分隔符时启动该模式。
	　　5. rsync [OPTION]... SRC [SRC]... [USER@]HOST::DEST
	  从本地机器拷贝文件到远程rsync服务器中。当DST路径信息包含"::"分隔符时启动该模式。 
	　　6. rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST]
	  列远程机的文件列表。这类似于rsync传输，不过只要在命令中省略掉本地机信息即可。
  

**参数详解编辑**

	-v, --verbose        详细模式输出
	-q, --quiet         精简输出模式
	-c, --checksum       打开校验开关，强制对文件传输进行校验
	-a, --archive       归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD
	-r, --recursive      对子目录以递归模式处理
	-R, --relative       使用相对路径信息
	-b, --backup        创建备份，也就是对于目的已经存在有同样的文件名时，将老的文件重新命名为~filename。可以使用--suffix选项来指定不同的备份文件前缀。
	--backup-dir        将备份文件(如~filename)存放在在目录下。
	-suffix=SUFFIX        定义备份文件前缀
	-u, --update        仅仅进行更新，也就是跳过所有已经存在于DST，并且文件时间晚于要备份的文件。(不覆盖更新的文件)
	-l, --links         保留软链结
	-L, --copy-links      想对待常规文件一样处理软链结
	--copy-unsafe-links    仅仅拷贝指向SRC路径目录树以外的链结
	--safe-links         忽略指向SRC路径目录树以外的链结
	-H, --hard-links      保留硬链结
	-p, --perms         保持文件权限
	-o, --owner        保持文件属主信息
	-g, --group        保持文件属组信息
	-D, --devices        保持设备文件信息
	-t, --times         保持文件时间信息
	-S, --sparse         对稀疏文件进行特殊处理以节省DST的空间
	-n, --dry-run        现实哪些文件将被传输
	-W, --whole-file      拷贝文件，不进行增量检测
	-x, --one-file-system   不要跨越文件系统边界
	-B, --block-size=SIZE   检验算法使用的块尺寸，默认是700字节
	-e, --rsh=COMMAND     指定使用rsh、ssh方式进行数据同步
	--rsync-path=PATH     指定远程服务器上的rsync命令所在路径信息
	-C, --cvs-exclude     使用和CVS一样的方法自动忽略文件，用来排除那些不希望传输的文件
	--existing         仅仅更新那些已经存在于DST的文件，而不备份那些新创建的文件
	--delete          删除那些DST中SRC没有的文件
	--delete-excluded     同样删除接收端那些被该选项指定排除的文件
	--delete-after       传输结束以后再删除
	--ignore-errors      及时出现IO错误也进行删除
	--max-delete=NUM     最多删除NUM个文件
	--partial         保留那些因故没有完全传输的文件，以是加快随后的再次传输
	--force           强制删除目录，即使不为空
	--numeric-ids       不将数字的用户和组ID匹配为用户名和组名
	--timeout=TIME      IP超时时间，单位为秒
	-I, --ignore-times    不跳过那些有同样的时间和长度的文件
	--size-only        当决定是否要备份文件时，仅仅察看文件大小而不考虑文件时间
	--modify-window=NUM    决定文件是否时间相同时使用的时间戳窗口，默认为0
	-T --temp-dir=DIR     在DIR中创建临时文件
	--compare-dest=DIR    同样比较DIR中的文件来决定是否需要备份
	-P               等同于 --partial
	-z, --compress      对备份的文件在传输时进行压缩处理
	--exclude=PATTERN     指定排除不需要传输的文件模式
	--include=PATTERN     指定不排除而需要传输的文件模式
	--exclude-from=FILE    排除FILE中指定模式的文件
	--include-from=FILE    不排除FILE指定模式匹配的文件
	--version         打印版本信息
	--address          绑定到特定的地址
	--config=FILE       指定其他的配置文件，不使用默认的rsyncd.conf文件
	--port=PORT        指定其他的rsync服务端口
	--blocking-io       对远程shell使用阻塞IO
	-stats           给出某些文件的传输状态
	--progress          在传输时显示传输过程
	--log-format=formAT    指定日志文件格式
	--password-file=FILE   从FILE中得到密码
	--bwlimit=KBPS       限制I/O带宽，KBytes per second
	-h, --help          显示帮助信
	
