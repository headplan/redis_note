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

#### 服务器下线判断

当一个Sentinel认为被监视的服务器已经下线时 , 它会向网络中的其他Sentinel进行确认 , 判断该服务器是否真的已经下线 .

如果下线的服务器为主服务器 , 那么Sentinel网络将对下线主服务器进行自动故障转移 , 通过将下线主服务器的某个从服务器提升为新的服务器 , 并让其他从服务器转为复制新的主服务器 , 以此来让系统重新回到上线状态 .

![](/assets/fuwqxx.png)

#### 自动故障转移

![](/assets/zidongguzzy.png)

#### 下线主服务器重新上线

![](/assets/xiaxianzhufwq.png)

### Sentinel的配置

指定要监视的主服务器 , 以及相关的监视参数 .

#### 监视配置选项

Sentinel在启动时必须指定相应的配置文件 :

```
$ redis-sentinel sentinel.conf
```

一个Sentinel配置文件至少要包含一个监视配置选项 , 用于指定被监视主服务器的相关信息 :

```
sentinel monitor <name> <ip> <port> <quorum>
```

* name - 用户为被监视主服务器设置的名字
* ip - 被监视主服务器的IP地址
* port - 被监视主服务器的端口号
* quorum - 确认这个主服务器已下线所需要的最少Sentinel数量

例如 , 配置`sentinel monitor mymaster 127.0.0.1 6379 2`表示要监视的主服务器的名字为mymaster , 它的IP地址为127.0.0.1 , 端口号为6379 , 而确认这个主服务器已下线最少需要两个Sentinel同意 .

Sentinel可以自动发现并监视主服务器属下的所有从服务器 , 所以用户只需要给出主服务器的地址和端口号就可以了 .

#### 端口配置选项

如果要在同一台机器上运行多个Sentinel实例 , 用户还需要通过port &lt;number&gt; 选项来为每个Sentinel设置不同的端口号 , 如果不进行设置 , 那么Sentinel的默认端口号为26379 .

举个例子 , 如果要在同一台机器上运行两个Sentinel实例 , 用户可以通过载入以下两个配置文件来分别将两个Sentinel实例的端口号设置为26379和26380 :

```
# sentinel1.conf
port 26379
sentinel monitor...

# sentinel2.conf
port 26380
sentinel monitor...
```

#### Sentinel配置实例

执行以下两条命令 , 将创建两个监视主服务器s1的Sentinel实例 :

```
$ redis-sentinel sentinel1.conf
$ redis-sentinel sentinel2.conf
```

其中sentinel1.conf的内容为 :

```
port 26379
sentinel monitor s1 127.0.0.1 6379 2
```

而sentinel2.conf的内容为 :

```
port 26380
sentinel monitor s1 127.0.0.1 6379 2
```



