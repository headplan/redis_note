# twenmproxy

#### 扩展系统处理写请求的能力

前面介绍了如何使用Redis提供的复制功能来扩展系统的读性能 , 但是在实际的使用Redis的时候 , 用户除了可能遇上读性能问题之外 , 还可能遇上写性能问题 .

为了解决这个问题 , 需要使用多台服务器来处理客户端的读请求 , 还必须使用多台服务器来处理客户端的写请求 .

达到这个目的的其中一种方法是使用分片\(shard\) , 这项技术的核心原理是将整个数据库分为多个部分 , 使用不同的服务器来储存不同部分 , 并负责处理相应部分的命令请求\(包括读请求和写请求\) .

![](/assets/fenpian1.png)

#### 关于twemproxy

twemproxy是Twitter开发的一个代理服务器 , 它兼容Redis和Memcached , 允许用户将多个Redis服务器添加到一个服务器池\(pool\)里面 , 并通过用户选择的散列函数和分布函数 , 将来自客户端的命令请求分发给服务器池中的各个服务器 . 

通过使用twemproxy , 可以将数据库分片到多台Redis服务器上面 , 并使用这些服务器来分担系统负责以及数据库容量 . 在服务器硬件条件相同的情况下 , 对于一个包含N台Redis服务器的池来说 , 池中每台服务器平均包含整个数据库数据的1/N , 并处理平均1/N的客户端命令请求 . 向池里面添加更多服务器可以线性地扩展系统处理命 令请求的能力,以及系统能够保存的数据量 . 

![](/assets/twemproxy1.png)

#### 安装配置并运行Twemproxy

**安装**

https://github.com/twitter/twemproxy

```
$ ./configure
$ make
$ sudo make install
```

**配置**

使用yml格式编写配置文件 : 

```
my_redis_server_group:    # 服务器池的名字,支持创建多个服务器池
  listen: 127.0.0.1:22121 # 这个服务器池的监听地址和端口号
  hash: fnv1a_64          # 键散列算法,用于将键映射为一个散列值
  distribution: ketama    # 键分布算法,决定键被分布到哪个服务器
  redis: true             # 代理Redis命令请求,不给定时默认代理Memcached请求
  servers:                # 池中各个服务器的地址和端口号,以及权重
   - 127.0.0.1:6379:1
   - 127.0.0.1:6380:1
   - 127.0.0.1:6381:1
```

**运行**

```
$ nutcracker -c nutcracker.yml
```

#### 使用twemproxy

用户可以使用Redis客户端连接上twemproxy , 并向twemproxy发送命令请求 , 接收到命令请求的twemproxy会根据用户指定的散列函数和分布函数来决定将命令交给哪个服务器来处理 : 

```
$ ./redis-cli -p 22121
127.0.0.1:22121> SET msg "hello world"
OK
127.0.0.1:22121> SADD numbers 1 3 5 7 9
(integer) 5
127.0.0.1:22121> RPUSH lst "a" "b" "c" "d" "e" "f" "g"
(integer) 7
127.0.0.1:22121> ZADD fruits-price 8.0 "apple" 3.5 "banana" 4.2 "cherry" 
(integer) 3
127.0.0.1:22121> INCRBY counter 10086
(integer) 10086
```

当twemproxy获取到命令请求的回复时 , 就会将命令回复返回给发送命令请求的客户端 ,整个代理过程对于客户端来说是完全透明的 . 

![](/assets/twemproxy2.png)

### 键分布

通过散列函数和分布函数来将数据库的键放置到不同的服务器里面 . 当客户端向twemproxy发送一个针对键key的命令请求时 , twemproxy会根据以下公式计算出应该由服务器池中的哪个服务器来处理这个命令请求 : 

```
# 计算键key的散列值
hash_value = hash_function(key)
```



