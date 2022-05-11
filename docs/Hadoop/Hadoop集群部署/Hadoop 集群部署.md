# Hadoop 集群部署

CDH6.3.1

测试安装节点

|序号|用途|IP|
|----|------|--------|
|1|CDH01|172.16.1.21|
|2|CDH02|172.16.1.22|
|3|CDH03|172.16.1.23|
|4|CDH04|172.16.1.24|



## 准备工作

### 安装文件下载
**JDK**
jdk-8u181-linux-x64.tar.gz
**mysql-connector**
mysql-connector-java-5.1.34.jar
**CDH安装文件**
百度网盘
链接:https://pan.baidu.com/s/1MrRv-omKl1Ech_ImueiOLQ  密码:8cf0

### 关闭防火墙，关闭selinux
```shell
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
# 关闭selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g'  /etc/sysconfig/selinux
```
### 修改hostname
```shell
vim /etc/hostname

# 添加解析，每台机器上都需要配置
vim /etc/hosts

192.168.31.211 cdh1-211
192.168.31.212 cdh2-212
192.168.31.213 cdh3-213
```

### ssh免密
```shell
# 每台机器上使用下面两条命令，使互相之间能够免密登录
ssh-keygen
ssh-copy-id -i
```

### 开启ntp
```shell
# 停止影响的服务
systemctl stop chronyd
systemctl disable chronyd

# 安装ntp软件
yum -y install ntp

# 配置主节点ntp配置文件
vim /etc/ntp.conf

driftfile /var/lib/ntp/drift
restrict 127.0.0.1
restrict -6 ::1
restrict default nomodify notrap
server ntp1.aliyun.com prefer
minpoll 6
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys 

# 配置其他节点ntp文件，同样修改ntp.conf文件
driftfile /var/lib/ntp/drift
restrict 127.0.0.1
restrict -6 ::1
restrict default kod nomodify notrap nopeer noquery
restrict -6 default kod nomodify notrap nopeer noquery
#这里是主节点的主机名或者ip
server cdh-master.test.com
minpoll 6
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
 
# 启动ntp服务
systemctl start ntpd

# 查看ntp服务
systemctl status ntpd

# 在其它节点上手动同步主节点的时间，换成你自己的主节点IP
ntpdate -u 192.168.31.211

# 配置开机自启动
systemctl enable ntpd

# 确认是否同步成功
ntpstat (若网络没问题，过一会synchronised to NTP server ...)
ntpq -p  (若网络没问题，过一会会看到有一行开头会有个星号)
```

### 虚拟内存参数设置
```shell
# （重启后永久生效）
echo vm.swappiness = 0 >> /etc/sysctl.conf
```

### 禁用 tuned 服务
```shell
# 查看tuned服务是否运行
systemctl status tuned
# 关闭服务
tuned-adm off
# 确保没有已激活的配置
tuned-adm list
如果输出内容中包含No current active profile表示关闭成功
# 关闭并且禁用tuned服务
systemctl stop tuned
systemctl disable tuned
```

### 禁用 Transparent Hugepages (THP)
大多数Linux平台都包含一个名为transparent hugepages的功能，该功能可能会严重降低Hadoop集群的性能
```shell
# 检查THP是否启用
cat /sys/kernel/mm/transparent_hugepage/enabled
cat /sys/kernel/mm/transparent_hugepage/defrag
```
要禁用透明大页面，请在所有群集主机上执行以下步骤
```shell
# 编辑文件，最下面添加两行配置，重启服务器生效
vim /etc/rc.d/rc.local
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag

# 赋予/etc/rc.d/rc.local文件可执行权限
chmod +x /etc/rc.d/rc.local

# 修改GRUB配置, 仅RHEL/CentOS 7.x需要进行该项操作
# 在GRUB_CMDLINE_LINUX项目后面添加一个参数
vim /etc/default/grub 
transparent_hugepage=never

# 执行命令
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 安装java环境

下载jdk反倒/usr/local/java

配置环境变量 /etc/profile

```shell
export JAVA_HOME=/usr/local/java
export JRE_HOME=/usr/local/java/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

### 安装Mysql5.7
CDH 对 MySQL 有一些限制，具体如下
- MySQL 默认的 datadir 目录是/var/lib/mysql，要确保该目录存在的分区有足够的空间。
- 如果 MySQL 启用了 GTID 复制，会导致 Cloudera Manager 安装失败。
- 数据库需要使用UTF8编码。对于MySQL和MariaDB，必须使用UTF8编码，而不是utf8mb4
- 对于MySQL5.7，必须要额外安装 MySQL-shared-compat 或者 MySQL-shared 包，因为 Cloudera Manager Agent 包的安装依赖这两个

下载地址：[MySQL-5.7](https://mirrors.tuna.tsinghua.edu.cn/mysql/downloads/MySQL-5.7/)
```shell
# 首先卸载操作系统可能会自带的mariadb-libs
yum -y remove mariadb-libs

解压mysql rpm-bundle tar包
tar -xvf mysql-5.7.25-1.el7.x86_64.rpm-bundle.tar

# 开始安装mysql, 一定要按照下面的顺序来安装，否则会安装不成功
rpm -ivh mysql-community-common-5.7.35-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.35-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.35-1.el7.x86_64.rpm
 rpm -ivh mysql-community-server-5.7.35-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-compat-5.7.35-1.el7.x86_64.rpm #（安装Cloudera Manager6需要）

# 如果遇到 libaio.so.1() is needed by ...
yum -y install libaio

# 启动mysql服务
systemctl start mysqld
# 如果无法启动，则需要修改mysql数据目录所有者：chown -R mysql:mysql /var/lib/mysql/

# 查看root用户初始密码
grep password /var/log/mysqld.log

# 登录mysql修改root密码
mysql -uroot -p
# 如果密码复杂度不够，则会禁止修改，默认密码规则为：包含数字、大小写字母、特殊字符，同时还有长度要求
# 可以通过修改全局参数来解决，但是还是要求密码长度至少为8位
mysql> set global validate_password_policy=0;
mysql> set password = password('12345678');

# 设置远程登录权限
mysql> grant all privileges on *.* to 'root'@'%' identified by '12345678';
mysql> flush privileges;

# 修改mysql数据库默认编码
查看原数据库编码：mysql> SHOW VARIABLES LIKE 'char%';可以看到数据库和服务端的编码都还不是utf8：
```
编辑/etc/my.cnf文件，在[mysqld]下面添加一行character-set-server=utf8

然后重启mysql
```shell
systemctl restart mysqld
```

my.cnf 参考配置
```
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
transaction-isolation = READ-COMMITTED
# Disabling symbolic-links is recommended to prevent assorted security risks;
# to do so, uncomment this line:
symbolic-links = 0
key_buffer_size = 32M
max_allowed_packet = 32M
thread_stack = 256K
thread_cache_size = 64
query_cache_limit = 8M
query_cache_size = 64M
query_cache_type = 1
max_connections = 550
character-set-server = utf8
#expire_logs_days = 10
#max_binlog_size = 100M
#log_bin should be on a disk with enough free space.
#Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
#system and chown the specified folder to the mysql user.
log_bin=/var/lib/mysql/mysql_binary_log
#In later versions of MySQL, if you enable the binary log and do not set
#a server_id, MySQL will not start. The server_id must be unique within
#the replicating group.
server_id=1
binlog_format = mixed
read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M
# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit  = 2
innodb_log_buffer_size = 64M
innodb_buffer_pool_size = 4G
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
innodb_log_file_size = 512M
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
sql_mode=STRICT_ALL_TABLES
```

## 配置和安装CDH
### 1.配置CM源

**在CDH01节点上安装并启动httpd**
```shell
yum -y install httpd
systemctl start httpd
systemctl enable httpd
```

把下载的文件放到指定的目录下
```bash
cd /var/www/html
# 拷贝cm安装包和jdk到cm6
cp cloudera-manager-* /var/www/html/cm6
cp enterprise-debuginfo-6.3.1-1466458.el7.x86_64.rpm /var/www/html/cm6/
cp oracle-j2sdk1.8-1.8.0+update181-1.x86_64.rpm /var/www/html/cm6/
# 拷贝cdh安装包和元数据文件
cp CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel /var/www/html/cdh6/
cp manifest.json /var/www/html/cdh6/
```

**安装 createrepo 命令，然后进入到 cm6 目录创建 yum 源**
```bash
# 下载 createrepo
yum install -y createrepo

# 进入到cm6安装包的httpd资源位置
cd /var/www/html/cm6

# 创建 yum 源的描述meta
createrepo .
```


**在所有节点上配置 yum 源**

```bash
cat >> /etc/yum.repos.d/cm.repo << EOF
[CM]
name=cm6
baseurl=http://cdh01/cm6/
gpgcheck=0
EOF
```
查看yum源是否生效
```bash
yum clean all
yum repolist
```

### 2.安装
**安装依赖（所有节点）**
yum install -y bind-utils libxslt cyrus-sasl-plain cyrus-sasl-gssapi portmap fuse-libs /lib/lsb/init-functions httpd mod_ssl openssl-devel python-psycopg2 Mysql-python fuse

**安装 cloudera-manager 和agent（cdh01)**
```shell
# 安装JDK（之前准备工作已安装可以不安装，安装181版本
yum install -y oracle-j2sdk1.8.x86_64
# 安装 cloudera-manager
yum install -y cloudera-manager-agent cloudera-manager-daemons cloudera-manager-server cloudera-manager-server-db-2 postgresq-server
```

**安装 mysql（CDH01）**
已安装

初始化管理节点数据库
```bash
# mysql驱动，分发到所有节点
mkdir -p /usr/share/java 
cp mysql-connector-java-5.1.34.jar /usr/share/java/mysql-connector-java.jar

# 执行数据库初始脚本（cdh01）
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql -h localhost -uroot -p12345678 --scm-host localhost scm root Znjf@123
# 初始化完成之后，root用户的密码变成了Znjf@1234
```

**安装其他的agent节点（cdh02、cdh03、cdh04）**
```bash
# 安装JDK（之前准备工作已安装可以不安装，安装181版本
yum install -y oracle-j2sdk1.8.x86_64

# 安装agent
yum install cloudera-manager-daemons cloudera-manager-agent -y
```

**修改配置文件（所有节点）**
修改 Cloudera Agent配置文件/etc/cloudera-scm-agent/config.ini, 配置server_host为主节点cdh01
```bash
vim /etc/cloudera-scm-agent/config.ini 
server_host=cdh01
# 也可以使用sed命令
sed -i "s/server_host=localhost/server_host=CDH01/g" /etc/cloudera-scm-agent/config.ini
```

**启动CDH**
1. 启动 Cloudera Manager(CDH01)
```bash
systemctl start cloudera-scm-server
systemctl enable cloudera-scm-server
```
2. 启动 Cloudera Agent（所有节点）
```bash
systemctl start cloudera-scm-agent
systemctl enable cloudera-scm-agent
```


## WEB 配置
4. 数据库配置（CDH01）
```sql
create database hive default charset utf8;
create user 'hive'@'%' identified by 'Znjf@123';
grant all on hive.* TO 'hive'@'localhost' identified by 'Znjf@123';
grant all on hive.* TO 'hive'@'%' identified by 'Znjf@123';
flush privileges;

create database activity default charset utf8;
grant all on activity.* TO 'activity'@'%' identified by 'Znjf@123';
flush privileges;

create database audit default charset utf8;
grant all on audit.* TO 'audit'@'%' identified by 'Znjf@123';
flush privileges;

create database metadata default charset utf8;
grant all on metadata.* TO 'metadata'@'%' identified by 'Znjf@123';
flush privileges;

create database oozie default charset utf8;
grant all on oozie.* TO 'oozie'@'%' identified by 'Znjf@123';
flush privileges;

create database hue default charset utf8;
grant all on hue.* TO 'hue'@'%' identified by 'Znjf@123';
flush privileges;

create database reports default charset utf8;
grant all on reports.* to 'reports'@'%' identified by 'Znjf@123';
flush privileges;
```

未在已配置的存储库中找到任何parcel 解决办法
```bash
cp /var/www/html/CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel* /opt/cloudera/parcel-repo/
cd /opt/cloudera/parcel-repo/
mv CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha1 CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha

# 重启 cloudera-scm-server
systemctl restart cloudera-scm-server
```


可以通过 172.16.1.21:7180 进行组件服务的安装操作了
用户名/密码：admin/admin


## 其他配置
**权限配置**
```bash
vim /etc/passwd
```
把 hdfs 的 /sbin/nologin 修改为 /bin/bash



