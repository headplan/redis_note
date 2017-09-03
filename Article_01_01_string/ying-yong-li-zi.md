# 应用实例

### 示例:使用Redis来进行缓存

可以使用Redis来缓存一些经常会被用到,或者需要消耗大量资源的内容.将这些内容放到Redis里\(也就是内存里\),程序可以快速的取得这些内容.

举个例子,对于一个网站,某个页面经常被访问到,或者创建页面时耗费的资源比较多,比如多次访问数据库,生成时间比较长等等.可以使用Redis将这个页面缓存起来,减轻网站的负担,降低网站的延迟值.

```
@app.route("/")
def index():
  # 尝试从缓存里面获取被缓存的页面
  cached_content = cache.get('index')
  # 判断缓存存在,直接返回页面
  if cached_content:
    return cached_content
  else:
    # 页面没有被缓存,访问数据库并重新生成页面
    content = fetch_and_create_index()
    # 缓存页面,方便下次去除
    cache.put('index',content)
    # 返回页面
    return content
```

** 缓存程序的API及其实现 **

| API | 效果 | 实现 |
| --- | --- | --- |
| Cache\(client\) | 设置缓存程序使用的客户端 |  |
| Cache.put\(name,content\) | 把指定的内容放到缓存里面,并使用name来命名它,以便之后取出 | 调用SET命令 |
| Cache.get\(name\) | 从缓存中取出以name命名的内容 | 调用GET命令 |

** 缓存程序的具体实现 **

```
# coding:utf-8
class Cache:
  def __int__(self, client):
    self.client = client
  def put(self, name, content):
    self.client.set(name, content)
  def get(self, name):
    return self.client.get(name)
```

---

### 示例:计数器\(counter\)

很多网站都使用了计数器来记录页面被访问的次数 .

### 示例:计数器\(counter\)

很多网站都使用了计数器来记录页面被访问的次数.  
** 计数器API及其实现 **

* Counter\(name,client\):设置计数器的名字以及客户端

* Counter.incr\(\):将计数器的值增一,然后返回计数器的值,调用INCR命令

* Counter.get\(\):返回计数器当前的值,调用GET命令

* Counter.reset\(n=0\):将计数器的值重置为n,默认重置为0

  * 调用GETSET命令.虽然使用SET命令也可以达到重置的效果,但使用GETSET可以在重置计数器的同时获得计数器之前的值,这有时候会有用.

```
encoding:utf-8
class Counter:
  def init(self, key, client):
    self.key = key
    self.client = client
def incr(self, n=1):
    counter = self.client.incr(self.key, n)
    return int(counter)
def decr(self, n=1):
    counter = self.client.decr(self.key, n)
    return int(counter)
def reset(self, n=0):
    counter = self.client.getset(self.key, n)
    if counter is None:
      counter = 0
    return int(counter)
def get(self):
    counter = self.client.get(self.key)
    if counter is None:
```

### id生成器

很多网站在创建新条目的时候,都会使用id生成器来为条目创建唯一标识符.例如一个论坛每注册一个新用户都会为这个新用户创建一个用户ID,比如12345,然后访问user/12345就可以看到这个用户的个人页面.或者一个用户发一个帖子,这个帖子也会创建一个帖子ID,/topic/10086 就可以看到这个帖子的内容.ID通常都是连续的,比如1003,1004,1005等.

** id生成器API以其实现 **

* IdGenerator\(name,client\):设置id生成器的名字和客户端
* IdGenerator.gen\(\):生成一个新的自增id,调用INCR命令
* IdGenerator.init\(n\):保留前n个id,防止抢注,需要在系统开始运作前执行,否则会出现重复id.例如,要保留前一万个id,那么就需要执行IdGenerator.init\(10000\),这样生成器创建的id就会从10001开始.调用SET命令.

```
coding:utf-8
class IdGenerator:
def init(self, key, client):
    self.key = key
    self.client = client
def init(self, n):
    self.client.set(self.key, n)
def gen(self):
    new_id = self.client.incr(self.key)
    return int(new_id)
```

### 示例:实现在线人数统计

### 示例:使用Redis缓存热门图片



