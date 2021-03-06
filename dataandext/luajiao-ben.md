# Lua脚本

在服务器端执行复杂的操作 .

**附加功能**

* 流水线 - 打包发送多条命令 , 并在一个回复里面接收所有被执行命令的结果
* 事务 - 一次执行多条命令 , 被执行的命令要么全部执行 , 要么都不执行 , 事务执行过程中不会被其他工作打断 . 
* 乐观锁 - 监视特定的键 , 防止事务出现竞争条件 . 

附加功能非常有用 , 但是他们也存在一些缺陷 .

#### 流水线的缺陷

尽管使用流水线可以一次发送多个命令 , 但是对于一个由多个命令组成的复杂操作来说 , 为了执行该操作而不断地重复发送相同的命令 , 这并不是最高效的做法 , 会对网络资源造成浪费 .

如果我们有办法避免重复地发送相同的命令 , 那么客户端就可以减少花在网络传输方面的时间 , 操作就可以执行得更快 .

#### 事务和乐观锁的缺陷

虽然使用事务可以一次执行多个命令 , 并且通过乐观锁可以防止事务产生竞争条件 , 但是在实际中 , 要正确地使用事务和乐观锁并不是一件容易的事情 .

1.对于一个复杂的事务来说 , 通常需要仔细思考才能知道应该对哪些键进行加锁 : 锁了不应该锁的键会增加事务失败的机会 , 甚至可能会造成程序出错 ; 而忘了对应该锁的键进行加锁的话 , 程序又会产生竞争条件 .

2.有时候为了防止竞争条件发生 , 即使操作本身不需要用到事务 , 但是为了让乐观锁生效 , 我们也会使用事务将命令包裹起来 , 这增加了实现的复杂度 , 并且带来额外的性能损耗 .

例如前面事务功能中实现的ZDECRBY命令 , 使用事务仅仅是为了让WATCH生效而已 :

```py
def ZDECRBY(key, decrment, member):
    # 监视输入的有序集合
    WATCH key
    # 取得元素当前的分 值
    old_score = ZSCORE key member
    # 使用当前分值减去指定的减量,得出新的分值 
    new_score = old_score - decrment
    # 使用事务包裹 ZADD 命令
    # 确保 ZADD 命令只会在有序集合没有被修改的情况下 执行
    MULTI
    ZADD key new_score member 
    # 为元素设置新分值,覆盖现有的分值 
    EXEC
```

最好的方式 , 就是可以以事务的方式执行多个命令 , 还不会引入任何竞争条件 , 来替代事务和乐观锁 .

> #### **现在遇到两个问题**
>
> **避免失误被勿用**
>
> 如果有一种方法可以用事务的方式执行多个命令 , 并且这种方法不会引入竞争条件 , 那么就可以使用这种方式来代替事务和乐观锁了 .
>
> **扩展Redis功能的繁琐**
>
> Redis针对每种数据结构都提供了响应的操作命令 , 也对数据库本身提供了操作命令 . 但是如果我们需要对数据进行一些Redis命令不支持的操作时 , 就需要客户端先取出数据 , 然后由客户端对数据进行处理 , 最后再讲处理后的数据存储回Redis服务器 .
>
> 例如 :
>
> Redis没有提供删除列表里面所有偶数数字的命令 , 要执行这一操作 , 客户端需要先取出列表里面所有项 , 然后在客户端里面进行过滤 , 最后将过滤后的项重新推入到列表里面 :
>
> ```
> lst = LRANGE lst 0 -1 # 取出列表包含的所有元素
> DEL lst                # 删除现有的列表
> for item in lst:        # 遍历整个列表
>     if item %2 != 0:    # 将非偶数元素推入到列表里面
>         RPUSH lst item
> ```
>
> 并且为了保证这个操作的安全性 , 还要用到事务和乐观锁 , 非常麻烦 .

#### Lua脚本

为了解决以上提到的问题 , Redis2.6版本开始在服务器内部嵌入了一个Luar脚本解释器 , 使得用户可以在服务器端执行Lua脚本 .

这个功能带来的好处是 :

1. 使用脚本可以直接在服务器端执行Redis命令 , 一般的数据处理操作可以直接使用Lua语言或者Lua解释器提供的函数库来完成 , 不必再返回给客户端进行处理 . 
2. 使用脚本都是以事务的形式来执行的 , 脚本在执行过程中不会被其他工作打断 , 也不会引起任何竞争条件 , 完全可以使用Lua脚本来代替事务和乐观锁 . 
3. 所有脚本都是可以重用的 , 也就是说重复执行相同的操作时候 , 只要调用存储在服务器内部的脚本缓存就可以了 , 不用重新发送整个脚本 , 从而尽可能的节约网络资源 . 

**执行Lua脚本**

```
EVAL script numkeys key [key...] arg [arg...]
```

* script参数是要执行的Lua脚本
* numkeys是脚本要处理的数据库键的数量
* key \[key...\]参数是指定脚本要处理的数据库键 , 被传入的键可以在脚本里面通过访问KEYS数组来取得 , 比如KEYS\[1\]就是取出第一个输入的键 , KEYS\[2\]取出第二个输入的键 , 诸如此类 . 
* arg \[arg...\]参数指定了脚本要用到的参数 , 在脚本里面可以通过访问ARGV数组来获取这些参数 . 

EVAL命令显示的指定了脚本里面用到的键是为了配合Redis集群对键的检查 , 如果不这样做的话 , 在集群里使用脚本可能会出错 .

另外 , 通过显示的指定脚本用到的数据库键以及相关参数 , 而不是将数据库键和参数硬写在脚本里面 , 用户可以更方便的重用同一个脚本 .

**脚本使用示例**

```
redis> eval "return 'hello world'" 0
hello world
redis> EVAL "return 1+1" 0
2
redis> EVAL "return {KEYS[1], KEYS[2], ARGV[1], ARGV[2]}" 2 "a" "b" "c" "d"
0 a
1 b
2 c
3 d
```

** 在Lua脚本中执行Redis命令**

通过调用redis.call\(\)函数或者redis.pcall\(\)函数 , 可以直接在Lua脚本里面执行Redis命令 .

```
redis> EVAL "return redis.call('PING')" 0
PONG
redis> EVAL "return redis.call('DBSIZE')" 0
46
redis> EVAL "return 'This message is ' .. redis.call('GET', KEYS[1])" 1 t.msg
This message is hello world
```

**redis.call\(\)和redis.pcall\(\)的区别**

这两个函数都可以用来执行Redis命令 , 它们的不同之处在于 , 当被执行的脚本出错时 , redis.call\(\)会返回出错脚本的名字以及EVAL命令的错误信息 , 而redis.pcall\(\)只返回EVAL命令的错误信息 .

也就是说 , 在被执行的脚本出错时 , redis.call可以提供更详细的错误信息 , 方便进行查错 .

**使用Lua脚本重新实现ZDECRBY命令**

创建zdecrby.lua文件 :

```
local old_score = redis.call('ZSCORE', KEYS[1], ARGV[2])
local new_score = old_score - ARGV[1]
return redis.call('ZADD', KEYS[1], new_score, ARGV[2])
```

执行脚本 :

```
$ redis-cli --eval zdecrby.lua salary, 300 peter
```

这里保存脚本文件并执行脚本文件要方便一些 , 而且这里的ZDECRBY也比使用事务和乐观锁实现的ZDECRBY要简单得多 .

**使用EVALSHA来减少网络资源消耗**

任何Lua脚本 , 只要被EVAL命令执行过一次 , 就会被存储到服务器的脚本缓存里 , 用户只要通过EVALSHA命令 , 指定被缓存脚本的SHA1值 , 就可以在不发送脚本的情况下 , 再次执行脚本 :

```
EVALSHA sha1 numkeys key [key...] arg [arg...]
```

通过SHA1值来重用返回前面的例子 :

```
EVALSHA edd7b92ba54a9663eeda50789cfa182ef35847e6 1 t.msg
# 这里就是前面的
EVAL "return 'This message is ' .. redis.call('GET', KEYS[1])" 1 t.msg
```

**脚本管理命令**

```
SCRIPT EXISTS sha1 [sha1...]
```

检查sha1值所代表的脚本是否已经被加入到脚本缓存里 , 是的话返回1 , 否则返回0 .

```
SCRIPT LOAD script
```

将脚本存储到脚本缓存里面 , 等待将来EVALSHA使用 .

```
SCRIPT FLUSH
```

清除脚本缓存存储的所有脚本

```
SCRIPT KILL
```

杀死运行超时的脚本 . 如果脚本已经执行过写入操作 , 还需要执行SHUTDOWN NOSAVE命令来强制服务器不保存数据 , 以免错误的数据被保存到数据库里面 .

**函数库**

Redis在Lua环境里面载入了一些常用的函数库 , 我们可以使用这些函数库 , 直接在脚本里面处理数据 :

标准库 :

* base库:包含Lua的核心\(core\)函数,比如assert、tostring、error、type等。
* string库:包含用于处理字符串的函数,比如find、format、len、reverse等。
* table库:包含用于处理表格的函数,比如concat、insert、remove、sort等。
* math库:包含常用的数学计算函数,比如abs、sqrt、log等。
* debug库:包含调试程序所需的函数,比如sethook、gethook等。

外部库 :

* struct库:在C语言的结构和Lua语言的值之间进行转换。
* cjson库:将Lua值转换为JSON对象,或者将JSON对象转换为Lua值。
* cmsgpack库:将Lua值编码为MessagePack格式,或者从MessagePack格式里面解码出Lua值。

还有一个用于计算sha1值的外部函数redis.sha1hex : 

```
redis> eval "return redis.sha1hex('aaaa')" 0
70c881d4a26984ddce795f6f71817c9cf4480e79
```



