##如何在Ubuntu 16.04上使用UFW设置防火墙

  UFW或Uncomplicated Firewall是iptables的接口，旨在简化配置防火墙的过程。 

一个带有sudo非root用户的Ubuntu 16.04服务器，您可以通过使用Ubuntu 16.04教程的初始服务器设置中的第1步-3进行设置 。

UFW默认安装在Ubuntu上。如果由于某种原因卸载它，你可以使用sudo apt install ufw 。

##第1步 - 使用IPv6与UFW（可选）

本教程是用IPv4编写的，但将适用于IPv6，只要您启用它。

如果您的Ubuntu服务器启用了IPv6，请确保UFW配置为支持IPv6，以便它将管理除IPv4之外的IPv6的防火墙规则。

为此，请使用vim或您喜欢的编辑器打开UFW配置。

	sudo vim /etc/default/ufw

然后确保IPV6值为yes 。它应该看起来像这样：

	sudo vim /etc/default/ufw
	...
	IPV6=yes
	...
保存并关闭文件。

现在，当启用UFW时，将配置为写入IPv4和IPv6防火墙规则。

但是，在启用UFW之前，我们将确保您的防火墙配置为允许您通过SSH连接。

让我们开始设置默认策略。

##第2步 - 设置默认策略

默认情况下，UFW设置为拒绝所有传入连接，并允许所有传出连接。

要设置UFW使用的默认值，请使用以下命令：

	sudo ufw default deny incoming
	sudo ufw default allow outgoing

##第3步 - 允许SSH连接

如果我们现在启用了UFW防火墙，它将拒绝所有传入的连接。

需要允许传入的SSH连接，以便连接和管理服务器。 可以使用以下命令：

	sudo ufw allow ssh

允许端口22上的所有连接，这是默认情况下SSH守护程序监听的端口。

但是，我们实际上可以通过指定端口。例如，此命令的工作原理与上述相同：

	sudo ufw allow 22

##第4步 - 启用UFW

要启用UFW，请使用以下命令：

	sudo ufw enable

您将收到一条警告，指出该命令可能会中断现有的SSH连接。

我们已经设置了允许SSH连接的防火墙规则，因此应该继续。

随意运行sudo ufw status verbose命令以查看设置的规则。

##第5步 - 允许其他连接

端口80上的HTTP，这是未加密的Web服务器使用的，使用

	sudo ufw allow http
	或
	sudo ufw allow 80

HTTPS端口443，这是加密的Web服务器使用，使用

	sudo ufw allow https
	或
	sudo ufw allow 443

FTP在端口21，用于未加密的文件传输（您可能不应该使用），使用

	sudo ufw allow ftp
	或
	sudo ufw allow 21/tcp

除了指定端口或已知服务之外，还有其他几种允许其他连接的方法。

###特定端口范围

您可以使用UFW指定端口范围。一些应用程序使用多个端口，而不是单个端口。 例如，
要允许使用端口6000 - 6007 X11连接，请使用以下命令：

	sudo ufw allow 6000:6007/tcp
	sudo ufw allow 6000:6007/udp

当使用UFW指定端口范围时，必须指定规则应应用于的协议（ tcp或udp ）。

我们以前没有提到过，因为没有指定协议只允许两个协议，这在大多数情况下是OK的。

###特定IP地址

使用UFW时，还可以指定IP地址。

例如，如果要允许来自特定IP地址（例如工作或家庭IP地址为15.15.15.51 ，则需要指定from ，然后指定IP地址：

	sudo ufw allow from 15.15.15.51

您还可以通过添加to any port后面跟端口号to any port指定允许IP地址连接的特定端口。

例如，如果要允许15.15.15.51连接到端口22 （SSH），请使用以下命令：

	sudo ufw allow from 15.15.15.51 to any port 22

###子网

如果要允许IP地址的子网，您可以使用CIDR表示法来指定网络掩码。

例如，如果要允许所有的IP地址范围从15.15.15.1到15.15.15.254您可以使用此命令：

	sudo ufw allow from 15.15.15.0/24

同样，您也可以指定允许子网15.15.15.0/24连接到的目标端口。 再次，我们将使用端口22 （SSH）作为示例：

	sudo ufw allow from 15.15.15.0/24 to any port 22

###连接到特定的网络接口

如果要创建仅适用于特定网络接口的防火墙规则，可以通过指定“allow in on”，然后指定网络接口的名称来实现。 

在继续之前，您可能需要查找网络接口。为此，请使用以下命令：

	ip addr

	...
	2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state
	...
	3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default 
	...

突出显示的输出指示网络接口名称。它们通常命名为像eth0或eth1 。 

因此，如果您的服务器有一个名为eth0的公共网络接口，您可以使用此命令允许HTTP流量（端口80 ）：

	sudo ufw allow in on eth0 to any port 80

这样做将允许您的服务器从公共Internet接收HTTP请求。 

或者，如果您希望您的MySQL数据库服务器（端口3306 ）监听专用网络接口eth1上的eth1 ，例如，您可以使用以下命令：

	sudo ufw allow in on eth1 to any port 3306

这将允许您的专用网络上的其他服务器连接到您的MySQL数据库

##第6步 - 拒绝连接

如果您尚未更改传入连接的默认策略，则UFW将配置为拒绝所有传入连接。

通常，通过要求创建明确允许特定端口和IP地址通过的规则，这简化了创建安全防火墙策略的过程。 

但是，有时您会希望根据源IP地址或子网拒绝特定连接，也许是因为您知道您的服务器正在从那里受到攻击。

此外，如果您希望将默认传入策略更改为允许 （这对于安全性不推荐），您需要为您不希望允许连接的任何服务或IP地址创建拒绝规则。 

要编写拒绝规则，可以使用上述命令，将allow替换为deny 。

例如，要拒绝HTTP连接，可以使用以下命令：

	sudo ufw deny http

或者如果你想拒绝15.15.15.51的所有连接，你可以使用这个命令：

	sudo ufw deny from 15.15.15.51

现在让我们来看看如何删除规则。

##第7步 - 删除规则
有两种不同的方法指定要删除的规则：按规则编号或实际规则（类似于创建规则时指定的规则）。

我们将从规则编号方法开始，因为如果你是UFW的新手，相比编写要删除的实际规则更容易。

###按规则编号

如果您使用规则编号删除防火墙规则，您首先要做的是获取防火墙规则列表。

UFW状态命令有一个选项，可以显示每个规则旁边的数字，如下所示：

	sudo ufw status numbered

	Status: active
	
	     To                         Action      From
	     --                         ------      ----
	[ 1] 22                         ALLOW IN    15.15.15.0/24
	[ 2] 80                         ALLOW IN    Anywhere

如果我们决定删除规则2，允许端口80（HTTP）连接，我们可以在UFW删除命令中指定它，如下所示：

	sudo ufw delete 2

这将显示确认提示，然后删除规则2，它允许HTTP连接。

请注意，如果启用了IPv6，则也要删除相应的IPv6规则。

###按实际规则

规则编号的替代方法是指定要删除的实际规则。

例如，如果要删除allow http规则，您可以这样写：

	sudo ufw delete allow http

您还可以通过allow 80指定规则，而不是按服务名称指定规则：

	sudo ufw delete allow 80

此方法将删除IPv4和IPv6规则（如果存在）。

###第8步 - 检查UFW状态和规则

在任何时候，您可以使用此命令检查UFW的状态：

	sudo ufw status verbose

如果UFW被禁用，默认情况下，你会看到这样：

	Status: active
	Logging: on (low)
	Default: deny (incoming), allow (outgoing), disabled (routed)
	New profiles: skip
	
	To                         Action      From
	--                         ------      ----
	22/tcp                     ALLOW IN    Anywhere

如果要检查UFW如何配置防火墙，请使用status命令。

##第9步 - 禁用或重置UFW（可选）
如果您决定不想使用UFW，可以使用此命令禁用它：

	sudo ufw disable

使用UFW创建的任何规则将不再处于活动状态。

如果需要稍后激活它，可以总是运行

	sudo ufw enable 。 

如果已配置UFW规则，但您决定要重新开始，可以使用reset命令：

	sudo ufw reset

这将禁用UFW并删除以前定义的任何规则。

请注意，如果您在任何时间修改默认策略，默认策略将不会更改为原始设置。

这应该给你一个新的开始与UFW。

##结论

您的防火墙现在应该配置为允许（至少）SSH连接。

请确保允许您的服务器的任何其他传入连接，同时限制任何不必要的连接，以便您的服务器可以正常工作和安全。
