## 从无到有的LMP部署

 

## 一、基本环境

### **1.** 操作系统

https://releases.ubuntu.com/18.04/

 ![image-20211210164338265](https://gitee.com/barryx/kernel_study/raw/master/img/202112101643495.png)

### **2.** ssh

```
root@ubuntu:~# apt-get install openssh-server

root@ubuntu:~# apt install vim -y

root@ubuntu:~# vim ~/.bashrc

	alias vi='vim'

root@ubuntu:~# vim /etc/ssh/sshd_config

	PermitRootLogin yes

root@ubuntu:~# /etc/init.d/ssh restart

[ ok ] Restarting ssh (via systemctl): ssh.service.
```



### **3.** IP

```
root@ubuntu:/etc/network# ip route show

default via 192.168.145.2 dev ens33 proto dhcp metric 100 

169.254.0.0/16 dev **ens33** scope link metric 1000 

192.168.145.0/24 dev ens33 proto kernel scope link src 192.168.145.134 metric 100

root@ubuntu:~# vim /etc/network/interfaces

	auto ens33

iface ens33 inet static

address 192.168.145.20

netmask 255.255.255.0

gateway 192.168.145.2

root@ubuntu:/etc/network# service networking restart
```

----这个方法配完之后连不上网了

使用另一种：

```
root@ubuntu:/etc/netplan# ls -l

-rw-r--r-- 1 root root 213 Nov 11 23:07 01-network-manager-all.yaml
```

添加：

```
 ethernets:

     ens33:

         dhcp4: true

         addresses: [192.168.145.20/24]
```

表示使用DHCP，但是自己多配了一个IP

注意前面要用空格隔开，不能用Tab

```
root@ubuntu:/etc/netplan# netplan apply
```



### 4.golang

https://studygolang.com/dl

![image-20211210164347833](../../../../../../../../../../../../Application%20Support/typora-user-images/image-20211210164347833.png)

放在/usr/local/下

```
root@ubuntu:/usr/local/go# tar -zxvf go1.16.10.linux-amd64.tar.gz 
```

安装参考https://www.jianshu.com/p/33cf4f41cae9

```
root@ubuntu:/usr/local/go# vim /etc/profile

export GOROOT=/usr/local/go

export GOPATH=/goProject

export GOBIN=$GOPATH/bin

export PATH=$PATH:$GOROOT/bin

export PATH=$PATH:$GOPATH/bin
```

以防万一建了以下目录：

![image-20211210164424853](https://gitee.com/barryx/kernel_study/raw/master/img/202112101644894.png)



```
root@ubuntu:/# source /etc/profile

root@ubuntu:/# go version

go version go1.16.10 linux/amd64
```

表示已经搭好了

 

### **5.** MySQL

https://downloads.mysql.com/archives/community/

查看有无冲突数据库

```
root@ubuntu:/usr/local# apt install rpm

root@ubuntu:/usr/local# rpm -qa|grep mysql

root@ubuntu:/usr/local# rpm -qa|grep mariadb
```

![image-20211210164436115](https://gitee.com/barryx/kernel_study/raw/master/img/202112101644149.png)

安装参考https://blog.csdn.net/qq_35977117/article/details/120194852

**tar**

-c: 建立压缩档案

-x：解压

-t：查看内容

-r：向压缩归档文件末尾追加文件

-u：更新原压缩包中的文件

这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。下面的参数是根据需要在压缩或解压档案时可选的。

-z：有gzip属性的

-j：有bz2属性的

-Z：有compress属性的

-v：显示所有过程

-O：将文件解开到标准输出

下面的参数-f是必须的

-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名

```
root@ubuntu:/usr/local# tar -xvf mysql-5.7.34-linux-glibc2.12-x86_64.tar

root@ubuntu:/usr/local# tar -zxvf mysql-5.7.34-linux-glibc2.12-x86_64.tar.gz

root@ubuntu:/usr/local# mv mysql-5.7.34-linux-glibc2.12-x86_64 mysql5.7.34

root@ubuntu:/usr/local# cat /etc/group | grep mysql

root@ubuntu:/usr/local# groupadd mysql

root@ubuntu:/usr/local# useradd -r -g mysql mysql

root@ubuntu:/usr/local# mkdir mysql5.7.34/data

root@ubuntu:/usr/local# chown -R mysql.mysql /usr/local/mysql5.7.34/

root@ubuntu:/usr/local# cd mysql5.7.34/support-files/

root@ubuntu:/usr/local/mysql5.7.34/support-files# vim my_default.cnf

[mysqld]
\#设置mysql的安装目录

basedir =/usr/local/mysql5.7.34

\#设置mysql数据库的数据存放目录

datadir = /usr/local/mysql5.7.34/data

\#设置端口

port = 3306

 

socket = /tmp/mysql.sock

\#设置字符集

character-set-server=utf8

\#日志存放目录

log-error = /usr/local/mysql5.7.34/data/mysqld.log

pid-file = /usr/local/mysql5.7.34/data/mysqld.pid

\#允许时间类型的数据为零(去掉NO_ZERO_IN_DATE,NO_ZERO_DATE)

sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

\#ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```



```
root@ubuntu:/usr/local/mysql5.7.34/support-files# cp my_default.cnf /etc/my.cnf


apt-get install libaio1 libaio-dev


root@ubuntu:/usr/local/mysql5.7.34/bin# ./mysqld --initialize --user=mysql --basedir=/usr/local/mysql5.7.34 --datadir=/usr/local/mysql5.7.34/data

root@ubuntu:/usr/local/mysql5.7.34/bin# cd .. 

root@ubuntu:/usr/local/mysql5.7.34# cat data/mysqld.log

2021-11-12T09:10:13.774274Z 1 [Note] A temporary password is generated for root@localhost: X!qTxjbuU2l5   ###临时密码
 

root@ubuntu:/usr/local/mysql5.7.34# cp support-files/mysql.server /etc/init.d/mysql
```

 

启动mysql:

```
root@ubuntu:/usr/local/mysql5.7.34# ./support-files/mysql.server start
Starting MySQL
.. * 

root@ubuntu:/usr/local/mysql5.7.34# ./bin/mysql -u root -p
Enter password: 临时密码
mysql> set password=password('123456');
Query OK, 0 rows affected, 1 warning (0.00 sec)
mysql> grant all privileges on *.* to root@'%' identified by '123456';
Query OK, 0 rows affected, 1 warning (0.00 sec)
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
exit
```



```
root@ubuntu:/usr/local/mysql5.7.34# ./support-files/mysql.server restart
export PATH=$PATH:/usr/local/mysql5.7.34/bin
root@ubuntu:~# vim ~/.bashrc 
root@ubuntu:~# source ~/.bashrc 
root@ubuntu:~# mysql -u root -p
```



### **6.** 防火墙

```
root@ubuntu:~# apt install firewalld
root@ubuntu:~# firewall-cmd --zone=public --add-port=3306/tcp --permanent
success
root@ubuntu:~# ufw allow 3306
Rules updated
Rules updated (v6)
root@ubuntu:~# firewall-cmd --reload
success
root@ubuntu:~# firewall-cmd --list-ports
3306/tcp
```



### **7.** Docker

安装参考https://www.runoob.com/docker/ubuntu-docker-install.html

```
root@ubuntu:~# apt install curl
root@ubuntu:~# apt-get update
root@ubuntu:~# apt-get install \
  apt-transport-https \

  ca-certificates \

  curl \

  gnupg-agent \

software-properties-common
```

添加 Docker 的官方 GPG 密钥：

```
root@ubuntu:~# curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add –
```

9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88 通过搜索指纹的后8个字符，验证现在是否拥有带有指纹的密钥：

```
root@ubuntu:~# apt-key fingerprint 0EBFCD88
root@ubuntu:~# add-apt-repository \
  "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/ \
 $(lsb_release -cs) \
 stable
root@ubuntu:~# apt-get update
root@ubuntu:~# apt-get install docker-ce docker-ce-cli containerd.io
```

执行命令结果如下图则成功

```
root@ubuntu:~# docker run hello-world
```

![image-20211210164505646](https://gitee.com/barryx/kernel_study/raw/master/img/202112101645680.png)

 

#### docker镜像

```
root@ubuntu:~# docker pull grafana/Grafana

root@ubuntu:~# docker pull influxdb:1.8
```

![image-20211210164512348](https://gitee.com/barryx/kernel_study/raw/master/img/202112101645384.png) 

### **8.** BCC

安装参考https://blog.csdn.net/weixin_44395686/article/details/106712543

```
root@ubuntu:~# apt-get -y install bison build-essential cmake flex git libedit-dev \

 libllvm6.0 llvm-6.0-dev libclang-6.0-dev python zlib1g-dev libelf-dev

root@ubuntu:~# apt-get -y install python3-distutils python3-pip python3-setuptools
```

https://github.com/iovisor/bcc/releases

![](https://gitee.com/barryx/kernel_study/raw/master/img/202112101645751.png)

我放在了/usr/local下

依次执行以下

```
mkdir bcc/build

cd bcc/build

cmake ..

make

sudo make install

cmake -DPYTHON_CMD=python3 .. # build python3 binding

pushd src/python/

make

sudo make install

popd
```

 

## 二、LMP部署

### **1.** 拉取项目

```
#git clone https://github.com/linuxkerneltravel/lmp
```

总是有问题

可以用git方式：

```
#git clone git://github.com/linuxkerneltravel/lmp
```

![image-20211210165555598](https://gitee.com/barryx/kernel_study/raw/master/img/202112101655678.png)

上面是从gitee中拉下来的，下面是使用git方式拉下来的，都是94M

(110M是因为make之后查看的)

 

```
root@ubuntu:~# service mysql.server start

root@ubuntu:~# service mysql start

root@ubuntu:~# cd /lmp

root@ubuntu:/lmp# make db

mysql -u root -p <./misc/init.sql

Enter password: 123456

root@ubuntu:/lmp# make
```

报错

go build -mod=vendor -o lmp main.go

vendor/github.com/go-playground/validator/v10/baked_in.go:20:2: cannot find package "." in:

​	/lmp/vendor/golang.org/x/crypto/sha3

Makefile:12: recipe for target 'all' failed

make: *** [all] Error 1

![image-20211210165606107](https://gitee.com/barryx/kernel_study/raw/master/img/202112101656192.png)

```
root@ubuntu:/lmp# go mod tidy

root@ubuntu:/lmp# go mod vendor
```

之后make就可以通过了

### **2.** 运行grafana

```
\# docker run -d \ 

-p 3000:3000 \ 

--name=grafana \ 

grafana/grafana
```

![image-20211210165617125](https://gitee.com/barryx/kernel_study/raw/master/img/202112101656211.png)

### **3.** 运行influxdb

```
\# docker run -d \

-p 8083:8083 \

-p 8086:8086 \

--name influxdb \

-v /lmp/test/influxdb_config/default.conf:/etc/influxdb/influxdb.conf \

-v /lmp/test/influxdb_config/data:/var/lib/influxdb/data \

-v /lmp/test/influxdb_config/meta:/var/lib/influxdb/meta \

-v /lmp/test/influxdb_config/wal:/var/lib/influxdb/wal influxdb:1.8
```

![image-20211210165641124](https://gitee.com/barryx/kernel_study/raw/master/img/202112101656220.png)

 

**查看是否正常启动：**

root@ubuntu:/lmp# docker ps -a

![image-20211210165648714](https://gitee.com/barryx/kernel_study/raw/master/img/202112101656796.png)

 

### **4.** 运行LMP

```
root@ubuntu:/lmp# make

go build -mod=vendor -o lmp main.go

root@ubuntu:/lmp# ./lmp 
```

![image-20211210165704628](https://gitee.com/barryx/kernel_study/raw/master/img/202112101657716.png)

### **5.** 防火墙

```
root@ubuntu:~# firewall-cmd --zone=public --add-port=8080/tcp --permanent

success

root@ubuntu:~# ufw allow 8080

Rules updated

Rules updated (v6)

root@ubuntu:~# firewall-cmd --reload

success

root@ubuntu:~# firewall-cmd --list-ports

3306/tcp 8080/tcp
```



### **6.** 数据库配置

这里的配置必须在虚拟机中配：

127.0.0.1:3000 用户:admin 密码:admin

-> skip-> DATA SOURCES-> 数据库influxdb

![](https://gitee.com/barryx/kernel_study/raw/master/img/202112101657863.png)

 

### **7.** 导入json并观测

/lmp/test/grafana-JSON下的lmp.json文件：

![image-20211210165724383](https://gitee.com/barryx/kernel_study/raw/master/img/202112101657472.png)

![image-20211210165732011](https://gitee.com/barryx/kernel_study/raw/master/img/202112101657094.png)

![image-20211210165744712](https://gitee.com/barryx/kernel_study/raw/master/img/202112101657801.png)

完成！！！

## 三、指标不显示

![image-20211210165756267](https://gitee.com/barryx/kernel_study/raw/master/img/202112101657355.png)

报错：ModuleNotFoundError: No module named 'influxdb'

使用：

```
root@ubuntu:/lmp# pip3 install influxdb
```

 

重新submit

报错：AttributeError: module 'yaml' has no attribute 'FullLoader'

原因：已存在pyyaml，且版本低于5.1

![image-20211210165819604](https://gitee.com/barryx/kernel_study/raw/master/img/202112101658693.png)

暴力忽视错误升级：

```
\# pip3 install --ignore-installed PyYAML
```

![](https://gitee.com/barryx/kernel_study/raw/master/img/202112101658289.png)

![image-20211210165833166](../../../../../../../../../../../../Application%20Support/typora-user-images/image-20211210165833166.png)

 

报错：ModuleNotFoundError: No module named 'elasticsearch'

```
root@ubuntu:/lmp# pip3 install elasticsearch
```

 

报错：ModuleNotFoundError: No module named 'db_writer_utils'

```
root@ubuntu:/lmp/plugins/db_writer# vim bufferImpl.py
```

![image-20211210165841378](https://gitee.com/barryx/kernel_study/raw/master/img/202112101658464.png)

 

报错：FileNotFoundError: [Errno 2] No such file or directory: '/lmp/log/wlog.log'

直接touch了一个，不知道会不会有问题

 

报错：ModuleNotFoundError: No module named 'settings'

打开writerImpl.py

const那句去掉settings：

from const import DatabaseType

添加

import sys

sys.path.append('/lmp/plugins/db_writer')

 

成功！！！

![image-20211210165856361](https://gitee.com/barryx/kernel_study/raw/master/img/202112101658429.png)