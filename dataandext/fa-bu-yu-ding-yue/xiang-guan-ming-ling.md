# 相关命令

#### 订阅频道

```
SUBSCRIBE channel [channel ...]
```

订阅给定的一个或多个频道 .

复杂度为O\(N\),N为被订阅频道的数量 .

```
127.0.0.1:6379> subscribe new::it
Reading messages... (press Ctrl-C to quit)
1) "subscribe" # 订阅频道时返回的消息
2) "new::it"   # 被订阅的频道
3) (integer) 1 # 客户端目前订阅频道的数量
1) "message"   # 这是从频道接收到的消息
2) "new::it"   # 消息来源的频道
3) "hi"        # 消息内容
```

#### 订阅模式

```
PSUBSCRIBE pattern [pattern ...]
```

订阅一个或多个模式,pattern参数可以包含glob风格的匹配符,比如:

* news::\*模式可以匹配news::bussiness、news::it、news::sports::football等频道;
* news::\[ie\]t模式可以匹配news::it频道或者news::et频道;
* news::?t模式可以匹配news::it、news::et、news::at等频道;

复杂度为O\(N\),N为被订阅模式的数量 .

```
redis> PSUBSCRIBE news::[ie]t
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"  # 订阅模式时返回的信息
2) "news::[ie]t" # 被订阅的模式
3) (integer) 1   # 客户端目前订阅的模式数量

1) "pmessage"    # 这是从模式接收到的消息
2) "news::[ie]t" # 被匹配的模式
3) "news::it"    # 消息的来源频道(被匹配的频道)
4) "hello"       # 消息正文

1) "pmessage" 
2) "news::[ie]t" 
3) "news::et" 
4) "world"
```

#### 退订频道和退订模式

**退订频道**

```
UNSUBSCRIBE [channel [channel ...]]
```

退订指定的频道 . 如果执行时没有指定任何频道 , 那么退订已订阅的所有频道 .

复杂度为O\(N\) , N为被退订的频道数量 .

**退订模式**

```
PUNSUBSCRIBE [pattern [pattern ...]]
```

退订指定的模式 . 如果执行时没有指定任何模式 , 那么退订已订阅的所有模式 .

复杂度为O\(M\) , M为服务器中被订阅模式的数量 .

退订命令的行为在各个客户端的表现都不同 , 比如redis-cli客户端就是通过直接退出客户端来进行退订 , 而Python或Ruby客户端则需要显示的执行退订命令 .

```
# redis-cli 客户端
redis> SUBSCRIBE news.it

Reading messages... (press Ctrl-C to quit) 
1) "subscribe"

2) "news::it"

3) (integer) 1
^C

# Python的Redis客户端
>>> from redis import Redis
>>> client = Redis()
>>> pubsub = client.pubsub()

>>> pubsub.subscribe('news::it')
 # 订阅频道
>>> pubsub.channels # 列出已订阅的频道 
set(['news::it'])
>>> pubsub.unsubscribe('news::it') # 退订频道 
>>> pubsub.channels
set([])
```

#### 发布消息

```
PUBLISH channel message
```

将消息发送至指定的频道 , 命令返回接收到消息的订阅者数量 .

复杂度为O\(N\) , N为接收到消息的订阅者数量\(包括通过订阅频道来接收消息的订阅者和通过订阅模式来接收消息的订阅者\) .

```
redis> PUBLISH news::it "hello world"

(integer) 2
redis> PUBLISH news::et "hello again"

(integer) 1
```

![](/assets/fabuxiaoxi.png)

---

### 订阅状态相关命令

**查看被订阅的频道**

```
PUBSUB CHANNELS [pattern]
```

列出目前至少有一个订阅者的频道 . 如果给定了可选的pattern参数 , 那么只列出与模式相匹配的频道 . 

复杂度为O\(N\) , N为服务器中被订阅频道的总数量 . 

```
redis> PUBSUB CHANNELS # 有客户端正在订阅news::et和news::it频道
1) "news::et"
2) "news::it"
redis> PUBSUB CHANNELS # 没有任何频道被订阅
(empty list or set)
redis> PUBSUB CHANNELS new::*
```

**查看频道的订阅者数量**

```
PUBSUB NUMSUB [channel-1 ... channel-N]
```

返回给定频道的订阅者数量 . 复杂度为O\(N\) , N为给定频道的数量 . 

```
redis> PUBSUB NUMSUB new::it new::et
1) "new::it"
2) (integer) 1
3) "new::et"
4) (integer) 1
```

**查看被订阅模式的数量**

```
PUBSUB NUMPAT
```

返回服务器目前被订阅的模式数量 . 复杂度为O\(1\) . 

```
redis> PUBSUB NUMPAT
(integer) 3 # 服务器目前有三个模式被订阅
```



