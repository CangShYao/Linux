# Oracle12c安装

为了代码块的美观，因此把当前代码块模式写在代码块第一行。注意查看。  

为了增加用户体验，推荐使用mobaXtrem或Xshell通过ssh连接服务器

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

# result
uid=1002(oracle) gid=1000(oinstall) groups=1000(oinstall),1001(dba)
```

### C、修改内核参数

1，修改系统配置

```bash
# root@localhost:/$

vim /etc/sysctl.conf
```

```bash
# vim

# 在/etc/sysctl.conf最下方添加
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

#### 1、修改文件

```bash
# root@localhost:/$

vim /etc/security/limits.conf
```

```bash
# vim

# 在/etc/security/limits.conf最下面添加
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
oracle soft stack 10240
oracle hard stack 10240
```

#### 2、修改登录库引用文件

```bash
# root@localhost:/$

vim /etc/pam.d/login
```

```bash
# vim

# 在/etc/pam.d/login最下面添加
session required /lib64/security/pam_limits.so
session required pam_limits.so
```

#### 3、修改用户登录环境

```bash
# root@localhost:/$

vim /etc/profile
```

```bash
# vim

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

#### 4、创建安装目录，配置安装权限

```bash
# root@localhost:/$

mkdir -p /u01/app/
chown -R oracle:oinstall /u01/app/
chmod -R 775 /u01/app
```

#### 5、配置Oracle用户环境变量

(1)、切换到oracle用户，更改配置

```bash
# root@localhost:/$

su - oracle

# oracle@localhost:/$
vim .bash_profile
```

```bash
# vim

# 在.bash_profile最下面添加
export ORACLE_BASE=/u01/app/oracle
export ORACLE_SID=orcl
```

```bash
# oracle@localhost:/$
# 使配置文件生效
 source .bash_profile
```

(2)、上传数据库压缩包

1. 创建一个oradb的文件夹

    ```bash
    oracle@localhost:/$ mkdir oradb

    # 然后把linuxx64_12202_database.zip上传到oradb目录下
    ```

2. 上传后，安装unzip插件

    切换到root用户安装unzip zip

    ```bash
    oracle@localhost:/$ su

    root@localhost:/$ yum install -y unzip zip
    ```

3. 解压文件
    切换到oracle用户

    ```bash
    root@localhost:/$ exit
    oracle@localhost:/$ unzip xxx.zip
    ```

（3）、复制相应模板（方便你后续弄错了，可以重新来过）

```bash
# oracle@localhost:/$

cd ~
mkdir etc
cp  /home/oracle/oradb/database/response/* /home/oracle/etc/

# 设置相应权限
su
chmod 700 /home/oralce/etc/*.rsp
# 退出root用户
exit
```

（4）、静默安装配置

```bash
# oracle@localhost:/$
vim /home/oracle/etc/db_install.rsp
```

```bash
# vim

# 可以一一对照，也可以直接使用我的db_install.rsp文件
# 推荐先打开vim的number设置，显示行号


# 30行，安装类型
oracle.install.option=INSTALL_DB_SWONLY

# 35行，安装组
UNIX_GROUP_NAME=oinstall

# 42行，目录
INVENTORY_LOCATION=/u01/app/oraInventory

# 46行，oracle home
ORACLE_HOME=/u01/app/oracle/product/12/db_1

# 51行，oracle 根目录
ORACLE_BASE=/u01/app/oracle

# 63行，安装版本
oracle.install.db.InstallEdition=EE

# 80行
oracle.install.db.OSDBA_GROUP=dba

# 86行
oracle.install.db.OSOPER_GROUP=oinstall

# 91行
oracle.install.db.OSBACKUPDBA_GROUP=oinstall

# 96行
oracle.install.db.OSDGDBA_GROUP=oinstall

# 101行
oracle.install.db.OSKMDBA_GROUP=oinstall

# 106行
oracle.install.db.OSRACDBA_GROUP=oinstall

# 180行，数据库类型
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE

# 185行
oracle.install.db.config.starterdb.globalDBName=orcl

# 190行
oracle.install.db.config.starterdb.SID=orcl

# 232行，自动内存管理
oracle.install.db.config.starterdb.memoryLimit=81920

# 259行，设定所有用户密码为oracle。方便
oracle.install.db.config.starterdb.password.ALL=oracle

# 386行，安全更新通过我的ORACLE支持
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false

# 398行，拒绝安全更新
DECLINE_SECURITY_UPDATES=true

```

#### 6、开始安装

##### 1. 执行db_install.rsp

```bash
# oracle@localhost:/$

cd ~
cd oracle/database/
./runInstaller -silent -responseFile /home/oracle/etc/db_install.rsp

# 然后就等待oracle安装完成
# 记得检查log文件
# 可以另开一个ssh，然后输入以下指令
tail -f xxx.log
```

##### 2. 执行命令

```bash
oracle@localhost$ su
root@localhost$ /u01/app/oraInventory/orainstRoot.sh
root@localhost$ /u01/app/oracle/product/12/db_1/root.sh
root@localhost$ exit
oracle@localhost$ 
```

##### 3. 修改oracle用户环境变量

```bash
# oracle@localhost

cd ~
vim ~/.bash_profile
```

```bash
# vim

#for oracle
export ORACLE_BASE=/u01/app/oracle
export ORACLE_SID=orcl
export ROACLE_PID=oral12
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/usr/lib
export ORACLE_HOME=/u01/app/oracle/product/12/db_1
export PATH=$PATH:$ORACLE_HOME/bin
export LANG="zh_CN.UTF-8"
# export NLS_LANG=AMERICAN_AMERICA.AL32UTF8
export NLS_LANG="SIMPLIFIED CHINESE_CHINA.AL32UTF8"
export NLS_DATE_FORMAT='yyyy-mm-dd hh24:mi:ss'
```

```bash
# oracle@localhost

source ~/.bash_profile
```

##### 4. 配置监听

```bash
# oracle@localhost

# 配置监听
netca /silent /responsefile /home/oracle/etc/netca.rsp

# 启动监听
    lsnrctl start
```

##### 5. 静默建库配置文件修改

```bash
# oracle@localhost

cd ~
vim /etc/dbca.rsp
```

```bash
# vim

# 32行
gdbName=orcl

# 42行
sid=orcl

# 52行
databaseConfigType=SI

# 162行
createAsContainerDatabase=true

# 172行
numberOfPDBs=1

# 182行
pdbName=orclpdb

# 223行,/u01/app/这个是你ORACLE_BASE路径
templateName=/u01/app/oracle/product/12/db_1/assistants/dbca/templates/General_Purpose.dbc

# 273行
emExpressPort=5500

# 313行
omsPort=0

# 468行
characterSet=AL32UTF8

# 526行
listeners=LISTENER

# 574行
memoryPercentage=40

# 594行
automaticMemoryManagement=false

# 604行
totalMemory=0
```

##### 6. 执行静默建库

```bash
# oracle@localhost:/$

dbca -silent -createDatabase  -responseFile  /home/oracle/etc/dbca.rsp

# 等待静默建库
# 中途会让输入SYS、SYSTEM、PDBADMIN的口令
```

##### 7. 创建oracle数据库的用户

```sql
sqlplus / as sysdba ;

create user 用户名 identified by 密码；

grant connect, resource，dba to 用户名;
```
