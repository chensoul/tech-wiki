

# 1、环境说明

系统环境：

- 操作系统：`CentOs 7.7.1908`
- Hadoop版本：`CDH6.3.2`
- JDK版本：`java-1.8.0-openjdk`
- 服务器IP：192.168.1.131
- 运行用户：root

注意：


1、CDH6.3.2安装包需要提前下载，并且配置号yum源。

    链接：https://pan.baidu.com/s/1MRbwWSgyvo9vQMuI5Xq8OQ 
    提取码：abcD



# 2、服务器配置



## 2.1、禁用防火墙

保存存在的iptables规则：

```bash
iptables-save > ~/firewall.rules
```

禁用防火墙：

```bash
systemctl disable firewalld 
systemctl stop firewalld
```



## 2.2、设置SELinux模式

```bash
setenforce 0  >/dev/null 2>&1 
sed -i "s/.*SELINUX=.*/SELINUX=disabled/g" /etc/selinux/config
```

一个考虑操作系统情况的综合示例：

```bash
[ -f /etc/init.d/iptables ] && FIREWALL="iptables"
[ -f /etc/init.d/SuSEfirewall2_setup ] && FIREWALL="SuSEfirewall2_setup"
[ -f /etc/init.d/boot.apparmor ] && SELINUX="boot.apparmor"
[ -f /usr/sbin/setenforce ] && SELINUX="selinux"
service $FIREWALL stop >/dev/null 2>&1
chkconfig $FIREWALL off > /dev/null 2>&1
if [ $SELINUX == "selinux" ]; then
    sed -i "s/.*SELINUX=.*/SELINUX=disabled/g" /etc/selinux/config
    setenforce 0  >/dev/null 2>&1
elif [ $SELINUX == "boot.apparmor" ]; then
    service boot.apparmor stop >/dev/null 2>&1
    chkconfig boot.apparmor off >/dev/null 2>&1
fi
```



## 2.3、禁用IPv6

CDH 要求使用 IPv4，IPv6 不支持，禁用IPv6方法：

    cat > /etc/sysctl.conf <<EOF
    net.ipv6.conf.all.disable_ipv6=1
    net.ipv6.conf.default.disable_ipv6=1
    net.ipv6.conf.lo.disable_ipv6=1
    EOF

使其生效：

    sysctl -p

最后确认是否已禁用：

    cat /proc/sys/net/ipv6/conf/all/disable_ipv6 
    1



## 2.4、配置hosts

修改每个节点/etc/hosts文：

```bash
cat > /etc/hosts <<EOF
127.0.0.1       localhost
192.168.1.131 cdh1 cdh1.example.com
EOF
```

在每个节点分别配置网络名称。例如在cdh1节点上：

```bash
$ hostname
cdh1
hostnamectl set-hostname $(hostname)
```

修改网络名称：

```bash
$ cat > /etc/sysconfig/network<<EOF
HOSTNAME=$(hostname)
EOF
```

查看hostname是否修改过来：

```bash
$ uname -a
Linux cdh1.example.com 3.10.0-327.4.5.el7.x86_64 #1 SMP Mon Jan 25 22:07:14 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```

查看ip是否正确：

```bash
$ yum install net-tools -y 
$ ifconfig |grep -B1 broadcast
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
--
enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.11  netmask 255.255.255.0  broadcast 192.168.56.255
```

检查一下：

```bash
$ yum install bind-utils -y && host -v -t A `hostname`
```



## 2.5、设置中文编码和时区

查看中文：

```bash
locale -a | grep zh
```

如果没有，则安装：

```bash
yum install glibc-common
```

修改配置文件：

```bash
cat >> ~/.bashrc <<EOF
export LC_ALL=zh_CN.utf8
export LANG=zh_CN.utf8
export LANGUAGE=zh_CN.utf8
EOF
localedef -c -f UTF-8 -i zh_CN zh_CN.utf8 
source ~/.bashrc 
echo $LANG
```

设置上海时区：

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```



## 2.6、设置root密码

```bash
# Setup sudo to allow no-password sudo for "admin". Additionally,
# make "admin" an exempt group so that the PATH is inherited.
$ cp /etc/sudoers /etc/sudoers.orig
$ echo "root            ALL=(ALL)               NOPASSWD: ALL" >> /etc/sudoers
$ echo 'redhat'|passwd root --stdin >/dev/null 2>&1
```



## 2.7、设置命名服务

```bash
# http://ithelpblog.com/os/linux/redhat/centos-redhat/howto-fix-couldnt-resolve-host-on-centos-redhat-rhel-fedora/
# http://stackoverflow.com/a/850731/1486325
$ echo "nameserver 8.8.8.8" | tee -a /etc/resolv.conf
$ echo "nameserver 8.8.4.4" | tee -a /etc/resolv.conf
```



## 2.8、配置SSH无密码登陆

先生成ssh公钥、私钥：

```bash
[ ! -d ~/.ssh ] && ( mkdir ~/.ssh ) && ( chmod 600 ~/.ssh )
[ ! -f ~/.ssh/id_rsa.pub ] && (yes|ssh-keygen -f ~/.ssh/id_rsa -t rsa -N "") \
  && ( chmod 600 ~/.ssh/id_rsa.pub ) && cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

配置无密码登陆：

```bash
ssh-copy-id -i /root/.ssh/id_rsa.pub root@cdh1
```

也可以使用expect来操作：

```bash
yum install expect -y
ssh_nopassword.expect cdh1 root #root是要设置的密码
```

ssh\_nopassword.expect文件内容如下：

```bash
#! /usr/bin/expect -f
set host [lindex $argv 0]
set password [lindex $argv 1]
spawn ssh-copy-id -i /root/.ssh/id_rsa.pub root@$host
expect {
yes/no  {send "yes\r";exp_continue}
-nocase "password:" {send "$password\r"}
}
expect eof
```



## 2.9、设置时钟同步

```bash
yum install ntp -y
```

修改时钟服务器：

```bash
$ sed -i "/^server/ d" /etc/ntp.conf   #删除行
$ cat << EOF | sudo tee -a /etc/ntp.conf
server ntp1.aliyun.com
server ntp2.aliyun.com
server ntp3.aliyun.com
server ntp4.aliyun.com
EOF
```

启动 ntp：

```bash
sudo systemctl start ntpd
sudo systemctl enable ntpd
```

同步硬件时钟：

```bash
hwclock --systohc
hwclock --show
```

同步时间：

```bash
ntpdate -u ntp1.aliyun.com
```

设置定时任务定时同步时间： 

## 2.10、虚拟内存设置

Cloudera 建议将`/proc/sys/vm/swappiness`设置为 0，当前设置为 60。


临时解决，通过`echo 0 > /proc/sys/vm/swappiness`即可解决。


永久解决：

    sysctl -w vm.swappiness=0 
    echo vm.swappiness = 0 >> /etc/sysctl.conf



## 2.11、设置透明大页面

```bash
echo "echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag" >/etc/rc.local
echo "echo never > /sys/kernel/mm/redhat_transparent_hugepage/enabled" >/etc/rc.local
chmod +x /etc/rc.d/rc.local
```



## 2.12、设置文件打开数

```bash
function addline {
    line=$1
    file=$2
    tempstr=`grep "$line" $file  2>/dev/null`
    if [ "$tempstr" == "" ]; then
        echo "$line" >>$file
    fi
}
rst=`grep "^fs.file-max" /etc/sysctl.conf`
if [ "x$rst" = "x" ] ; then
	echo "fs.file-max = 727680" >> /etc/sysctl.conf || exit $?
else
	sed -i "s:^fs.file-max.*:fs.file-max = 727680:g" /etc/sysctl.conf
fi
addline "*	soft		nofile	327680" /etc/security/limits.conf
addline "*	hard	    nofile	327680" /etc/security/limits.conf
addline "root	soft	nofile	327680" /etc/security/limits.conf
addline "root	hard	nofile	327680" /etc/security/limits.conf
curuser=`whoami`
for user in hdfs mapred hbase zookeeper hive impala flume $curuser ;do
    addline "$user	soft	nproc	131072" /etc/security/limits.conf
    addline "$user	hard	nproc	131072" /etc/security/limits.conf
done
```



# 3、安装过程



## 3.1、配置yum源

安装vsftpd，参考：<https://phoenixnap.com/kb/how-to-setup-ftp-server-install-vsftpd-centos-7>

```bash
yum -y install httpd
service httpd restart
```

搭建本地yum源：

```bash
unzip CDH6.3.2.zip
cd CDH6.3.2
tar zxvf cm6.3.1-redhat7.tar.gz
mv cm6.3.1 /var/www/html/ 
cd /var/www/html/cm6.3.1 
createrepo .
yum clean all
```

创建yum文件：

```basj
cat >> /etc/yum.repos.d/cloudera-manager.repo  <<EOF
[cloudera-manager]
name=Cloudera Manager 6.3.1
baseurl=http://192.168.1.131/cm6.3.1/
gpgcheck=0
enabled=1
EOF
```



## 3.2、安装jdk

```bash
yum install oracle-j2sdk1.8 -y
cat << EOF | sudo tee -a /etc/profile
export JAVA_HOME=/usr/lib/jvm/java
export CLASSPATH=.:\$JAVA_HOME/lib:\$JAVA_HOME/jre/lib:\$CLASSPATH
export PATH=\$JAVA_HOME/bin:\$JAVA_HOME/jre/bin:\$PATH
EOF
source /etc/profile
```



## 3.3、安装MySQL数据库

    tar -zxvf mysql-5.7.27-1.el7.x86_64.rpm-bundle.tar 
    yum remove mariadb* -y \
    && yum install -y libaio numactl \
    && rpm -ivh mysql-community-common-5.7.27-1.el7.x86_64.rpm \
    && rpm -ivh mysql-community-libs-5.7.27-1.el7.x86_64.rpm \
    && rpm -ivh mysql-community-client-5.7.27-1.el7.x86_64.rpm \
    && rpm -ivh mysql-community-server-5.7.27-1.el7.x86_64.rpm \
    && rpm -ivh mysql-community-libs-compat-5.7.27-1.el7.x86_64.rpm \
    && echo character-set-server=utf8 >> /etc/my.cnf \
    && yum clean all \
    && rpm -qa |grep mysql

数据库：

```bash
cat >> /root/c.sql <<EOF
set password for root@localhost = password('123456Aa.');
grant all privileges on *.* to 'root'@'%' identified by '123456Aa.';
flush privileges;
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY '123456Aa.';
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY '123456Aa.';
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY '123456Aa.';
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY '123456Aa.';
GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY '123456Aa.';
GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY '123456Aa.';
GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY '123456Aa.';
GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY '123456Aa.';
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY '123456Aa.';
SHOW DATABASES;
EOF
```

获取MySQL初始密码：

```bash
systemctl start mysqld 
PASSWORD=`grep password /var/log/mysqld.log | sed 's/.*\(............\)$/\1/'`
echo $PASSWORD
```

执行SQL脚本 `{password}`为刚刚查询的初始密码：

```bash
mysql -uroot –p"$PASSWORD"
```

登陆后执行：

    source /root/c.sql

配置mysql jdbc驱动

```bash
mkdir -p /usr/share/java/ 
cp mysql-connector-java-5.1.48.jar /usr/share/java/mysql-connector-java.jar 
ls /usr/share/java/
```



## 3.4、安装Cloudera Manager

```bash
yum install -y cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server 
yum clean all 
rpm -qa | grep cloudera-manager
```



## 配置parcel库

```bash
cp CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel* /opt/cloudera/parcel-repo
cd /opt/cloudera/parcel-repo
sha1sum CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel | awk '{ print $1 }' > CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha 
chown -R cloudera-scm:cloudera-scm /opt/cloudera/parcel-repo/* 
ll /opt/cloudera/parcel-repo/
```



## 初始化scm库

```bash
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql -h cdh1.example.com scm scm 123456Aa.
```



## 启动cloudera-server服务

```bash
systemctl start cloudera-scm-server
systemctl enable cloudera-scm-server
```

使用`tail`命令查看运行日志，当出现`Started Jetty server.`字眼时表示启动成功

```bash
tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log | grep "INFO WebServerImpl:com.cloudera.server.cmf.WebServerImpl: Started Jetty server"
```



## 登陆CM管理平台

访问 <http://192.168.1.131:7180/cmf/login>，账号密码：admin/admin
