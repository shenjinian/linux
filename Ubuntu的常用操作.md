##Ubuntu的操作

###系统重启
    sudo shutdown -r now

###服务重启
    sudo systemctl restart smbd.service 

###查看网络地址
    ip addr show
    ip -s link
    ss

 
##防火墙
###防火墙状态
    sudo ufw status
    sudo ufw enable
    sudo ufw disable

###防火墙规则添加
    sudo ufw  allow from 202.119.188.50 to any port 445
###防火墙规则删除
    sudo ufw  delete allow from 202.119.188.50 to any port 445

##更新系统及软件
###更新源
    sudo apt-get update
###更新已安装的包
    sudo apt-get upgrade
###升级系统
    sudo apt-get dist-upgrade

##Anaconda更新
###指定更新包
    conda update anaconda
###更新所有的包
    conda update --all

##jupyter notebook后台启动
    nohup jupyter notebook --ip=202.119.189.51  >/dev/null 2>&1 & 








