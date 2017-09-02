# 应用实例

### 使用Redis来进行缓存

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



