# 事务功能

#### 事务

Redis的事务功能允许用户将多个命令包裹起来 , 然后一次性的按顺序执行被包裹的所有命令 . 在事务执行的过程中 , 服务器不会中断事务而去执行其他命令请求 , 只有在事务包裹的所有命令都被执行完毕之后 , 服务器才会去处理其他命令请求 .

假如setex命令不存在于Redis中 , 并且set命令也不支持EX seconds参数 , 使用set来实现setex命令的话 : 

```
def SETEX(key, seconds, value):
    SET key value
    EXPIRE key seconds
```

在一般情况下 , 这个自制的SETEX命令可以达到设置键值对并设置生产时间的效果 , 但是它存在一个缺陷 : 如果服务器在成功的执行SET命令并保存数据之后崩溃了 , 那么EXPIRE命令将没办法执行 . 这时虽然我们已经设置了键 , 但并没有为键设置过期时间 , 如果我们没有发觉的话 , 那么这个本来应该定期被删除的键就会一直留在数据库里面占用着内存 , 甚至造成之后的程序出错 . 

#### 事务命令

为了避免前面提到的那种情况 , 就需要用到Redis的事务功能了, 通过事务 , 让Redis一次性的执行多个命令 , 并且确保事务中的命令要么全部都执行 , 要么就一个都不执行 . 

| 命令 | 作用 |
| :--- | :--- |
| MULTI | 开始一个新的事务 |
| DISCARD | 放弃事务 |
| EXEC | 执行事务中的所有命令 |

**开始事务**

**MULTI**

开始一个事务 . 执行这个命令后 , 客户端发送的所有针对数据库或者数据库键的命令都不会被立即执行 , 而是被放入到一个事务队列里面 , 并返回QUEUED表示命令已入队 . 

```
redis> MULTI # 开始一个事务
OK
redis> SET msg "hello world" # 将这个 SET 命令放入事务队列
QUEUED
redis> EXPIRE msg 10086 
QUEUED
```

复杂度为O\(1\)

**放弃事务**

**DISCARD**

取消事务 , 放弃执行事务队列中的所有命令 , 复杂度为O\(1\) . 

```
redis> MULTI
OK
redis> SET msg "hello world" # 将这个 SET 命令放入事务队列
QUEUED
redis> EXPIRE msg 10086 
QUEUED
redis> DISCARD # 事务已经被取消
OK
```

**执行事务**

**EXEC**

按照命令被入队的事务队列中的顺序 , 执行事务队列中的所有命令 . 

命令复杂度是队列中的所有命令的复杂度之和 . 

命令的返回值是一个列表 , 列表里包含了事务队列中所有被执行命令的返回值 . 

```
redis> MULTI
OK
redis> SET msg "hello world"
QUEUED
redis> EXPIRE msg 10086
QUEUED
redis> EXEC
1) OK # SET 命令的返回值
2) (integer) 1 
```

#### **使用事务保证操作的安全性**

前面的例子中 , 使用自定义的SETEX会带有安全缺陷 , 因为无法确保所有命令都执行到 , 使用事务可以保证操作的安全性 , 要么命令都执行 , 要么都不执行 : 

```
def SETEX(key, seconds, value):
MULTI
SET key value EXPIRE key seconds EXEC
```

#### 流水线和事务的却别

| 功能 | 性质 |
| :--- | :--- |
| 流水线 | 确保多条命令会被一起发送 |
| 事务 | 确保多条命令会被一起执行 |

---

### 乐观锁

#### 使用乐观锁来保证数据的正确性

通过使用MULTI和EXEC , 我们可以将多条命令放到一个事务里面执行 , 确保事务里面的命令要么全部都被执行 , 要么就一个都不执行 , 从而防止数据出错 . 但是有时候只使用事务还是无法保证数据的正确性 , 这时候就需要使用Redis提供的乐观锁功能\(Optimistic Locking\) . 

举个例子 , 之前在有序集合中介绍过 , Redis只提供了对元素分值进行自增操作的zincrby命令 , 并没有提供与之相对应的自减操作 , 实现改命令 , 以python的形式 : 

```
def ZDECRBY(key, decrment, member):
    # 取得元素当前的分值
    old_score = ZSCORE key member
    # 使用当前分值减去指定的减量,得出新的分值
    new_score = old_score - decrment
    # 为元素设置新分值,覆盖现有的分值
    ZADD key new_score member
```

**竞争条件问题**

前面实现的ZDECRBY实现了减少元素分值的效果 , 但是实现包含了一个竞争条件 , 当多个客户端同时对同一个元素调用ZDECRBY时候 , 这个竞争条件可能就会出现 . 

举个例子 , 现在abc有序集合中的tom元素的分值是4000 , 如果两个客户端同时对其进行ZDCRBY操作 :

```
ZDECRBY abc 500 tom
ZDECRBY abc 300 tom
```

正确的情况是 , tom元素会扣除500和300 , 分值变为3200 . 这里会出现的问题是 , 当客户端B执行ZADD时 , 它不知道tom元素当前的分值已经出现了变化 , 导致它计算出的新分值3700已经过期了 , 而是继续一味地进行设置 , 使得计算出现了错误 . 

#### **WATCH命令**

为了消除ZDECRBY实现中的竞争条件 , 我们需要用到Redis提供的WATCH命令 , 这个命令需要在开始一个事务之前执行 , 它接受任意多个键作为参数, 并对输入的键进行监视 : 

```
WATCH key [key...]
```

如果被监视的键在事务提交之前\(也即是EXEC命令执行之前\) , 已经被其他客户端抢先修改了 , 那么服务器将拒绝执行客户端提交的事务 , 并返回nil作为事务的回复 : 

```
redis> WATCH msg # 监视键 msg 
OK
redis> MULTI
OK
redis> SET msg "hello world"
QUEUED
redis> EXEC # 在这个事务之前,已经有其他客户端对键 msg 进行了修改 
(nil)
```

**使用WATCH来方式竞争条件**

修改前面的例子 : 

```
def ZDECRBY(key, decrment, member):
    # 监视输入的有序集合
    WATCH key
    # 取得元素当前的分值
    old_score = ZSCORE key member
    # 使用当前分值减去指定的减量,得出新的分值 
    new_score = old_score - decrment
    # 使用事务包裹 ZADD 命令
    # 确保 ZADD 命令只会在有序集合没有被修改的情况下执行
    MULTI
    ZADD key new_score member 
    # 为元素设置新分值,覆盖现有的分值 
    EXEC
```

现在这个心的ZDCRBY执行时 , 就会出现两种情况 : 

* 执行期间 , 有序集合无变化 , ZADD正常更新元素的值
* 执行期间 , 有序集合发生变化 , 服务器阻止事务执行 , 使得ZADD无法执行 . 有效避免元素分值被设置为错误的值 . 

| 时间 | 客户端A | 客户端B |
| :--- | :--- | :--- |
| T1 | WATCH salary | WATCH salary |
| T2 | 执行ZSCORE salary peter,获得分值4000,计算4000 - 500,得出3500。 | 执行ZSCORE salary peter,获得分值4000,计算4000 - 300,得出3700。 |
| T3 | 执行MULTI、ZADD salary 3500 peter和EXEC命令, 事务执行成功。 |  |
| T4 |  | 执行MULTI、ZADD salary 3700 peter和EXEC命令, 因为键salary已经被修 改,事务执行失败。 |

#### **和乐观锁相关的其他命令**

* UNWATCH - 取消对所有键的监视 , 复杂度O\(1\)
* DISCARD - 放弃执行事务 , 并且取消对所有键的监视\(相当于执行了UNWATCH\) , 复杂度O\(1\)

```
redis> WATCH msg name fruits # 监视三个键 
OK
redis> UNWATCH # 取消对上面三个键的监视 
OK
```



