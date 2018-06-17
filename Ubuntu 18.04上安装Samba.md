## Ubuntu 18.04上安装Samba ##

Samba是一款开源软件，可为SMB / CIFS客户端提供无缝文件和打印服务。

Samba使Linux系统（包括Ubuntu）能够与Windows系统（包括Windows 10）共享文件。

## 第1步：在UBUNTU上安装SAMBA ##

安装Samba

	sudo apt install samba samba-common python-glade2 system-config-samba


## 第2步：配置SAMBA公用共享 ##

备份其默认配置文件

	sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak

新建Samba配置文件。
	
	sudo vim /etc/samba/smb.conf

将以下内容复制并粘贴到文件中并保存，这将创建一个名为Public的共享，其中每个人都可以在Ubuntu上访问它...
	#============================ Global definition ================================
	 
	[global]
	workgroup = WORKGROUP
	server string = Samba Server %v
	netbios name = ubuntu—shen
	security = user
	map to guest = bad user
	name resolve order = bcast host
	dns proxy = no
	bind interfaces only = yes
	
	#============================ Share Definitions ============================== 
	
	[Public]
	   path = /data/public
	   writable = yes
	   guest ok = yes
	   guest only = yes
	   read only = no
	   create mode = 0777
	   directory mode = 0777
	   force user = nobody

保存您的更改

## 第3步：创建公用文件夹以进行共享 ##

创建公共文件夹，每个人都可以访问上面Samba配置中定义的

	sudo mkdir -p /data/public

设置权限，以便每个人都可以读取和写入。

	sudo chown -R nobody：nogroup /data/public 
	sudo chmod -R 0775 /data/public

重新启动Samba并打开Windows文件资源管理器以查看Ubuntu上的共享位置

	sudo service smbd restart

现在进入你的Windows机器，当你浏览文件管理器时，你会看到Ubuntu上的共享公用文件夹。

 
## 第4步：配置SAMBA专用共享 ##

私有和受保护的共享，只有属于已批准组的用户才能使用密码访问安全位置。

首先创建一个名为smbgroup的samba组作为共享，只有成员才能访问。

要在Ubuntu中创建组，请运行以下命令。

	sudo addgroup smbgroup

然后通过运行下面的命令将用户添加到组中

	sudo usermod -aG smbgroup cpu

最后，所有需要访问受保护的samba共享的用户都需要输入密码。要将用户添加到samba密码数据库，请为每个用户运行以下命令。

	sudo smbpasswd -a cpu

系统将提示用户输入并确认密码。此密码将用于访问受保护的Samba共享。

接下来，在/ samba目录中创建一个受保护的共享。
	
	sudo mkdir -p /data/cpu

然后只给根和成员组访问此份额。

	cd /data
	sudo chown -R cpu:smbgroup /data
	sudo chmod -R 0770 /data

完成创建受保护共享后，请转到smb.conf文件中进行共享。

	sudo vim /etc/samba/smb.conf

然后将配置块添加到上面的smb.conf文件中

	[shen]
	  path = /data
	  valid users = @smbgroup
	  guest ok = no
	  writable = yes
	  browsable = yes

保存更改并重新启动Samba

	sudo service smbd restart


## 第5步：配置ufw防火墙 ##

1.如果允许所有IP访问此smb服务，开放139de TCP即可

	sudo ufw port 139/tcp

2.如果允许指定IP访问此smb服务，命令如下

	sudo ufw allow from 202.119.189.50 to any port 139

