---
title: MySQL主从同步配置
tag:
	- MySQL
	- 架构
	- 主从同步
---



### 安装VMS虚拟机+ubuntu server



**第一台**

安装：`multipass launch --name server8081`

运行：`multipass shell server8081`

IP：192.168.64.2



**第二台**

安装：`multipass launch --name server8082`

运行：`multipass shell server8082`

IP：192.168.64.3



**第三台**

安装：`mps launch jammy --name server8083 `

运行：`mps shell server8083`

IP：192.168.64.4



启动/关闭/暂停所有实例

```
mps start --all
mps stop --all
mps suspend --all
```




在虚拟机上安装mysql

  ```
  sudo apt update
    sudo apt install mysql-server
    sudo systemctl status mysql
    
    /etc/mysql/mysql.cnf
    /lib/systemd/system/mysql.service
    /var/log/mysql/error.log
```

配置账户
```
    登录mysql：sudo mysql
    配置账户：ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
    flush privileges;
    



    授权远程访问：
    登录服务：mysql -u root -p
    use mysql;
    update user set host = '%' where user = 'root';
    
    ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';



    flush privileges;
    
    修改
    sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
    bind-address配置改为IP地址192.168.64.4
    
    flush privileges;



mysql启动

    sudo systemctl restart mysql
    sudo systemctl start mysql
    sudo systemctl stop mysql

```

## 主从同步



server8081设为主库，server8082、server8083为从库；



### 主从配置

[https://segmentfault.com/a/1190000022115647](https://segmentfault.com/a/1190000022115647)

注意在 从库配置时

    change master to master_host='192.168.64.2' , master_port=3306, master_user='root',master_password='123456', master_log_file = 'mysql-bin.000002',master_log_pos= 2066;



master*log*pos的值要与主库对应，可以在主库通过命令show master status;查看对应的值是多少。

