##Centos 7 安装Python3.6 + Flask + Gunicorn + Nginx 

Flask 是一个免费的开源微博客框架，旨在帮助开发人员构建安全，可扩展和可维护的 Web 应用程序。 Flask 基于 Werkzeug 并使用 Jinja2 作为模板引擎。

与 Django 不同，默认情况下， Flask 不包含 ORM ，表单验证或第三方库提供的任何其他功能。 Flask 构建时考虑了扩展，这些扩展 Python 包为 Flask 应用程序添加功能。

根据您的需要，可以使用不同的方法安装 Flask 。它可以使用 pip 在系统范围内安装或在 Python 虚拟环境中安装。

Flask 包也包含在 EPEL 存储库中，可以使用 yum 包管理器进行安装。这是在 CentOS 7 上安装 Flask 的最简单方法，但不如在虚拟环境中安装那么灵活。此外，存储库中包含的版本始终落后于最新版本的 Flask 。

Python 虚拟环境的主要目的是为不同的 Python 项目创建一个独立的环境。这样，您可以在一台计算机上拥有多个不同的 Flask 环境，并在每个项目的基础上安装特定版本的模块，而不必担心它会影响您的其他 Flask 安装。如果将 Flask 安装到全局环境中，则只能在计算机上安装一个 Flask 版本

###1.安装Python 3

我们将从 Software Collections(SCL)存储库安装 Python 3.6 。

CentOS 7 附带 Python 2.7.5 ，这是 CentOS 基础系统的关键部分。
SCL 将允许您安装较新版本的 python 3.x 以及默认的 python v2.7.5 ，以便 yum 等系统工具可以继续正常工作。

通过安装 CentOS 附加存储库中包含的 CentOS SCL 发布文件来启用 SCL ：

	yum install centos-release-scl
启用存储库后，使用以下命令安装 Python 3.6 ：

	yum install rh-python36

###2.创建虚拟环境

一旦安装了 Python 3.6 ，我们就可以为 Flask 应用程序创建一个虚拟环境。

首先导航到要存储 Python 3 虚拟环境的目录。它可以是您的主目录或您的用户具有读写权限的任何其他目录。

要访问 Python 3.6 ，您需要使用该scl工具启动新的shell实例：

	scl enable rh-python36 bash
为 Flask 应用程序创建一个新目录并导航到它：

	mkdir /home/web
    cd /home/web
运行以下命令以创建新的虚拟环境：

	python3 -m venv venv
上面的命令将创建一个名为 venv 的目录，其中包含 Python 二进制文件的副本， Pip 包管理器，标准 Python 库和其他支持文件。您可以为虚拟环境使用任何名称。

使用 activate 脚本激活虚拟环境：

	source venv/bin/activate

	激活后，虚拟环境的 bin 目录将添加到 $PATH 变量的开头。
	此外，您的 shell 提示符也会更改，它将显示您当前使用的虚拟环境的名称。在我们的情况下是 venv

说明：退出虚拟环境：

	deactivate

###3.安装Flask

现在虚拟环境已激活，您可以使用 Python 包管理器 pip 来安装 Flask ：

	pip install flask

	在虚拟环境中，你可以使用命令 pip 来代替 pip3，用 python 代替 python3 。

使用以下命令验证安装，该命令将打印 Flask 版本：

	python -m flask --version
在撰写本文时，最新的官方 Flask 版本是 1.0.3

###4.安装Gunicorn

安装gunicorn，在以上虚拟环境中安装：

	pip install gunicorn

创建一个简单的app程序：onekeystop.py

	vim onekeystop.py

	from flask import Flask 
	app = Flask(__name__) 

	@app.route("/") 
	def hello(): 
    	return "<h1 style='color:red'>Hello Python!</h1>" 

	if __name__ == "__main__": 
    	app.run(host='0.0.0.0')

运行测试：

	python onekeystop.py 

访问Url地址：http://x.x.x.x:5000 成功！


启动：

	gunicorn -w 4 -b 0.0.0.0:8000 onekeystop:app

	参数说明：    
	使用8000端口进行访问，-w 表示开启了多少个 worker, -b 表示绑定的访问地址。
	onekeystop就是 onekeystop.py 的文件名，onekeystop.py 相当于一个库文件被 gunicorn 调用。
	app 则是 onekeystop.py 里创建的 app，这样 gunicorn 才可以定位 flask 应用。


后台启动

	nohup gunicorn -w 4 -b 0.0.0.0:8000 onekeystop:app&     //关闭远程连接时程序在后台继续运行

此时访问的时候不再是5000端口了，现在的端口是8000；

说明：关闭gunicorn 

	pkill gunicorn  //关闭gunicorn


为了方便添加一个开机启动gunicorn文件：
	
	vim /etc/systemd/system/onekeystop.service

	[Unit] 
	Description=Gunicorn instance to server onekeystop 
	After=network.target 

	[Service] 
	User=nginx 
	Group=nginx 
	PrivateTmp=true 
	WorkingDirectory=/home/web
	Environment="PATH=/home/web/venv/bin" 
	ExecStart=/home/web/venv/bin/gunicorn --workers 4 --bind 0.0.0.0:8000 onekeystop:app 

	[Install] 
	WantedBy=multi-user.target

启动程序：

	systemctl start onekeystop

查看运行状态：

	systemctl status onekeystop

添加开机启动：

	systemctl enable onekeystop


###5.安装Nginx

安装EPEL源

	yum -y install epel-release

安装Nginx

	yum install nginx -y
添加到开机启动：

	systemctl enable nginx 
	systemctl start nginx

编辑配置文件：
	vim /etc/nginx/conf.d/onekeystop.conf

	server {  
    	listen 80;  
    	server_name default_server;  


    	location / {  
        	proxy_set_header Host $http_host;  
        	proxy_set_header X-Real-IP $remote_addr;  
        	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
        	proxy_set_header X-Forwarded-Proto $scheme;  
        	proxy_pass http://127.0.0.1:8000;  
    	}  
	}

问题：nginx访问127.0.0.1会报502错误，直接访问x.x.x.x:8000正常

    处理：
    
	setsebool httpd_can_network_connect 1

	用内网IP，127.0.0.1,  0.0.0.0均正常
