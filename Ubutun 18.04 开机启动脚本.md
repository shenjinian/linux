##Ubutun 18.04 开机启动脚本
 
    ubutun 18.04不再使用initd管理系统，改用systemd。
    为了像以前一样，在/etc/rc.local中设置开机启动程序，需要以下几步：
 
##实现原理
    systemd默认会读取/etc/systemd/system/下的配置文件，该目录下的文件会链接到/lib/systemd/system/下的文件。
    一般系统安装完，/lib/systemd/system/下会有rc-local.service文件，即我们需要的配置文件。
 
###1.编辑/lib/systemd/system/rc-local.service文件
    sudo vim  /lib/systemd/system/rc-local.service
 
    # This unit gets pulled automatically into multi-user.target by
    # systemd-rc-local-generator if /etc/rc.local is executable.
    [Unit]
    Description=/etc/rc.local Compatibility
    Documentation=man:systemd-rc-local-generator(8)
    ConditionFileIsExecutable=/etc/rc.local
    After=network.target syslog.target remote-fs.target nss-lookup.target
 
    [Service]
    Type=forking
    ExecStart=/etc/rc.local start
    TimeoutSec=0
    RemainAfterExit=no
    GuessMainPID=no
 
    #这一段原文件没有，需要自己添加
    [Install]
    WantedBy=multi-user.target
    Alias=rc-local.service
 
####说明
一般正常的启动文件的主要分成3个部分
1) [Unit] 区块：启动顺序与依赖关系。 
2) [Service] 区块：启动行为,如何启动，启动类型。 
3) [Install] 区块，定义如何安装这个配置文件，即怎样做到开机启动。
 
###2.将/lib/systemd/system/rc-local.service链接到/etc/systemd/system/目录下。
    sudo ln -fs /lib/systemd/system/rc-local.service /etc/systemd/system/rc-local.service
 
###3.ubutun-18.04默认是没有/etc/rc.local这个文件的，需要创建
    sudo vim /etc/rc.local
 
    #!/bin/bash -e
    nohup jupyter notebook --ip=202.119.189.51  >/dev/null 2>&1 &
 
###4.赋可执行权限
    sudo chmod 755 /etc/rc.local
 
###4.重启
    sudo shutdown -r now
 
###6.总结
就是利用systemd的启动原理，通过/etc/systemd/system/rc-local.service文件来达到启动时执行/etc/rc.local文件的目的
