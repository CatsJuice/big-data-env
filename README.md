# big-data-env
大数据环境搭建


  - [1. 基础环境搭建](#1-%e5%9f%ba%e7%a1%80%e7%8e%af%e5%a2%83%e6%90%ad%e5%bb%ba)
    - [1.1 修改主机名（hostname）](#11-%e4%bf%ae%e6%94%b9%e4%b8%bb%e6%9c%ba%e5%90%8dhostname)
    - [1.2 配置 hosts 文件](#12-%e9%85%8d%e7%bd%ae-hosts-%e6%96%87%e4%bb%b6)
    - [1.3 关闭防火墙](#13-%e5%85%b3%e9%97%ad%e9%98%b2%e7%81%ab%e5%a2%99)
    - [1.4 时间同步](#14-%e6%97%b6%e9%97%b4%e5%90%8c%e6%ad%a5)
    - [1.5 配置 SSH 免密登录](#15-%e9%85%8d%e7%bd%ae-ssh-%e5%85%8d%e5%af%86%e7%99%bb%e5%bd%95)
    - [1.6 JDK安装](#16-jdk%e5%ae%89%e8%a3%85)
  - [2. Zookeeper](#2-zookeeper)
    - [2.1 修改主机名称到IP地址映射配置](#21-%e4%bf%ae%e6%94%b9%e4%b8%bb%e6%9c%ba%e5%90%8d%e7%a7%b0%e5%88%b0ip%e5%9c%b0%e5%9d%80%e6%98%a0%e5%b0%84%e9%85%8d%e7%bd%ae)
    - [2.2 安装 zookeeper](#22-%e5%ae%89%e8%a3%85-zookeeper)
  - [3. Hadoop](#3-hadoop)
    - [3.1 配置环境变量](#31-%e9%85%8d%e7%bd%ae%e7%8e%af%e5%a2%83%e5%8f%98%e9%87%8f)
    - [3.2 配置 Hadoop 的各个组件](#32-%e9%85%8d%e7%bd%ae-hadoop-%e7%9a%84%e5%90%84%e4%b8%aa%e7%bb%84%e4%bb%b6)
      - [3.2.1 hadoop-env.sh](#321-hadoop-envsh)
      - [3.2.2 core-site.xml](#322-core-sitexml)
      - [3.2.3 yarn-site.xml](#323-yarn-sitexml)
      - [3.2.4 hdfs-site.xml](#324-hdfs-sitexml)
      - [3.2.5 mapred-site.xml](#325-mapred-sitexml)
    - [3.3 设置节点文件](#33-%e8%ae%be%e7%bd%ae%e8%8a%82%e7%82%b9%e6%96%87%e4%bb%b6)
    - [3.4 分发 Hadoop](#34-%e5%88%86%e5%8f%91-hadoop)
    - [3.5 格式化 HDFS](#35-%e6%a0%bc%e5%bc%8f%e5%8c%96-hdfs)
    - [3.6 开启集群](#36-%e5%bc%80%e5%90%af%e9%9b%86%e7%be%a4)
    - [3.7 访问集群 Web UI](#37-%e8%ae%bf%e9%97%ae%e9%9b%86%e7%be%a4-web-ui)
    - [3.8 Hadoop 脚本使用](#38-hadoop-%e8%84%9a%e6%9c%ac%e4%bd%bf%e7%94%a8)
  - [4. Spark On YARN](#4-spark-on-yarn)
    - [4.1 Scala 安装](#41-scala-%e5%ae%89%e8%a3%85)
    - [4.2 Spark 安装](#42-spark-%e5%ae%89%e8%a3%85)
      - [4.2.1 安装](#421-%e5%ae%89%e8%a3%85)
      - [4.2.2 配置spark-env.sh文件](#422-%e9%85%8d%e7%bd%aespark-envsh%e6%96%87%e4%bb%b6)
      - [4.2.3 配置spark从节点，修改slaves文件](#423-%e9%85%8d%e7%bd%aespark%e4%bb%8e%e8%8a%82%e7%82%b9%e4%bf%ae%e6%94%b9slaves%e6%96%87%e4%bb%b6)
      - [4.2.4 配置spark环境变量](#424-%e9%85%8d%e7%bd%aespark%e7%8e%af%e5%a2%83%e5%8f%98%e9%87%8f)
      - [4.2.5 发送配置好的spark安装包到子节点](#425-%e5%8f%91%e9%80%81%e9%85%8d%e7%bd%ae%e5%a5%bd%e7%9a%84spark%e5%ae%89%e8%a3%85%e5%8c%85%e5%88%b0%e5%ad%90%e8%8a%82%e7%82%b9)
      - [4.2.6 测试 SPARK 环境](#426-%e6%b5%8b%e8%af%95-spark-%e7%8e%af%e5%a2%83)
      - [4.2.7 访问SparkWeb界面](#427-%e8%ae%bf%e9%97%aesparkweb%e7%95%8c%e9%9d%a2)
      - [4.2.8 开启spark-shell](#428-%e5%bc%80%e5%90%afspark-shell)




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

## 3. Hadoop

### 3.1 配置环境变量

首先 在 `master` 上进行操作

**(1) 创建工作目录**

```bash
mkdir /usr/hadoop
```

**(2) 解压 Hadoop**

这里我的安装包位于：

```bash
/usr/pkg/hadoop-2.7.3.tar.gz
```

执行解压命令

```bash
tar -zxvf /usr/pkg/hadoop-2.7.3.tar.gz -C /usr/hadoop
```

**(3) 修改环境变量**

```bash
vi /etc/profile
```

增加如下环境变量

```bash
export HADOOP_HOME=/usr/hadoop/hadoop-2.7.3 export CLASSPATH=$CLASSPATH:$HADOOP_HOME/lib
export PATH=$PATH:$HADOOP_HOME/bin 
```

然后生效配置文件

```bash
source /etc/profile
```

### 3.2 配置 Hadoop 的各个组件

 `hadoop` 的各个组件的都是使用XML进行配置，这些文件存放在 `hadoop` 的 `etc/hadoop` 目录下。

 | 组件名        | 文件名          |
 | :------------ | :-------------- |
 | Common组件    | core-site.xml   |
 | HDFS组件      | hdfs-site.xml   |
 | MapReduce组件 | mapred-site.xml |
 | YARN组件      | yarn-site.xml   |

#### 3.2.1 hadoop-env.sh

修改 java 环境变量

```bash
export JAVA_HOME=/usr/java/jdk1.8.0_171
```

#### 3.2.2 core-site.xml

在 `<configuration></configuration>`加入代码：

```xml
<configuration>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://master:9000</value>
    </property>

    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/hadoop/hadoop-2.7.3/hdfs/tmp</value>
        <description>A base for other temporary directories.</description>
    </property>

    <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
    </property>

    <property>
        <name>fs.checkpoint.period</name>
        <value>60</value>
    </property>

    <property>
        <name>fs.checkpoint.size</name>
        <value>67108864</value>
    </property>
</configuration>
```

完整代码：[core-site.xml](https://github.com/CatsJuice/big-data-env/blob/master/hadoop/core-site.xml)

#### 3.2.3 yarn-site.xml

```xml
<configuration>
    <property>
        <name>yarn.resourcemanager.adress</name>
        <value>master:18040</value>
    </property>

    <property>
        <name>yarn.resourcemanager.scheduler.adress</name>
        <value>master:18030</value>
    </property>

    <property>
        <name>yarn.resourcemanager.webapp.adress</name>
        <value>master:18088</value>
    </property>

    <property>
        <name>yarn.resourcemanager.resource-tracker.adress</name>
        <value>master:18025</value>
    </property>

    <property>
        <name>yarn.resourcemanager.admin.adress</name>
        <value>master:18141</value>
    </property>

    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <property>
        <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>

    <!-- Site specific YARN configuration properties -->

</configuration>
```

完整代码： [yarn-site.xml](https://github.com/CatsJuice/big-data-env/blob/master/hadoop/yarn-site.xml)

#### 3.2.4 hdfs-site.xml

```xml
<configuration>

    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>

    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/hadoop/hadoop-2.7.3/hdfs/name</value>
        <final>true</final>
    </property>

    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/hadoop/hadoop-2.7.3/hdfs/data</value>
        <final>true</final>
    </property>

    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>master:9001</value>
    </property>

    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>

    <property>
        <name>dfs.permissions</name>
        <value>false</value>
    </property>

</configuration>
```

完整代码： [hdfs-site.xml](https://github.com/CatsJuice/big-data-env/blob/master/hadoop/hdfs-site.xml)

#### 3.2.5 mapred-site.xml

`hadoop` 是没有这个文件的，需要将 `mapred-site.xml.template` 复制为 `mapredsite.xml`

```bash
cp mapred-site.xml.template mapred-site.xml 
```
修改如下：

```xml
<configuration>

<property>
    <!-- 指定Mapreduce运行在yarn上-->
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property> 

</configuration>
```

### 3.3 设置节点文件

编写 `slaves` 文件,添加子节点 `slave1` 和 `slave2` 

```bash
vi slaves
```

设置内容如下：

```bash
slave1
slave2
```

编写 `master` 文件，添加主节点 `master`

```bash
vi master
```

设置内容如下：

```bash
master
```

### 3.4 分发 Hadoop

```bash
scp -r /usr/hadoop root@slave1:/usr/ 
scp -r /usr/hadoop root@slave2:/usr/
```

### 3.5 格式化 HDFS

```bash
hadoop namenode -format 
```

当出现 `Exiting with status 0` 的时候，表明格式化成功。 

### 3.6 开启集群

仅在 `master` 主机上开启操作命令。它会带起从节点的启动;
在 `Hadoop` 根目录下( `/usr/hadoop/hadoop-2.7.3/` )：

```bash
sbin/start-all.sh 
```

查看进程：

```bash
[root@master hadoop-2.7.3]# jps
21888 SecondaryNameNode
21703 NameNode
22039 ResourceManager
22298 Jps
21150 QuorumPeerMain
[root@master hadoop-2.7.3]# 
```

子节点上查看：

```bash
[root@slave2 zookeeper-3.4.10]# jps
21680 NodeManager
21179 QuorumPeerMain
21580 DataNode
21788 Jps
[root@slave2 zookeeper-3.4.10]# 


[root@slave1 zookeeper-3.4.10]# jps
2992 DataNode
3092 NodeManager
3194 Jps
2541 QuorumPeerMain
[root@slave1 zookeeper-3.4.10]# 
```

### 3.7 访问集群 Web UI

使用 浏览器（同一网段）访问 `master:50070`, 如果无法访问， 首先排查防火墙是否关闭

### 3.8 Hadoop 脚本使用

注： 按以上步骤， 在子节点中未配置 `Hadoop` 的环境变量， 所以在子节点中无法直接使用 `hadoop` 命令， 可以在 `master` 上将 `/etc/profile` 分发到子节点并在子节点上 生效配置文件

```bash
[root@master hadoop-2.7.3]# scp /etc/profile slave1:/etc/profile
profile                           100% 2200     2.0MB/s   00:00    
```

```bash
[root@slave1 zookeeper-3.4.10]# source /etc/profile
```

**查看dfs根目录文件**

```bash
hadoop fs -ls /
```

**在 hdfs 上创建文件**

```bash
hadoop fs -mkdir /data
```

**再次查看根目录**

```bash
[root@master hadoop-2.7.3]# hadoop fs -ls /
Found 1 items
drwxr-xr-x   - root supergroup          0 2019-09-22 22:53 /data
[root@master hadoop-2.7.3]#
```

**Web UI 中查看创建的文件**

进入顶部 TAB 栏中的 `Utilities` -> `Browse the file system`

![截图](https://catsjuice.cn/mkdown_imgs/20190922225550.jpg)

## 4. Spark On YARN

### 4.1 Scala 安装

我们需要在拥有 `hadoop` 集群的所有节点中安装 `scala` 语言环境，因为 `spark` 的 源代码为 `scala` 语言所编写，所以接下来我们进行安装 `scala`

**解压 scala 的 tar 包**

这里我的安装包位于：

```bash
/usr/pkg/scala-2.11.12.tgz
```

创建Scala目录

```bash
mkdir /usr/scala
```

进行解压：

```bash
tar -zxvf /usr/pkg/scala-2.11.12.tgz -C /usr/scala
```

**配置环境变量**

进入 `scala` 安装目录， 确认 `scala` 的安装路径:

```bash
[root@master pkg]# cd /usr/scala/scala-2.11.12/
[root@master scala-2.11.12]# pwd
/usr/scala/scala-2.11.12
[root@master scala-2.11.12]# 
```

编辑 `/etc/profile`

```bash
vi /etc/profile
```

增加 环境变量：

```bash
export SCALA_HOME=/usr/scala/scala-2.11.12
export PATH=$SCALA_HOME/bin:$PATH
```

然后生效配置文件：

```bash
source /etc/profile
```

查看 `scala` 版本号:

```bash
scala -version
```

**分发Scala到子节点**

```bash
scp -r /usr/scala root@slave1:/usr/
scp -r /usr/scala root@slave2:/usr/
```

分发完毕后， 配置子节点的环境变量并生效；
这里直接分发：

```bash
scp -r /etc/profile slave1:/etc/profile
scp -r /etc/profile slave2:/etc/profile
```

并分别在 子结点上生效配置文件：

```bash
source /etc/profile
```

### 4.2 Spark 安装

#### 4.2.1 安装

这里我的安装包位于：

```bash
/usr/pkg/spark-2.4.0-bin-hadoop2.7.tgz
```

先创建 `/usr/spark` 目录：

```bash
mkdir /usr/spark
```

解压安装包 到 `/usr/spark`

```bash
tar -zxvf /usr/pkg/spark-2.4.0-bin-hadoop2.7.tgz -C /usr/spark
```

#### 4.2.2 配置spark-env.sh文件

进入到 `spark` 配置文件目录（`conf`）:

```bash
[root@master pkg]# cd /usr/spark/spark-2.4.0-bin-hadoop2.7/conf/
```

将 `spark-env.sh.template` 复制为 `spark-env.sh`:

```bash
scp spark-env.sh.template spark-env.sh
```

编辑 `spark-env.sh` 添加内容如下：

```bash
#!/usr/bin/env bash
export SPARK_MASTER_IP=master 
export SCALA_HOME=/usr/scala/scala-2.11.12 
export SPARK_WORKER_MEMORY=8g 
export JAVA_HOME=/usr/java/jdk1.8.0_171 
export HADOOP_HOME=/usr/hadoop/hadoop-2.7.3 
export HADOOP_CONF_DIR=/usr/hadoop/hadoop-2.7.3/etc/hadoop
```

#### 4.2.3 配置spark从节点，修改slaves文件

依旧在 `spark` 的配置文件目录（`conf`）,

复制 `slaves.template` 重命名为 `slaves`

```bash
cp slaves.template slaves
```

编辑 slaves 文件:

```bash
vi slaves
```

在 `slaves` 中添加 `spark` 的工作节点 ：

```bash
slave1
slave2
```

#### 4.2.4 配置spark环境变量 

```bash
vi /etc/profile
```

添加内容如下：

```bash
export SPARK_HOME=/usr/spark/spark-2.4.0-bin-hadoop2.7 
export PATH=$SPARK_HOME/bin:$PATH 
```

使环境变量生效

```bash
source /etc/profile
```

#### 4.2.5 发送配置好的spark安装包到子节点 

```bash
scp -r /usr/spark root@slave1:/usr/
scp -r /usr/spark root@slave2:/usr/
```

修改子节点的环境变量，同上， 直接分发：

```bash
scp /etc/profile slave1:/etc/profile
scp /etc/profile slave2:/etc/profile
```

并在子节点中运行

```bash
source /etc/profile
```

来生效配置

#### 4.2.6 测试 SPARK 环境

首先确保 `hadoop` 环境已开启， 如果未开启，则在 `master` 上使用如下命令开启：

```bash
/usr/hadoop/hadoop-2.7.3/sbin/start-all.sh
```

开启 `Spark` 集群

```bash
/usr/spark/spark-2.4.0-bin-hadoop2.7/sbin/start-all.sh
```

#### 4.2.7 访问SparkWeb界面 

在浏览器中输入 `master` 节点的 IP 地址， 端口为 `8080`

![screenshot](http://catsjuice.cn/mkdown_imgs/20190922235004.jpg)


#### 4.2.8 开启spark-shell

```
[root@master sbin]# spark-shell
2019-09-22 23:52:51 WARN  NativeCodeLoader:62 - Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://master:4040
Spark context available as 'sc' (master = local[*], app id = local-1569167578016).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.0
      /_/
         
Using Scala version 2.11.12 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_171)
Type in expressions to have them evaluated.
Type :help for more information.

```

`spark-shell` 默认进入的是 `scala` 环境的 `spark` 交互模式

要进入 `python` 环境下的 `spark` 交互模式， 使用：

```
[root@master sbin]# pyspark
Python 2.7.5 (default, Jun 20 2019, 20:27:34) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-36)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
2019-09-22 23:55:59 WARN  NativeCodeLoader:62 - Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.4.0
      /_/

Using Python version 2.7.5 (default, Jun 20 2019 20:27:34)
SparkSession available as 'spark'.
>>> 
```