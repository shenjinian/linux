##CentOS7 安装 privoxy 和L2TP拨号

###一 安装和配置privoxy
####1.	安装privoxy 

yum install epel-release
yum install -y privoxy 

启动Privoxy

	systemctl enable privoxy
	systemctl start privoxy
	systemctl status privoxy

####2.配置Privoxy，使用 Privoxy 转发
#####配置Privoxy 

① 修改配置文件/etc/privoxy/config

	vim /etc/privoxy/config

	修改配置文件.
	vim /etc/privoxy/config 
	把 listen-address  127.0.0.1:8118
	改为 listen-address  0.0.0.0:8118       # 可以允许局域网中的连接


	防火墙开启TCP的8118端口

#####设置http/https代理 

① 修改配置文件/etc/profile

	vim /etc/profile
	
添加下面两句
	
	export http_proxy=http://127.0.0.1:8118
	export https_proxy=http://127.0.0.1:8118
	注：端口和privoxy 中的监听端口保持一致

 运行一下：

	source /etc/profile
	
###二 安装和配置L2TP VPN Client
####1.安装ppp、pptp、pptp-setup。

	yum install -y ppp pptp pptp-setup


####2.安装xl2tp
因为CentOS7默认的yum源里没有xl2tpd，需要先安装epel

	yum install -y epel
	yum install -y xl2tpd


####3.配置xl2tpd.conf
原始的xl2tpd.conf里面有[lns default]，这个是将xl2tpd当做l2tpd服务器的关键语句。 
要将xl2tpd作为l2tp的client端，需要把xl2tpd.conf里面的[lns default]部分删掉，加入[lac testvpn]部分。

	[root@localhost ~]# vim /etc/xl2tpd/xl2tpd.conf
	
	[lac njeu_vpn]
	name = xxx                                 ;L2TP的账号
	lns = X.X.X.X                             ;L2TP的服务器IP
	pppoptfile = /etc/ppp/peers/njeu_vpn.l2tpd       ;PPPD拨号时的配置文件
	ppp debug = no
	redial=yes
	redial timeout=15
	max redials=5
	require pap=no
	require chap=yes
	require authentication=yes

####4.设置拨号配置文件

文件路径：xl2tpd.conf文件中pppoptfile =/etc/ppp/peers/njeu_vpn.l2tpd

	
	[root@localhost ~]# vim /etc/ppp/peers/njeu_vpn.l2tpd  
	
	remotename njeu_vpn
	user "xxx"
	password *******"
	ipcp-accept-local
	ipcp-accept-remote
	refuse-eap
	require-mschap-v2
	noccp
	noauth
	noipdefault
	mtu 1410
	mru 1410
	usepeerdns
	debug
	connect-delay 5000

####5.启动xl2tpd（注意启动不代表拨号）

	systemctl start xl2tpd


####6.开始拨号，连接VPN服务器。

	echo 'c njeu_vpn' > /var/run/xl2tpd/l2tp-control

拨号成功的话，通过ifconfig可以看见有个ppp0的接口

	[root@localhost ~]# ifconfig ppp0
		
断开拨号命令：

	echo 'd njeu_vpn' > /var/run/xl2tpd/l2tp-control


####7.添加路由

l2tp连接上后，需要数据通过此ppp0接口出去的话，就需要配置路由了。

删除原来默认路由

	route del -net default 

添加路由

	route add -net  X.X.X.X netmask 255.255.255.255 gw 本地的网关IP
	route add -net default metric 10  dev ppp0

	说明：
	X.X.X.X  为L2TP服务器的IP，gw为 本地的网关IP

####8.验证是否可用

	curl ip.cn      # 查看ip是否是你的服务器地址