# 字符串
Redis中最简单的数据结构,存储文字,数字或者二进制数据.这几种类型分别设置了相应的操作命令,可以针对不同的值做不同的处理.
![string](Snip20160706_3.png)
三个字符串分别存储了文字(msg),数字(number)以及二进制数据(bits).
## 基本操作
### 为字符串键设置值
```
SET key value
```
* 将字符串键key的值设置为value,命令返回OK表示设置成功.
* 如果字符串键key已经存在,那么用新值覆盖原来的旧值.
* 复杂度:O(1)
  ```
  SET msg "hello world"
  OK
  SET msg "goodbye" # 覆盖上面的"hello world"
  OK
  ```
```
SET key value [NX|XX]
```
SET命令还支持可选项NX和XX:
* 给定NX选项,那么命令仅在键key不存在的情况下,才进行设置操作,如果key已经存在,则不做操作(即不会覆盖旧值).
* 给定XX选项,那么命令仅在键key存在的情况下,才进行设置操作,也就是说一定会覆盖旧的,如果没有旧的就不操作.

注:在NX和XX选项给定的情况下,SET命令设置成功返回OK,失败返回nil.
```
SET nx-str "如果没有这个键" XX # 如果没有这个键设置失败
nil
SET nx-str "如果没有这个键" NX # 如果没有这个键则创建成功
OK
SET nx-str "如果这个键已经存在" NX # 如果这个键已经存在则创建失败
nil
SET nx-str "如果这个键已经存在" XX # 如果这个键已经存在则替换原来的value
OK
```
### 获取字符串的值
```
GET key
```
* 返回字符串键key存储的值
* 复杂度为O(1)
  ```
  SET msg "hello world"
  OK
  GET msg
  hello world
  SET number 10086
  OK
  GET number
  10086
  ```
### 使用Redis来进行缓存
可以使用Redis来缓存一些经常会被用到,或者需要消耗大量资源的内容.将这些内容放到Redis里(也就是内存里),程序可以快速的取得这些内容.

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
| -- | -- | -- |
| Cache(client) | 设置缓存程序使用的客户端 |  |
| Cache.put(name,content) | 把指定的内容放到缓存里面,并使用name来命名它,以便之后取出 | 调用SET命令 |
| Cache.get(name) | 从缓存中取出以name命名的内容 | 调用GET命令 |

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
### 仅在键不存在的情况下进行设置
```
SETNX key value
```
仅在键key不存在的情况下,将键key的值设置为value,效果和SET key value NX一样.NX的意思就是"Not eXists(不存在)".
* 键不存在并且设置成功时,命令返回1
* 键已经存在而导致设置失败时,命令返回0
* 复杂度为O(1)
```
SETNX new-key "new key"
1
SETNX new-key "new new key"
0 # 键已存在设置失败返回0
GET new-key
new key
```
### 同时设置或获取多个字符串键的值
* MSET key value [key value...]
* 一次为一个或多个字符串键设置值,效果和同时执行多个SET命令一样,命令返回OK
* O(N)复杂度,N为字符串键的数量

* MGET key [key...]
* 一次返回一个或多个字符串键的值,效果和同时执行多个GET命令一样
* O(N)复杂度,N为要获取的字符串键的数量

** 设置或获取个人信息 **

通过MSET和MGET一次性设置和获取用户名,电子邮件,个人网站等信息.
```
MSET test::email "test@qq.com" test::homepage "http://test.com"
MGET test::email test::homepage
```

### 键的命名
因为Redis的数据库不能出现两个同名的键,所有通常会使用field1::,field2::这样的格式来区分同一类型的字符串键.::是分隔符,当然也可以根据个人喜好设置分隔符,比如test/email,test|email等等.
```
一些复杂的分隔符:
user::10086::info - ID为10086的用户的信息
news::sport::cache - 新闻网站体育分类的缓存
message::123321::content - ID为123321的消息的内容
```

### 一次设置多个不存在的键
```
MSETNX key value [key value...]
```
* 只有在所有给定键都不存在的情况下,MSETNX会为所有给定键设置值,效果和同时执行多个SETNX一样.如果给定的键至少有一个是存在的,那么将设置失败.
* 返回1表示设置成功,返回0表示设置失败.
* 复杂度为O(N),N为给定的键数量.
```
MSETNX nx-1 "hello" nx-2 "world" nx-3 "good luck"
1
SET ex-key "bad key here"
OK
MSETNX nx-4 "apple" nx-5 "banana" ex-key "cherry" nx-6 "durian"
0
因为ex-key键已经存在,所以返回0,执行失败
```

### 设置新值并返回旧值
```
GETSET key new-value
```
* 将字符串键的值设置为new-value,并返回字符串键在设置新值之前存储的就的值(old value).
* 复杂度为O(1)
```
SET getset-str "imoldvalue"
OK
GETSET getset-str "imnewvalue"
imoldvalue
GET getset-str
imnewvalue
```
** 用伪代码表示GETSET的定义 **
```
def GETSET(key, new-value)
  old-value = GET(key) # 记录旧值
  SET(key, new-value) # 设置新值
  return old-value # 返回旧值
```

### 追加内容到字符串末尾
```
APPEND key value
```
* 将值value推入到字符串键key已存储内容的末尾,返回值的长度
* O(N)复杂度,其中N为被推入值的长度
* 向一个不存在的键key末尾追加会新创建一个key-value
```
SET myPhone "nokia"
OK
APPEND myPhone "-1110"
10
GET myPhone
"nokia-1110"
```
### 返回值的长度
```
STRLEN key
```
* 返回字符串键key存储的值的长度
* 因为Redis会记录每个字符串值的长度,所以获取该值的长度的复杂度为O(1)
```
SET msg "hello"
OK
STRLEN msg
5
APPEND msg " world"
11
STRLEN msg
11
```