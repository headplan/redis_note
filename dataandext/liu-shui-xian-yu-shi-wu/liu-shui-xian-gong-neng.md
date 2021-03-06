# 流水线功能

通过减少客户端与服务器之间的通信次数来提高程序的执行效率 .

#### 通信

在一般情况下 , 用户没执行一个Redis命令 , 客户端与服务器都需要进行一次通信 : 客户端会将命令请求发送给服务器 , 而服务器则会将执行命令所得的结果返回给客户端 .

![](/assets/liushuixian.png)

当程序执行一些复杂的操作时 , 客户端可能需要执行多个命令 , 并与服务器进行多次通信 .

#### 多次通信示例

假设我们正在构建一个为图书打标签\(tag\)的网站 , 这个网站上的每本图书都可以被打上任意多个标签 . 并且为了记录哪些标签的图书是最多人阅览的 , 我们会为每个标签创建一个点击计数器,每当用户浏览网站上的某本书时,程序就会对该书拥有的各个标签的点击计数器执行增一操作 .

我们现在为新添加的书打标签 , 例如一本计算机相关的书 , 可能会给它打上“计算机”、“编程”、“数据库”和“Redis”这四个标签 , 每当访问这本书的介绍页面时 , 程序就会对这四个标签的点击计数器执行+1的操作 :

```
INCR tag::计算机::click_counter 
INCR tag::编程::click_counter 
INCR tag::数据库::click_counter 
INCR tag::Redis::click_counter
```

对于这个点击计数程序来说,客户端要执行的INCR命令次数取决于书本的标签数量 , 而标签的数量越多 , 要执行的INCR命令就越多 .

![](/assets/liushuixian2.png)

在多数情况下 , 客户端在执行Redis命令时 , 大部分等待时间都耗费在了发送命令请求和接收命令回复上面 . 这里就要用到了流水线功能 .

#### 流水线功能

减少程序执行时客户端与服务器之间的通信次数 , 就能够有效地提升程序的性能 , 这就是流水线功能\(pipeline\)的目的 .

Redis的流水线功能允许客户端一次将多个命令请求发送给服务器 , 并将被执行的多个命令请求的结果在一个命令回复中全部返回给客户端 , 使用这个功能可以有效地减少客户端在执行多个命令时需要与服务器进行通信的次数 .

使用流水线功能 , 前面的四次通信会变为一次通信 :

![](/assets/liushuixian3.png)

各个客户端使用流水线功能的方法都不一样 , 例如在Python中 : 

```
from redis import Redis
# 创建 Redis 客户端
client = Redis()
# 创建一个流水线对象,包裹四个 INCR 命令
# transaction=False 表示关闭事务功能,只使用流水线功能 # 关于事务方面的详细信息我们稍后就会介绍
pipeline = client.pipeline(transaction=False) pipeline.incr('tag::计算机::click_counter') pipeline.incr('tag::编程::click_counter') pipeline.incr('tag::数据库::click_counter') pipeline.incr('tag::Redis::click_counter')
# 以流水线形式发送被包裹的四个命令
pipeline.execute()
```

返回结果

```
# 多个命令的结果会以列表的形式返回
# 列表的第一个项 10087 是 tag::计算机::click_counter 的值 
# 第二个项 5001 是 tag::编程::click_counter 的值
# 第三个项 3421 是 tag::数据库::click_counter 的值
# 第四个项 1001 是 tag::Redis::click_counter 的值
[10087, 5001, 3421, 1001]
```



