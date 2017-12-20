# 安装Redis

* https://redis.io/
* https://github.com/antirez/redis

Redis能够兼容绝大部分的POSIX系统 , 除了使用常用的软件管理软件进行安装 , 可以直接使用源码安装 : 

```
# 基本的安装只需6步
1.下载源码包
2.解压源码包
3.建立一个redis目录的软连接,指向redis-0.0.0(这里建立软连目录,没固定在原来的版本目录,有利于未来升级.)
4.进入redis目录
5.编译(确保安装了gcc)
6.安装(安装其实是将运行文件复制到/usr/local/bin/下,这样就可以在任意目录执行redis命令)
wget http://download.redis.io/releases/redis-4.0.6.tar.gz
tar xzf redis-4.0.6.tar.gz
ln -s redis-4.0.6 redis
cd redis
make
make install
```



