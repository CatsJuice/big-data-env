# big-data-env
大数据环境搭建

以 3 个节点为例， 一个主节点 `master`, 两个从节点 `slave1` 和 `slave2`

## 1. 基础环境搭建

### 1.1 修改主机名（hostname）

**(1) 首次切换到 root 用户**

```bash
su 
```

**(2) 修改主机名**

语法：`hostnamectl set hostname <主机名>`

```bash
hostnamectl set hostname master
```

**(3) 永久修改主机名**

编辑 `/etc/sysconfig/network`:

```bash
vi /etc/sysconfig/network
```

内容如下:

```conf
NETWORKING=yes
HOSTNAME=master
```

**(4) 重启计算机**

```bash
reboot
```

**(5) 查看是否生效**

```bash
hostname
```

### 1.2 配置 hosts 文件

**(1) 查看ip**

```bash
ifconfig
```

如果命令不存在，则使用 

```bash
ip addr
```

**配置hosts文件**

在每一台主机上都做如下修改（或者在master上修改并分发给其余主机）

```bash
vi /etc/hosts
```

增加内容如下(`ip` + `对应主机名`):

```bash
192.168.9.131 master
192.168.9.133 slave1
192.168.9.132 slave2
```

### 1.3 关闭防火墙

Centos 7下：

关闭防火墙：

```bash
systemctl stop firewalld
```

查看防火墙状态

```bash
systemctl status firewalld
```

### 1.4 时间同步

**(1) 查看自己机器的时间**

```bash
date
```

**(2) 选择时区**

```bash
tzselect
```

依次选择：

- 亚洲： `5`
- 中国： `9`
- 北京时间： `1`
- 确认： `1`

**(3) 下载ntp**

在每一台主机上安装： 

```bash
yum install -y ntp
```

**(4) master作为ntp服务器， 修改ntp配置文件**

在 master 上执行：

```bash
vi /etc/ntp.conf
```

在文件尾部追加以下内容：

```bash
server 127.127.1.0
fudge 127.127.1.0 stratum 10
```

**(5) 重启ntp服务**

```bash
/bin/systemctl restart ntpd.service
```

**(6) 其他机器同步**

（等待大概五分钟）

```bash
ntpdate master
```

### 1.5 配置 SSH 免密登录

**(1) 每个节点分别产生公私密钥**

```bash
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
```

密钥产生在用户主目录下的.ssh目录中(`~/.ssh`)

**(2) 在master上将公钥文件（id_dsa.pub）复制成 authorized_keys**

仅在 `master` 上，在 `.ssh` 目录下执行

```bash
cat id_dsa.pub >> authorized_keys
```

**(3) 在master上连接自己**

(这一步可以跳过)

```bash
ssh master
```

**(4) 实现master免密登录子节点**

在 `slave1` 和 `slave2` 中分别进行如下操作：

切换到 `.ssh` 目录(路径: `~/.ssh`)

复制 `master` 的公钥文件 并重命名为 `master_dsa.pub`(过程需要输入master密码验证)

```bash
scp master:~/.ssh/id_dsa.pub ./master_dsa.pub
```

将 `master` 结点的公钥文件追加到 `authorized_keys` 文件:

```bash
cat master_dsa.pub >> authorized_keys
```

**(5) 在master上验证登录**

(非必要步骤)

连接 slave1:

```bash
ssh slave1
```

连接 slave2:

```bash
ssh slave2
```

### 1.6 JDK安装

**(1) 建立工作路径**

```bash
mkdir /usr/java
```

**(2) 解压安装文件**

这里我的安装包路径为(仅在master上)：

```bash
/usr/pkg/jdk-8u171-linux-x64.tar.gz
```

在 master 上（任意位置）执行解压命令：

```bash
tar -zxvf /usr/pkg/jdk-8u171-linux-x64.tar.gz -C /usr/java/
```

进入 `/usr/java`, 使用 `ls` 查看， 进入刚解压出来的 `jdk` 目录， 使用 `pwd` 查看当前路径如下：

```bash
[root@master java]# ls
jdk1.8.0_171
[root@master java]# cd jdk1.8.0_171/
[root@master jdk1.8.0_171]# pwd
/usr/java/jdk1.8.0_171
```

**(3) 修改环境变量**

在 master 上 编辑 `/etc/profile`

```bash
vi /etc/profile
```

添加内容如下：

```bash
export JAVA_HOME=/usr/java/jdk1.8.0_171
export CLASSPATH=$JAVA_HOME/lib/
export PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
```

生效环境变量：

```bash
source /etc/profile
```

验证安装：

```bash
[root@master jdk1.8.0_171]# java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)

```

**(4) 在其余主机上安装**

在 `master` 上， 将 `/usr/java` 复制到其余主机:

```bash
scp -r /usr/java slave1:/usr/
scp -r /usr/java slave2:/usr/
```

在 `slave1` 和 `slave2` 上配置环境变量,

这里可以直接将 `master` 上配置好的文件直接分发给子节点：

```bash
scp /etc/profile slave1:/etc/profile
scp /etc/profile slave2:/etc/profile
```

分发完毕后， 需要在子节点上执行

```bash
source /etc/profile
```

使配置生效

同上， 在子节点上使用 `java -version` 查看是否成功

## 2. Zookeeper

### 2.1 修改主机名称到IP地址映射配置


**(1) 编辑hosts**

在 `master` 主机上

```bash
vi /etc/hosts
```

**(2) 修改内容如下**

```bash
192.168.9.131 master
192.168.9.133 slave1
192.168.9.132 slave2
```

改为

```bash
192.168.9.131 master master.root
192.168.9.133 slave1 slave1.root
192.168.9.132 slave2 slave2.root
```

**(3) 修改其他节点的hosts**

在局域网环境下， 其他节点的hosts配置文件相同， 方便起见可直接分发：

```bash
scp /etc/hosts slave1:/etc/hosts
scp /etc/hosts slave2:/etc/hosts
```

### 2.2 安装 zookeeper

**(1) 建立工作文件夹**

首先在 master 上进行

```bash
mkdir /usr/zookeeper
```

**(2) 解压缩**

这里我的安装包位于 `/usr/pkg/zookeeper-3.4.10.tar.gz`

执行解压命令， 解压至刚新建的 `zookeeper` 文件夹

```bash
tar -zxvf /usr/pkg/zookeeper-3.4.10.tar.gz -C /usr/zookeeper
```

**(3) 配置zoo.cfg**

进入 `zookeeper` 配置文件夹 `conf`:

```bash
cd /usr/zookeeper/zookeeper-3.4.10/conf/
```

将 `zoo_sample.cfg` 复制一份命名为 `zoo.cfg`:

```bash
scp zoo_sample.cfg zoo.cfg
```

编辑 `zoo.cfg` ( 需要在 `/usr/zookeeper/zookeeper-3.4.10/conf/` 目录下)

```bash
vi zoo.cfg
```

主要 修改 / 添加 的内容如下：

```bash
dataDir=/usr/zookeeper/zookeeper-3.4.10/zkdata
clientPort=2181
dataLogDir=/usr/zookeeper/zookeeper-3.4.10/zkdatalog

server.1=master:2888:3888
server.2=slave1:2888:3888 server.3=slave2:2888:3888 
```

`server.A=B：C：D`

**A**：是一个数字(机器重启默认从`0`开始)，表示这个是第几号服务器； 

**B**：是这个服务器的ip地址，`zookeeper`是在`hosts`中映射了本机的`IP`，因此也可以写为服务器的映射名；

**C**：表示的是这个服务器与集群中的`Leader`服务器交换信息的端口； 

**D**：表示的是万一集群中的`Leader`服务器挂了，需要一个端口来重新进行选举，选出一个新的`Leader`， 而这个端口就是用来执行选举时服务器相互通信的端口。 `2888`端口号是服务之间通信的端口，而`3888`是 `zookeeper` 与其他应用程序通信的端口

**(4) 创建配置文件中设置的两个文件夹**

在 `zookeeper` 安装根目录下(`/usr/zookeeper/zookeeper-3.4.10/`)

```bash
mkdir zkdata
mkdir zkdatalog
```

文件夹名称需要与配置文件中对应

**(5) 修改 myid**

进入上一步创建的 `zkdata` 文件夹, 执行

```bash
vi myid
```

内容为数字 `1` ( 对应配置文件中的配置 )

```bash
[root@master zkdata]# vi myid
[root@master zkdata]# cat myid
1
[root@master zkdata]# 
```

**(6) 将 zookeeper 分发到子节点**

```bash
scp -r /usr/zookeeper root@slave1:/usr/
scp -r /usr/zookeeper root@slave2:/usr/
```

**(7) 修改子节点的myid**

修改 `zookeeper` 根目录下的 `zkdata` 文件夹下的 `myid`, 内容为一个数字， 数值对应 `zoo.cfg` 配置文件中的设置

```bash
cd /usr/zookeeper/zookeeper-3.4.10/zkdata
vi myid
```

**(8) 配置环境变量**

在 `master` 上:

```bash
vi /etc/profile
```

增加配置如下：

```bash
export ZOOKEEPER_HOME=/usr/zookeeper/zookeeper-3.4.10  
PATH=$PATH:$ZOOKEEPER_HOME/bin
```

保存后， `source /etc/profile` 使生效;

可以在 `slave1` 和 `slave2` 中执行相同的操作， 也可以在 `master` 直接分发到子节点:

```bash
scp /etc/profile slave1:/etc/profile
scp /etc/profile slave2:/etc/profile
```

分发完毕后， 需要在 子节点主机上 执行 `source /etc/profile` 时其生效

**(9) 启动zookeeper集群**

在所有主机上执行如下操作：

切换到 `zookeeper` 根目录：

```bash
cd /usr/zookeeper/zookeeper-3.4.10/
```

启动集群：

```bash
bin/zkServer.sh start
```

所有主机启动后， 分别查看状态：

```bash
bin/zkServer.sh status
```

成功如下：

```bash

[root@master zookeeper-3.4.10]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/zookeeper/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: follower
[root@master zookeeper-3.4.10]# 


[root@slave1 zookeeper-3.4.10]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/zookeeper/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: leader
[root@slave1 zookeeper-3.4.10]# 


[root@slave2 zookeeper-3.4.10]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/zookeeper/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: follower
[root@slave2 zookeeper-3.4.10]# 
```