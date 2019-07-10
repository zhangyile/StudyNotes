# Linux

## basic concept

###  孤儿进程 & 僵尸进程

**孤儿进程**：一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。

**僵尸进程**：一个进程使用fork创建子进程，如果子进程退出，而父进程并没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵死进程。

## SSL/TLS

| NAME | DESC |
| --- | --- |
| TLS | Transport Layer Security |
| SSL | Secure Socket Layer |
| CSR | Certificate Signing Request (证书签名请求) |
| CRT | certificate (证书) |
| X.509 | 一种证书格式 |
| (X.509)PEM | Privacy Enhanced Mail,打开看文本格式,以"-----BEGIN..."开头, "-----END..."结尾,内容是BASE64编码. |
| (X.509)DER | Distinguished Encoding Rules,打开看是二进制格式,不可读. |

> [参考文档](https://www.cnblogs.com/yjmyzz/p/openssl-tutorial.html)

openssl 给自己颁发证书

```bash
openssl genrsa -des3 -out server.key 2048           # 生成 server 的私钥
openssl rsa -in server.key -out server.key          # 加密私钥文件
openssl req -new -key server.key -out server.csr    # 生成服务端证书签名请求文件

openssl genrsa -des3 -out ca.key 1024               # 生成 CA key 文件
openssl req -new -x509 -key ca.key -out ca.crt      # 生成 CA 自签名证书 [-days 365]

openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key  -CAcreateserial -out server.crt  # 使用 CA 证书给 server.csr 颁发一个证书

openssl x509 -in server.crt -noout -text           # 查看证书内容
openssl req -in server.csr -noout -text # 查看证书签名请求文件的内容 
```


## Iptables

四表： Filter、NAT、[ Mangle、Raw ]
五链： Prerouting、Forward、Input、Output、Postrouting
**链表关系**
PREROUTING       的规则可存在于：nat表、mangle表、raw表 
INPUT          的规则可存在于：mangle表、filter表（nat表centos6没有，centos7有） 
FORWARD       的规则可存在于：mangle表、filter表 
OUTPUT          的规则可存在于：raw表、mangle表、nat表、filter表 
POSTROUTING     的规则可存在于：mangle表、nat表 
**表链关系**
nat         ：PREROUTING、OUTPUT、POSTROUTING （INPUT链 centos7有，centos6没有） 
filter      ：INPUT、FORWARD、OUTPUT 
raw         ：PREROUTING、OUTPUT 
mangle      ：PREROUTING、INPUT、FORWARD、OUTPUT、POSTROUTING 

![](media/15111691153372/15545377688882.jpg)



```bash
iptables FORWARD ACCEPT  设置 Filter 表的默认 forward 链为 accept 
```

## topic 

### 切分文件

* sed
  
  `sed -n '2,4p' test.txt`
  
* cut `cut -d: -f 2 test.txt`
 主要参数
-b ：以字节为单位进行分割。这些字节位置将忽略多字节字符边界，除非也指定了 -n 标志。
-c ：以字符为单位进行分割。
-d ：自定义分隔符，默认为制表符。
-f ：与-d一起使用，指定显示哪个区域。
-n ：取消分割多字节字符。仅和 -b 标志一起使用。如果字符的最后一个字节落在由 -b 标志的 List 参数指示的<br />范围之内，该字符将被写出；否则，该字符将被排除。


### generator random string

`openssl rand -base64 16`

-------


## linux 常用命令

### vim

[常用技巧](https://mp.weixin.qq.com/s/aQZa55a-dr7Zd2a6RRPZMw)

### tcpdump

获取指定ip port 的通信包
`tcpdump -i eth0 tcp port 80 and host 88.198.175.2`

### iptables & ip  

**基本用法**
查看当前防火墙规则： `iptables-save`
查看外网出口 IP： `curl https://ipinfo.io/ip`


**ipset**
查看当前ipset 列表 `ipset list`

**ip route**

```bash
//显示默认路由表
# ip route show 
//显示指定表的路由表
# ip route show table main

//测试目标ip地址从哪个网卡（路由）发出
# ip route get 1.1.1.1
[root@vip126mx1 coremail]# ip r get 10.144.8.16
10.144.8.16 via 10.120.102.253 dev eth0  src 10.120.102.211
    cache  mtu 1500 hoplimit 64

//添加到主机的路由
# route add –host 192.168.168.110 dev eth0
# route add –host 192.168.168.119 gw 192.168.168.1

//添加到网络的路由
# route add –net IP netmask MASK eth0
# route add –net IP netmask MASK gw IP
# route add –net IP/24 eth1

//添加默认网关
# route add default gw IP

//删除路由
# route del –host 192.168.168.110 dev eth0

//若要显示 IP 路由表的全部内容，请键入：
# route print
```

**ip rule**

```bash
# 显示路由规则
ip rule list/show  
0:	from all lookup local
32766:	from all lookup main
32767:	from all lookup default

# 添加路由规则
ip rule add from 192.168.1.10/32 table 1 pref 100
# 如果pref值不指定，则将在已有规则最小序号前插入

# 刷新路由缓冲
ip rule flush cache
```

开机启动自动加载route table

```bash
# vim /etc/iproute2/rt_tables
#
# reserved values
#
255     local
254     main
253     default
200     HW
201     DX
202     LT
0       unspec
#
# local
#
#1      inr.ruhep
```

```bash
# vim /etc/rc.local
...
/sbin/ip route add default via 10.230.250.129 dev eth1 table HW
/sbin/ip route add default via 14.29.82.1 dev eth2 table DX
/sbin/ip route add default via 122.13.158.1 dev eth3 table LT

/sbin/ip rule add from 10.230.250.139 table HW
/sbin/ip rule add from 14.29.82.19 table DX
/sbin/ip rule add from 122.13.158.37 table LT

# default route uses HW
/sbin/ip route add default via 10.230.250.129
...
```

重启后查看是否加载成功

```bash
[coremail@hw-sz-vip126smtp1 ~]$ ip rule show
0:	from all lookup local
32763:	from 122.13.158.37 lookup LT
32764:	from 14.29.82.19 lookup DX
32765:	from 10.230.250.139 lookup HW
32766:	from all lookup main
32767:	from all lookup default
```


### ping

指定网卡
`ping -I eth0 8.8.8.8`


### curl & httpie


```
curl -X POST -H "Content-Type: application/json" --user admin:adminadmin http://localhost:12122/pe/test/api/snest/report_node_info/ --data '{"srv_type": "webapp_tomcat", "id": 119, "data": {"startup_time_second": 76}}'

```


```
curl -i http://localhost:12122/pe/test/api/snest/report_node_info/ --user admin:adminadmin -X POST -H "Content-Type: application/json" --data '{"srv_type": "webapp_tomcat", "id": 119, "data": {"startup_time_second": 76}}'

```

### xargs



-------


## Git

### git 常见用法

#### git rewriting History
> https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History

1. modify your last commit message
>`git commit --amend` 
    
2. Merge commit message 
> `git rebase -i HEAD~3`

    ```git
    pick f7f3f6d changed my name a bit
    pick 310154e updated README formatting and added blame
    pick a5f4a0d added cat-file
    ```
to this:  

    ```git
    pick f7f3f6d changed my name a bit
    squash 310154e updated README formatting and added blame
    squash a5f4a0d added cat-file
    ```
    
3. splitting a commit 

#### Changing a commit message

Commit has not been pushed online
`git commit --amend`


#### reset

强制刷新当前工作文档到指定提交id的那个版本 
`git reset --hard commit_id`

#### push
`git push origin dev:dev`

#### 拉取远程分支到本地


```bash
git checkout -b ac_branch origin/ac_branch   拉取远程分支到本地(方式一)
git fetch origin ac_branch:ac_branch  拉取远程分支到本地(方式二)

```

### 基本工作流

```bash
git clone remote_repository
git add .
git commit -m 'some update info'
# 将本地修改push to remote repository
git push -u origin master_local:master_remote
```


## SQL

### 事务和锁机制

#### 事务的ACID特性

**Atomicity 原子性**
事务必须是原子工作单元；对于其数据修改，要么全都执行，要么全都不执行。通常，与某个事务关联的操作具有共同的目标，并且是相互依赖的。如果系统只执行这些操作的一个子集，则可能会破坏事务的总体目标。原子性消除了系统处理操作子集的可能性。

**Consistency 一致性**
事务在完成时，必须使所有的数据都保持一致状态。在相关数据库中，所有规则都必须应用于事务的修改，以保持所有数据的完整性。事务结束时，所有的内部数据结构（如 B 树索引或双向链表）都必须是正确的。某些维护一致性的责任由应用程序开发人员承担，他们必须确保应用程序已强制所有已知的完整性约束。例如，当开发用于转帐的应用程序时，应避免在转帐过程中任意移动小数点。

**Isolation 隔离性**
指的是在并发环境中，当不同的事务同时操纵相同的数据时，每个事务都有各自的完整数据空间。由并发事务所做的修改必须与任何其他并发事务所做的修改隔离。事务查看数据更新时，数据所处的状态要么是另一事务修改它之前的状态，要么是另一事务修改它之后的状态，事务不会查看到中间状态的数据。

**Durability 持久性**
指的是只要事务成功结束，它对数据库所做的更新就必须永久保存下来。即使发生系统崩溃，重新启动数据库系统后，数据库还能恢复到事务成功结束时的状态。

#### 事务隔离级别 

数据库事务的隔离级别有4个，由低到高依次为
Read-uncommitted (读未提交)
Read-committed (读提交)
Repeatable-read （重复读）
Serializable （序列化）
这四个级别可以逐个解决脏读、不可重复读、幻读这几类问题。



### 建库语句

`grant all on dbname.tbname to 'username'@'%' identified by '***'`

### 建表语句

```sql
create table hostpool( 
    hostname varchar(100) not null primary key, 
    ip_addr varchar(100) not null unique, 
    phy_hostname varchar(100) , 
    cpu_count int, 
    disk_total int, 
    mem_total int, 
    mem_reserved int, 
    mem_used int, 
    built_qa varchar(500), 
    status tinyint(1), 
    memo text
);

-- ------------------------------------------------
--           服务列表信息
-- ------------------------------------------------
CREATE TABLE `SERVICE_LIST` (
    `SvcID` bigint(20) NOT NULL AUTO_INCREMENT ,
    `SvcName` varchar(64) NOT NULL COMMENT '服务名称', 
    `SvcEnv` varchar(32) NOT NULL COMMENT '服务环境',
    `CreateTime` bigint(20) NOT NULL,
    `ModifyTime` bigint(20) NOT NULL,
    `Memo` varchar(1024),
    PRIMARY KEY (`Id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT 'es集群信息'

```

### 索引 index

> [参考文档](http://www.runoob.com/mysql/mysql-index.html)
 - key or index 普通索引
 - unique
 - primary
 - fulltext

### 常规操作


#### 授权与取消授权

`GRANT ALL ON $database.* TO $user@'%' IDENTIFIED BY '$PASSWORD'`

`REVOKE all on *.* from dba@localhost;`

#### 设置自增ID字段的起始值

`ALTER TABLE table_name AUTO_INCREMENT = 1;`


### 常规查询

**对指定条件的某个字段去重并计数**

`distinct/DISTINCT`: `SELECT DISTINCT $FILED_NAME FROM $TABLE_NAME;`
`select count(*) as cnt, srv_type from service_list group by srv_type;`

**查询 json 格式的数据**

```sql
SELECT 
    CONCAT(srv_name,"-",ip,"-",port) as server_id,
    DATE_FORMAT(FROM_UNIXTIME(perky_info->"$.start_timestamp"),'%Y/%m/%d %H:%i'), 
    perky_info->"$.start_used_time" as start_used_time 
FROM webapp_tomcat 
GROUP BY server_id 
ORDER BY start_used_time 
DESC LIMIT 500
```

-------


## MongoDB

### 基本用法



## InfluxDB

> [参考文档](https://www.jianshu.com/p/a1344ca86e9b)

### 基本用法

#### construct

name: census

| time | butterflies | honeybees | location | scientist |
| --- | --- | --- | --- | --- |
|2015-08-18T00:00:00Z |     1   |       30  |   1   |   perpetua|
|2015-08-18T00:06:00Z |     11  |       28  |   1   |   langstroth|
|2015-08-18T00:06:00Z |     3   |       28  |   1   |   perpetua|
|2015-08-18T05:54:00Z |     2   |       11  |   2   |   langstroth|

**field**

field key -> butterflies，honeybees
field value -> 1, 30
butterflies和honeybees两列数据称为字段(fields)，influxdb的字段由field key和field value组成。其中butterflies和honeybees为field key，它们为string类型，用于存储元数据。
field key和field value对组成的集合称之为field set

**tag**

location和scientist这两列称为标签(tags)，标签由tag key和tag value组成。location这个tag key有两个tag value：1和2，scientist有两个tag value：langstroth和perpetua。tag key和tag value对组成了tag set

> tags是可选的，但是强烈建议你用上它，因为tag是有索引的，tags相当于SQL中的有索引的列。
> tag value只能是string类型 

**measurement**

measurement是fields，tags以及time列的容器，measurement的名字用于描述存储在其中的字段数据，类似mysql的表名。如上面例子中的measurement为census。measurement相当于SQL中的表。

**retention policy**

retention policy指数据保留策略，示例数据中的retention policy为默认的autogen。它表示数据一直保留永不过期，副本数量为1。你也可以指定数据的保留时间，如30天。

**series**

series是共享同一个retention policy，measurement以及tag set的数据集合。

![](media/15111691153372/15241918726203.jpg)

#### schema 查询语法

`SHOW RETENTION POLICIES [ON <database_name>]`
返回指定数据库的保留策略的列表。

```sql
> SHOW RETENTION POLICIES ON NOAA_water_database

name      duration   shardGroupDuration   replicaN   default
----      --------   ------------------   --------   -------
autogen   0s         168h0m0s             1          true

该保留策略具有无限持续时间，持续时间七天的shard group，副本数为1，并且是数据库的DEFAULT保留策略。
```

`SHOW FIELD KEYS`
`SHOW TAG KEYS`



## Nginx

### 正向代理 Vs 反向代理

- [原文链接](http://www.jianshu.com/p/bed000e1830b)

#### 正向代理  
  
正向代理（Forward Proxy）通常都被简称为代理，就是在用户无法正常访问外部资源，比方说受到GFW的影响无法访问twitter的时候，我们可以通过代理的方式，让用户绕过防火墙，从而连接到目标网络或者服务。

正向代理的工作原理就像一个跳板，比如：我访问不了google.com，但是我能访问一个代理服务器A，A能访问google.com，于是我先连上代理服务器A，告诉他我需要google.com的内容，A就去取回来，然后返回给我。从网站的角度，只在代理服务器来取内容的时候有一次记录，有时候并不知道是用户的请求，也隐藏了用户的资料，这取决于代理告不告诉网站。

结论就是，正向代理是一个位于客户端和原始服务器(origin server)之间的服务器。为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。

#### 反向代理

反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

举个例子，比如我想访问 http://www.test.com/readme ，但www.test.com上并不存在readme页面，于是他是偷偷从另外一台服务器上取回来，然后作为自己的内容返回用户，但用户并不知情。这里所提到的 www.test.com 这个域名对应的服务器就设置了反向代理功能。

结论就是，反向代理服务器对于客户端而言它就像是原始服务器，并且客户端不需要进行任何特别的设置。客户端向反向代理的命名空间(name-space)中的内容发送普通请求，接着反向代理服务器将判断向何处(原始服务器)转交请求，并将获得的内容返回给客户端，就像这些内容原本就是它自己的一样。

![](media/15111691153372/15114028738205.jpg)


## SaltStack

[官方文档](https://docs.saltstack.com/en/getstarted/system/index.html)


## Ansible

### Base Usage

1. outputs a list of matching hosts; does not execute anything else

 ```bash 
ansible -i "ip1,ip2"  all --list-hosts
ansible -i ./test.txt all --list-hosts   
ansible yanxuan-web --list-hosts
```

2. execute shell command

 ```bash
ansible yanxuan-web -m shell -a 'ls /home/webedit'
# -m module
# -a argument
```

3. host pattern

 | <host pattern>参数值 | 解释 | 
| --- | --- |
| * | 通配符 |
| yanxuan-web:online | yanxuan-web 与 online 的并集 |
| yanxuan-web&online | yanxuan-web 与 online 的交集 |
| yanxuan-web!online | 在yanxuan-wen分组但不在online分组 |


## Redis

### Base Command

`redis-cli -h $ip -p $port` login 
`info`  显示当前节点的基本信息
`keys some_key` 显示该key 是否存在
`ttl some_key`  显示该key的ttl -2: 表示不存在， -1: 表示已过期
`get some_key`  显示该key对应的value


## Puppet 

### 常规操作

**client**

`puppet agent ` 
  


## 常规软件安装

### nodejs


```bash
curl -sL https://rpm.nodesource.com/setup_8.x | bash -
yum install nodejs
```


## 网络文件系统存储

### NFS

```bash
yum install nfs-utils -y
systemctl start nfs

cat /etc/exports
/home/nfs  *(rw,sync,all_squash,anonuid=65534,anongid=65534)

showmount -e ${IP}  # 查看该 ip 分享的目录
> Export list for node15.localhost:
> /home/nfs *

exportfs -arv # 刷新 nfsd 配置

mount -t nfs ${IP}:/home/nfs /home/zyl

```

/etc/exports文件内容格式：
`<输出目录> [客户端1 选项（访问权限,用户映射,其他）] [客户端2 选项（访问权限,用户映射,其他）]`
a. 输出目录：
输出目录是指NFS系统中需要共享给客户机使用的目录；

b. 客户端：
客户端是指网络中可以访问这个NFS输出目录的计算机
客户端常用的指定方式
指定ip地址的主机：192.168.0.200
指定子网中的所有主机：192.168.0.0/24 192.168.0.0/255.255.255.0
指定域名的主机：david.bsmart.cn
指定域中的所有主机：*.bsmart.cn
所有主机：`*`

c. 选项：
选项用来设置输出目录的访问权限、用户映射等。
NFS主要有3类选项：
访问权限选项
设置输出目录只读：ro
设置输出目录读写：rw

d. 用户映射选项
all_squash：将远程访问的所有普通用户及所属组都映射为匿名用户或用户组（nfsnobody）；
no_all_squash：与all_squash取反（默认设置）；
root_squash：将root用户及所属组都映射为匿名用户或用户组（默认设置）；
no_root_squash：与rootsquash取反；
anonuid=xxx：将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）；
anongid=xxx：将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）；

e. 其它选项
secure：限制客户端只能从小于1024的tcp/ip端口连接nfs服务器（默认设置）；
insecure：允许客户端从大于1024的tcp/ip端口连接服务器；
sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性；
async：将数据先保存在内存缓冲区中，必要时才写入磁盘；
wdelay：检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率（默认设置）；
no_wdelay：若有写操作则立即执行，应与sync配合使用；
subtree：若输出目录是一个子目录，则nfs服务器将检查其父目录的权限(默认设置)；
no_subtree：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；


