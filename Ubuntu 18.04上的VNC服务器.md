##Ubuntu 18.04上的VNC服务器

###目的
	目标是在Ubuntu 18.04 Bionic Beaver Linux上设置VNC服务器。
	操作系统和软件版本
	操作系统： - Ubuntu 18.04仿生海狸

##一 安装桌面环境

假如不安装桌面环境的话，VNC 连接后是灰屏什么也看不到的。

在这里介绍安装gnome。

	仅安装核心组件,假如不安装例如 office、浏览器、等等的额外组件，可以使用如下命令：
	sudo apt install --no-install-recommends ubuntu-desktop gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal -y

使用apt方式安装

对于一般软件来说，直接使用apt install 软件名就可以安装了，但是由于Ubuntu默认的软件下载源没有chrome的下载源，所以我们要手动添加chrome的下载源，然后再使用apt install 软件名来安装，全部命令如下：

	# 添加chrome下载源到系统下载源
	sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/

	# 导入Google软件的公钥
	wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -

	# 更新系统软件可用列表
	sudo apt update

	# 安装Chrome
	sudo apt install google-chrome-stable


##二 安装VNC服务器和Xfce桌面管理器核心文件：

	$ sudo apt install vnc4server xfce4 xfce4-goodies
	一旦安装了VNC服务器，我们就可以在创建远程连接时设置VNC客户端使用的用户密码来开始配置：
	
	设置密码并运行vncserver
	vncserver 
	
	配置xstartup文件
	修改xstartup文件内容，针对Xfce4桌面环境.
	修改~/.vnc/xstartup中的内容为：
	
	备份自动生成的原有文件
	cp /home/cpu/.vnc/xstartup  /home/cpu/.vnc/xstartup.bak
	
	
	修改文件
	sudo vim /home/cpu/.vnc/xstartup
	
	#!/bin/sh

	# Uncomment the following two lines for normal desktop:
	unset SESSION_MANAGER
	# exec /etc/X11/xinit/xinitrc
	unset DBUS_SESSION_BUS_ADDRESS  
	startxfce4 &

	[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
	[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
	xsetroot -solid grey
	vncconfig -iconic &
	x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
	x-window-manager &
	
	
	重启vncserver
	
	修改配置文件后，运行如下命令结束掉之前产生的窗口:1

	vncserver -kill :1  
	重新启动

	vncserver
	成功
	

	启动/home/cpu/.vnc/xstartup中指定的应用程序
	日志文件是/home/cpu/.vnc/ubuntu:1.log
	
	VNC服务器将为您创建的每个新VNC桌面打开一个新端口。您的Ubuntu系统现在应该在端口上监听5901传入的VNC连接：
	
	$ ss -ltn
	State       Recv-Q Send-Q Local Address:Port               Peer Address:Port
	LISTEN      0      128    0.0.0.0:22                 0.0.0.0:* 
	LISTEN      0      128    0.0.0.0:6001               0.0.0.0:* 
	LISTEN      0      128       [::]:22                    [::]:*  
	LISTEN      0      5            *:5901                     *:* 
	 
	如果您启用了UFW防火墙，请打开5901传入连接的端口，或参阅下面的介绍如何通过SSH协议传输VNC连接：
	$ sudo ufw允许从任何端口5901 proto tcp
	
	sudo ufw  allow 5901
	sudo ufw  allow 5902

	规则补充说
	添加规则（v6）

	查看规则
	sudo ufw  status


##三 连接到VNC服务器
    tigerVNC
    chrome 插件

##四 VNC服务器系统启动脚本

	尽管当前的配置有效，但可能需要设置systemd启动脚本以轻松管理多个VNC桌面会话。
	创建一个新文件，sudo vim /etc/systemd/system/vncserver@.service  例如：
	vncserver选项或参数：

	[Unit]
	Description=Start VNC server at startup
	After=syslog.target network.target
	
	[Service]
	Type=forking
	User=cpu
	PIDFile=/home/cpu/.vnc/%H:%i.pid
	#ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
	ExecStart=/usr/bin/vncserver -depth 24 -geometry 1918x1017 :%i
	ExecStop=/usr/bin/vncserver -kill :%i
	
	[Install]
	WantedBy=multi-user.target
	 

	保存并关闭文件。
	
	接下来，让系统知道新的单元文件。
	
	sudo systemctl daemon-reload
	启用单位文件。
	
	sudo systemctl enable vncserver@1.service
	停止VNC服务器的当前实例（如果它仍在运行）。
	
	vncserver -kill :1
	然后启动它，因为您将启动任何其他systemd服务。
	
	sudo systemctl start vncserver@1
	您可以验证它以此命令开始：
	
	sudo systemctl status vncserver@1

