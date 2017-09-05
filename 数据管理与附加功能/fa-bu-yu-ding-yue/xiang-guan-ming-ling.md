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

```



