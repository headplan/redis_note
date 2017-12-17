# Sentinel

监视主从服务器 , 并在主服务器下线时自动进行故障转移 .

#### 启动Redis Sentinel

通过执行Redis安装文件夹中的redis-sentinel程序 , 可以启动一个Sentinel实例 : 

```
$ redis-sentinel sentinel.conf
```

因为Redis的Sentinel实际上就是一个运行在Sentienl模式下的Redis服务器 , 所以我们同样可以使用以下命令来启动一个Sentinel实例 : 

```
$ redis-server sentinel.conf --sentinel
```

启动Sentinel时需要指定配置文件 , 该文件记录了要监视的主服务器 , 以及相关的配置参数 . 

#### 使用Sentinel监视主服务器以及它的从服务器

每个Sentinel实例可以监视任意多个主服务器 , 以及被监视的主服务器属下的所有从服务器 . 

![](/assets/sentienl.png)

#### Sentinel网络

多个Sentinel实例可以监视同一个主服务器 , 监视相同主服务器的这些Sentinel们会自动地互相连接 , 组成一个分布式的Sentinel网络 , 互相通信并交换彼此关于被监视服务器的信息 . 

右图展示了一个由三个Sentinel实例组成的Sentinel网络 , 这三个Sentinel监视着主服务器s1以及它的两个从服务器s2和s3 . 

![](/assets/sentinel22.png)

