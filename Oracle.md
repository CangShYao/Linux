# Oracle12c安装

## SELINUX关闭

## FIREWALL关闭

```bash
# root@localhost:/$ 

# 关闭防火墙
systemctl stop firewalld.service

# 禁止防火墙开机启动
systemctl disable firewalld.service

# 查看防火墙状态
systemctl status firewalld.service
```

## 操作步骤(重要)

A、环境准备
B、用户组创建
C、修改系统内核参数
D、修改用户配置
E、数据库应用安装
F、数据库初始化

### A、环境准备

1，各种插件安装

```bash
# root@localhost:/$
yum -y install binutils compat-libcap1 compat-libstdc++-33 compat-libstdc++-33*i686 compat-libstdc++-33*.devel compat-libstdc++-33 compat-libstdc++-33*.devel gcc gcc-c++ glibc glibc*.i686 glibc-devel glibc-devel*.i686 ksh libaio libaio*.i686 libaio-devel libaio-devel*.devel libgcc libgcc*.i686 libstdc++ libstdc++*.i686 libstdc++-devel libstdc++-devel*.devel libXi libXi*.i686 libXtst libXtst*.i686 make sysstat unixODBC unixODBC*.i686 unixODBC-devel unixODBC-devel*.i686
```

2，检测安装是否完全

```bash
# root@localhost:/$
rpm -q binutils compat-libcap1 compat-libstdc++-33 gcc gcc-c++ glibc glibc-devel ksh libaio libaio-devel libgcc libstdc++ libstdc++-devel libXi libXtst make sysstat unixODBC unixODBC-devel
```

### B、用户组创建

1，创建oinstall和dba组

```bash
#root@localhost:/$
groupadd oinstall
groupadd dba
```

2，创建Oracle，设置密码

```bash
# root@localhost:/$
useradd -g oinstall -G dba oracle

# 这样可以设置简单的密码
echo 密码 | passwd --stdin 用户名
```

3，查看用户id

```bash
# root@localhost:/$
id oracle
uid=1002(oracle) gid=1000(oinstall) groups=1000(oinstall),1001(dba)
```

### C、修改内核参数

1，修改系统配置

```bash
# root@localhost:/$
vim /etc/sysctl.conf
```

```bash
# 在最下方添加
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 4294967296
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
```

2，使配置生效

```bash
# root@localhost:/$
sysctl -p
```

### D、修改用户配置

1、修改文件

```bash
# root@localhost:/$
vim /etc/security/limits.conf
```

```bash
# 在最下面添加
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
oracle soft stack 10240
oracle hard stack 10240
```

2、修改登录库引用文件

```bash
# root@localhost:/$
vim /etc/pam.d/login
```

```bash
# 在最下面添加
session required /lib64/security/pam_limits.so
session required pam_limits.so
```

3、修改用户登录环境

```bash
# root@localhost:/$
vim /etc/profile
```

```bash
# 文件末尾添加
# 如果报错，就自己手动输入
if [ $USER = "oracle" ]; then
     if [ $SHELL = "/bin/ksh" ]; then
         ulimit -p 16384
         ulimit -n 65536
     else
         ulimit -u 16384 -n 65536
     fi
fi
```

```bash
# root@localhost:/$
# 使配置文件生效
source /etc/profile
```

4、创建安装目录，配置安装权限

```bash
root@localhost:/$ mkdir -p /u01/app/
root@localhost:/$ chown -R oracle:oinstall /u01/app/
root@localhost:/$ chmod -R 775 /u01/app
```

5、配置Oracle用户环境变量

切换到oracle用户，更改配置

```bash
root@localhost:/$ su - oracle
root@localhost:/$ vim .bash_profile
```

```bash
# 在下面添加
export ORACLE_BASE=/usr/app/oracle
export ORACLE_SID=orcl
```

```bash
# 使配置文件生效
root@localhost:/$ source .bash_profile
```
