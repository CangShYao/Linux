# CentOS-7一些问题

## yum换源

```bash
# 更换之前确保自己安装wget
yum list wget

#若没有安装
yum -y install wget

# 备份原来的源
cd /etc/yum.repos.d
sudo mv CentOS-Base.repo CentOS-Base.repo.bak

# 下载阿里源
sudo wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# 生成yum缓存
yum clean all  
yum makecache
```

## vim安装（个人喜好）

```bash
# 查找vim安装的版本
rpm -qa | grep vim

# 安装剩下没安装的vim
yum -y install vim-enhanced or  
yum -y install vim-*

# vim设置显示行数
set nu

# set nu永久生效，需修改vim配置
vim ~/.vimrc

# 在.vimrc中添加set nu
set nu

# root用户可以通过编辑/etc/vimrc来实现全局添加
vim etc/vimrc
```

## bash不显示名字和路径的问题

```bash
vim ~/.bash_profile

# 在.bash_profile中添加下面语句
export PS1='[\u@\h \W]\$'

# 这里说一下我自己的配置（主机）
# 普通用户
PS1="\[\e[32;40m\]\u\[\e[0m\]@\[\e[35;40m\]\h\[\e[0m\]:\W\$ "

# root用户
PS1='\[\e[37;40m\]\[\e[37;41m\]\u\[\e[37;41m\]@\h\[\e[37;40m\]:\W\$ '

# 使bash配置文件生效
source ~/.bash_profile
```

## JDK配置

```bash
# 解压JDK到指定目录
# Java版本可以换成其他的
# /home/jdk可以换成其他路径
tar -zxvf jdk1.8.0_171.tar.gz -C /home/jdk

# 修改环境变量
vim /etc/profile

# 在末尾添加如下参数
# 这里的/home/jdk就是上面你解压的路径
export JAVA_HOME=/home/jdk
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${CLASSPATH}:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH

# 使配置文件生效
source /etc/profile

# 验证Java环境是否生效
java -version

# 如果Java环境没有生效，请检查路径是否正确
# 请检查参数是否正确
```

## 本机与服务器传输文件

[^_^]: # 终于不用理会恶心的FTP了

```bash
# Linux安装lrzsz插件
yum install -y lrzsz

# 上传文件：输入rz命令
# mobaXtrem右键终端，点击send file using z-modem

# 下载文件：输入sz filename命令
# mobaXtrem右键终端，点击receive file using z-modem
```
