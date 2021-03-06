# 获取数据库中的键

#### 随机的获取某个键

**RANDOMKEY**

从当前数据库中随机的返回一个键 , 被返回的键不会被删除 .

如果数据库不包含任何键值对\(为空\) , 那么命令返回nli . 复杂度为O\(1\) .

```
redis> RANDOMKEY
message2
redis> RANDOMKEY
fruits-8-13&14-pop
```

#### 返回数据库中与给定模式相匹配的键

**KEYS pattern**

返回当前数据库中 , 所有匹配给定模式parrern的键 .

pattern参数的值可以是任何glob风格的匹配符 , 以下是一些模式示例 :

| 模式 | 匹配的键 |
| :--- | :--- |
| KEYS \* | 数据库中的所有键 |
| KEYS h?llo | hello,hallo,hxllo等键 |
| KEYS h\*llo | hello,hllo,heeeello等键 |
| KEYS h\[ae\]llo | 只匹配hello和hallo键 |

命令复杂度为O\(N\) , N为数据库包含的键值对数量 .

#### 渐进的遍历整个数据库

因为KEYS命令会一次性的遍历整个数据库来获取所有与给定模式匹配的键 , 所以随着数据库包含的键值对越来越多 , 这个命令的执行速度也会越来越慢 , 而对一个非常大的数据库\(比如几千万,甚至几亿键值对\)执行keys命令 , 将导致服务被阻塞一段时间 .

为了解决这个问题 , Redis从2.8.0版本开始提供scan命令 , 已渐进的方式 , 分多次遍历整个数据库 , 并返回匹配给定模式的键 .

**SCAN命令用法**

```
SCAN cursor [MATCH pattern] [COUNT number]
```

cursor参数是遍历时使用的游标 , 在开始一次新的遍历时 , 用户需要将cursor设置为0 , 之后每次调用scan命令都会返回一个新的游标值 , 用户再次调用sacn命令时需要输入这个游标值来继续上次的遍历 , 当命令返回的游标为0时 , 遍历结束 .

MATCH pattern选项用于指定要匹配的模式 , 类似KEYS命令的pattern参数 .

COUNT number选项指定这次遍历最多要返回多少键 , 命令实际返回的键数量可能比这个值多或少 . 有时候 , 命令甚至可能会一个键也不返回 , 但只要命令返回的游标值不为0 , 就说明遍历未结束 . number参数的默认值为10 .

scan命令使用的算法可以保证 , 遍历从开始到结束的期间 , 一直存在于数据库里面的键肯定会被遍历到 , 但是中途删除的或者中途添加的键是否会被遍历是不确定的 , 可能会也可以不会 . 另外 , 命令可能会返回一个键多次 , 要求无重复结果的话 , 需要自己在客户端里进行过滤 .

```
redis> scan 0 match * count 20 # 游标从0开始
redis> scan 22 match * count 20 # 根据前一次返回的游标22
redis> scan 35 match * count 20 # 根绝前一次返回的游标35,本次返回游标为0,也表示遍历结束
```

#### 对比KEYS和SCAN

|  | KEYS | SCAN |
| :--- | :--- | :--- |
| 处理方式 | 一次性遍历整个数据库并返回所有结果 | 每次遍历数据库中的一部分键 , 并返回一部分结果 . |
| 是否会阻塞服务器 | 可能会 | 不会 |
| 是否会出现重复值 | 不会 | 可能会 |
| 复杂度 | O\(N\) , N为数据库包含的键值对数量 . | 每次执行的复杂度为O\(1\) , 遍历整个数据库的总复杂度为O\(N\) , N为数据库包含的键值对数量 . |

#### 其他渐进遍历命令

```
SSCAN key cursor [MATCH pattern] [COUNT count]
```

代替可能会阻塞服务器的SMEMBERS命令 , 遍历集合包含的各个元素 . 

```
HSCAN key cursor [MATCH pattern] [COUNT count]
```

代替可能会阻塞服务器的HGETALL命令 , 遍历散列包含的各个键值对 . 

```
ZSCAN key cursor [MATCH pattern] [COUNT count]
```

代替可能会阻塞服务器的ZRANGE命令 , 遍历有序集合包含的各个元素 . 

> 这些命令的MATCH选项和COUNT选项的意思和SACAN命令的一样

