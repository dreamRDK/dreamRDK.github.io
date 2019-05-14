---
layout: post
title: 'Linux下安装redis'
subtitle: ''
date: 2018-10-23
categories: 技术
tags: linux redis
---
## linux下安装redis步骤及遇到的坑
### 1. redis官网下载最新redis并上传到服务器
```java
redis 官网下载地址https://redis.io/download
```
### 2. 解压安装包
### 3. 进入redis目录下使用make命令编译
	这时可能没有gcc会报错
### 4.安装gcc
```java
yum -y install gcc
验证gcc是否安装成功
rpm -qa|grep gcc
```
	
### 5.继续 make redis
	这时还是会报错，jemalloc 找不到
	这是一个管理内存的库，可以从github下载压缩包，解压
```java
github地址https://github.com/jemalloc/jemalloc/releases
```
	预编译jemalloc
```java
./configure --prefix=/usr/local/jemalloc
```
	然后编译 jemalloc
```java
make && make install
```
### 5.回到redis目录继续编译
```java
make MALLOC=/usr/local/jemalloc/lib
```
	
### 6.继续redis安装
```java
cd src
make install PREFIX=/usr/local/redis-5.0.0/
```
	
### 7.make test
	然后会报错，需要安装tcl
```java
yum -y install tcl
```	

## redis 启动
	复制redis.conf文件到/usr/local/redis-5.0.0/bin
	修改redis.conf 配置
	vi /etc/redis.conf 
#查找daemonize no改为 yes以守护进程方式运行 即以后台运行方式去启动
daemonize yes 
#修改dir ./为绝对路径, 默认的话redis-server启动时会在当前目录生成或读取dump.rdb 所以如果在根目录下执行redis-server /etc/redis.conf的话
#, 读取的是根目录下的dump.rdb,为了使redis-server可在任意目录下执行 所以此处将dir改为绝对路径 
dir /usr/local/redis-4.0.11/bin
#修改appendonly为yes 
#指定是否在每次更新操作后进行日志记录， Redis在默认情况下是异步的把数据写入磁盘， 
#如果不开启，可能会在断电时导致一段时间内的数据丢失。 因为 redis本身同步数据文件是按上面save条件来同步的， 
#所以有的数据会在一段时间内只存在于内存中。默认为no 
appendonly yes 
#redis 日志生成位置
logfile "/app/log/redis.log"

启动redis
```java
cd /usr/local/redis-5.0.0/bin
./redis-server redis.conf
```

启动客户端测试
```java
./redis-cli 
```

## 使用脚本设置开机自启动

启动脚本 redis_init_script 位于位于Redis的 /utils/ 目录下

```java
#!/bin/sh
#说明启动优先级
# chkconfig:   2345 90 10
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

### BEGIN INIT INFO
# Provides:     redis_6379
# Default-Start:        2 3 4 5
# Default-Stop:         0 1 6
# Short-Description:    Redis data structure server
# Description:          Redis data structure server. See https://redis.io
### END INIT INFO
#端口
REDISPORT=6379
REDIS=redis
#服务启动脚本位置
EXEC=/usr/local/redis-4.0.11/bin/redis-server
#客户端连接脚本位置
CLIEXEC=/usr/local/redis-4.0.11/bin/redis-cli
#启动PID所在位置
PIDFILE=/var/run/redis_${REDISPORT}.pid
#启动配置文件所在位置
CONF="/usr/local/redis-4.0.11/bin/${REDIS}.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```

将启动脚本复制到/etc/init.d目录下，本例将启动脚本命名为redis（通常都以d结尾表示是后台自启动服务）
进入/etc/init.d目录下设置为开机自启动，直接配置开启自启动 chkconfig redis on
如果是ubuntu系统使用以下命令
update-rc.d redis defaults
可以使用 service redis start 启动redis
service redis stop 停止redis


	
