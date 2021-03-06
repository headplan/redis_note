# 集合\(set\)

Redis的集合以无序的方式存储多个各不相同的元素.

用户可以快速的向集合添加元素,或者从集合里面删除元素,也可以对多个集合进行集合运算操作,比如计算并集,交集和差集.

### 元素操作

**添加元素**

SADD key element \[element ...\]

将一个或多个元素添加到给定的集合里面,已经存在的自动忽略,返回新添加到集合的元素的数量.

复杂度为O\(N\),N为成功添加的元素数量.

```
SADD friends "peter" "tom"
(int) 2
SADD friends "jack" "tom"
(int) 1
```

**移除元素**

SREM key element \[element ...\]

移除集合中的一个或者多个元素,不存在于集合中的元素自动忽略,命令返回存在并且被移除的元素数量.

复杂度为O\(N\),N为成功添加的元素数量.

```
SREM friends "peter" "tom" "abc"
(int) 2
```

**检查给定元素是否存在于集合**

SISMEMBER key element

检查给定的元素是否存在于集合,存在的话返回1;

如果元素不存在,或者给定的键不存在,那么返回0;

复杂度为O\(1\)

```
SISMEMBER friends "peter"
(int) 1
SISMEMBER friends "haha"
(int) 0
```

**返回集合的大小**

SCARD key

返回集合包含的元素数量,即是集合的基数.

复杂度为O\(1\),因为Redis会存储集合的长度

```
SCARD friends
(int) 6
```

**返回集合包含的所有元素**

SMEMBERS key

返回集合包含的所有元素

复杂度为O\(N\),N为集合的大小,当基数比较大时,可能会造成服务器阻塞,Redis还有更好的方式来迭代集合中的元素.

```
SMEMBERS friends
1)"jack"
2)"peter"
...
```

**集合的无序性质**

对于相同的一集元素,同一个集合命令可能会返回不同的结果\(里面的元素顺序不同\).

结论:不要使用集合来存储有序的数据

* 如果要存储有序并且有重复的值,可以使用列表;
* 如果想要存储有序且无重复的值,可以使用有序集合;

**从集合里面随机的弹出一个元素**

SPOP key

随机的从集合中移除并返回一个元素,复杂度为O\(1\)

```
SADD friends "peter" "jack" "tom" "john" "may" "ben"
(int) 6
SPOP friends
"may"
```

**从集合里面随机的返回元素**

SRANDMEMBER key \[count\]

如果没有给定可选的count参数,那么命令随机的返回集合中的一个元素

如果给定了count参数,那么:

* 当count为正数,并且少于集合基数时,命令返回一个包含count个元素的数组,数组中的每个元素各不相同.**如果count大于或等于集合基数,那么命令返回整个集合**
* 当count为负数时,命令返回一个数组,数组中的元素可以能会重复出现多次,而数组的长度为count的绝对值

和SPOP不同,不会移除被返回的元素.  
命令的复杂度为O\(N\),N为被返回元素的数量.

**SRANDMEMBER的使用示例**

```
SADD friends "peter" "jack" "tom" "john" "may"
SRANDMEMBER friends # 随机的返回一个元素
SRANDMEMBER friends 3 # 随机的返回三个无重复的元素
SRANDMEMBER friends -3 # 随机的返回三个可能有重复的元素
```

### **集合运算操作**

**差集运算**

SDIFF key \[key ...\]

计算所有给定集合的差集,并返回结果.O\(N\)复杂度,N为所有参与差集计算的元素数量之和.

SDIFFSTORE destkey key \[key ...\]

计算所有给定集合的差集,并将结果存储到destkey.O\(N\)复杂度,N为所有参与差集计算的元素数量之和.

```
SADD number1 "123" "456" "789"
(int) 3
SADD number2 "123" "456" "999"
(int) 3
SDIFF number1 number2
"789"
```

**交集运算**

SINTER key \[key ...\]

计算所有给定集合的交集,并返回结果.O\(N\*M\)的复杂度,N为给定集合当中基数最小的集合,M为给定集合的个数.

SINTERSTORE destkey key \[key ...\]

计算所有给定集合的交集,并将结果存储到destkey.O\(N\*M\)的复杂度,N为给定集合当中基数最小的集合,M为给定集合的个数.

```
SADD number1 "123" "456" "789"
(int) 3
SADD number2 "123" "456" "888"
(int) 3
SINTER number1 number2
"123"
"456"
```

**并集运算**

SUNION key \[key ...\]

计算所有给定集合的并集,并返回结果.O\(N\)复杂度,N为所有参与计算的元素数量.

SUNIONSTORE destkey key \[key ...\]

计算所有给定集合的并集,并将结果存储到destkey.O\(N\)复杂度,N为所有参与计算的元素数量.

```
SADD number1 "123" "456" "789"
(int) 3
SADD number2 "123" "456" "888"
(int) 3
SUNION number1 number2
"123"
"456"
"789"
"888"
```



