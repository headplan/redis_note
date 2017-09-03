# 应用实例

### 示例:使用散列重新实现计算器

```
Counter(name, client):设置计数器的名字以及客户端
Counter.incr():将计数器的值增一,然后返回计数器的值,调用HINCRBY命令
Counter.get():返回计数器当前的值,调用HGET命令
Counter.reset(n=0):将计数器的值重置为n,默认重置为0,调用HGET命令和HSET命令
```

这个计数器的功能和API,跟之前用字符串键实现的计数器完全一样,不同之处在于,这个实现会将所有计数器都存储在同一个散列里面,一个域值对就是一个计数器.

```
# encoding:utf-8

# 保存所有计数器的散列键
COUNTER_KEY = 'hash-counter'

class Counter:
  def __init__(self, name, client):
    self.name = name
    self.client = client

  def incr(self, n=1)
    counter = self.client.hincrby(COUNTER_KEY, self.name, n)
    return int(counter)
  def decr(self, n=1)
    minus_n = -n
    counter = self.client.hincrby(COUNTER_KEY, self.name, minus_n)
    return int(counter)
  def reset(self, n=0)
    # 这里的代码带有竞争条件
    # 可以在学习了事务之后,回过头来进行修改
    counter = self.client.hget(COUNTER_KEY, self.name)
    if counter is None:
      counter = 0
    self.client.hset(COUNTER_KEY, self.name, n)
    return int(counter)
  def get(self):
    counter = self.client.hget(COUNTER_KEY, self.name)
    if counter is None:
      counter = 0
    return int(counter)
```



