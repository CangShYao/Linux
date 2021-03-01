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

# 遗留问题，这是对当前Linux用户设置，其他用户（如root）无法享用
```

## bash不显示名字和路径的问题

```bash
vim ~/.bash_profile

# 在.bash_profile中添加下面语句
export PS1='[\u@\h \W]\$'

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
