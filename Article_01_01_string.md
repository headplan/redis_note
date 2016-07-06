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
| 0:3 | 1:3 | 2:3 |
| 0:4 | 1:4 | 2:4 |

