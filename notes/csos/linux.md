# 常用命令
- [linux系统包管理工具](##linux系统包管理工具yum和apt-get)
- [内存命令](##内存命令)
- [查询端口号](##查询端口号)
- [tomcat远程调试](##tomcat远程调试)
- [crontab定时任务](##crontab定时任务)
- [删除旧文件](##crontab定时任务)
- [压缩解压文件](##crontab定时任务)
- [date取当前系统时间前后时间](##crontab定时任务)
## linux系统包管理工具yum和apt-get
linux系统基本上分两大类：
- RedHat系列：Redhat、Centos、Fedora等
  - 常见的安装包格式 rpm包,安装rpm包的命令是“rpm -参数” 
  - 包管理工具 yum 
```
yum search xxx           搜索软件包（以名字为关键字） 
yum install xxx          安装xxx软件 
yum info xxx             查看xxx软件的信息 
yum remove xxx           删除软件包 
yum list                 列出软件包 
yum clean                清除缓冲和就的包 
yum provides xxx         以xxx为关键字搜索包（提供的信息为关键字） 
yum update               系统升级 
yum list updates         列出所有升级源上的可以更新包；
```
- Debian系列：Debian、Ubuntu等
  - 常见的安装包格式 deb包,安装deb包的命令是“dpkg -参数” 
  - 包管理工具 apt-get 
```
apt-cache search package 搜索包 
apt-cache show package 获取包的相关信息，如说明、大小、版本等 
sudo apt-get install package 安装包 
sudo apt-get install package - - reinstall 重新安装包 
sudo apt-get -f install 修复安装"-f = ——fix-missing" 
sudo apt-get remove package 删除包 
sudo apt-get remove package - - purge 删除包，包括删除配置文件等 
sudo apt-get update 更新源 
sudo apt-get upgrade 更新已安装的包 
sudo apt-get dist-upgrade 升级系统 
sudo apt-get dselect-upgrade 使用 dselect 升级 
apt-cache depends package 了解使用依赖 
apt-cache rdepends package 是查看该包被哪些包依赖 
sudo apt-get build-dep package 安装相关的编译环境 
apt-get source package 下载该包的源代码 
sudo apt-get clean && sudo apt-get autoclean 清理无用的包 
sudo apt-get check 检查是否有损坏的依赖
```

## 释放内存命令：
### 释放内存
```
sync
echo 1 > /proc/sys/vm/drop_caches

drop_caches的值可以是0-3之间的数字，代表不同的含义：
0：不释放（系统默认值）
1：释放页缓存
2：释放dentries和inodes
3：释放所有缓存

释放完内存后改回去让系统重新自动分配内存。
echo 0 >/proc/sys/vm/drop_caches

free -m #看内存是否已经释放掉了。

如果我们需要释放所有缓存，就输入下面的命令：
echo 3 > /proc/sys/vm/drop_caches
```
## 查询端口号
```
netstat -apn | grep "端口"
```

## tomcat远程调试
```
系统上tomcat下catalina.sh文件内加一行
CATALINA_OPTS="-Xdebug -Xrunjdwp:transport=dt_socket,address=60222,suspend=n,server=y"
```
## crontab定时任务
```
1.安装crontab：
yum install crontabs
服务操作说明：
/sbin/service crond start //启动服务
/sbin/service crond stop //关闭服务
/sbin/service crond restart //重启服务
/sbin/service crond reload //重新载入配置
查看crontab服务状态：
service crond status
手动启动crontab服务：
service crond start
查看crontab服务是否已设置为开机启动，执行命令：
ntsysv
加入开机自动启动：
chkconfig –level 35 crond on

2.自己定义crontab任务
1）列出crontab文件
crontab -l
2）添加crontab任务
crontab -e
```
## 删除旧文件
find /root/sqlbak -mtime +30 -type f -name *.gz -exec rm -f {} \;
删除超过30天的旧文件

## 压缩解压文件
tar -czf /opt/log/s`date +%Y%m%d`.tar.gz all.log-*	压缩opt/logs下的all.log打头的文件进入当前时间的压缩文件
tar -xzvf file.tar.gz 解压tar.gz

## date取当前系统时间前后时间
date -d ‘2 days ago’        //显示2天以前的时间  
date -d ‘60 second ago’     //显示60秒以前的时间   
date -d '3 months 1 day'    //显示3月零1天以后的时间    
date -d '25 Dec' +%j        //显示12月25日在当年的哪一天   
date -d '1970-01-01 00:00:30 +0000' +%s   //自UTC 时间 1970-01-01 00:00:00 以来所经过的秒数
