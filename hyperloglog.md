# HyperLogLog

使用常量空间计算大量元素的基数.

**应用场景:记录网站每天获得的独立IP数量**

**使用集合实现**

使用集合来存储每个访客的IP,通过集合中每个元素都各不相同的特性,来得到多个独立IP,然后通过调用SCARD命令来得出独立IP的数量.

```
ip = get_vistor_ip()
SADD '2016.7.30::unique::ip' ip
```

然后使用SCARD就可以获取当前的唯一IP数量:

SCARD 'keyname'

使用集合实现独立IP记录会带来一个问题,字符串记录一个IPv4最多需要消耗15个字节,IPv6会更多一些,这样持续下去,下号的内存会越来越多,下面是一个内存消耗表:

![](/assets/Snip20160728_1.png)

为了更好的解决类似独立IP计算这种问题,Redis在2.8.9版本添加了HyperLogLog结构.

---

### HyperLogLog介绍

HLL可以接受多个元素作为输入,并给出输入元素的基数估算值

* 基数:就是集合中不同元素的数量,例如集合中有5个元素,2个相同,另外3个相同,则基数为2
* 估算值:算法给出的基数并非精确的,但误差也会在合理的范围内.

HLL类型的优点是,即使输入的元素数量或者体积非常大,计算基数所需要的空间也总是固定,并且很小的.  
在Redis中,每个HLL键只需要12KB内存,就可以计算2^64个不同元素的基数.但是,HLL只会根据输入元素来计算基数,而不会存储输入元素的本身,意思就是不会像集合一样,返回输入的各个元素.

**添加元素**

```
PFADD key element [element ...]
```

将任意数量的元素添加到指定的HLL里面.命令可能会对HLL进行修改,以便反映新的基数估算值,如果HLL的基数估算值在命令执行之后出现了变化,那么命令返回1,否则返回0.

命令的复杂度为O\(N\),N为被添加元素的数量

**返回基数估算值**

```
PFCOUNT key [key ...]
```

当给定一个HLL时,命令返回给定HLL的基数估算值.

当给定多个HLL时,命令会先对给定的HLL进行并集计算,得出一个合并后的HLL,然后再返回合并后的基数估算值,但是这个合并后计算出来的估算值不会被存储,使用之后就会被删掉.

命令作用于单个HLL时,复杂度为O\(1\),并且具有非常低的平均常数时间;

命令作用于多个HLL时,复杂度为O\(N\),常数时间比处理单个时要大得多.

**合并多个HyperLogLog**

PFMERGE destkey sourcekey \[sourcekey ...\]

将多个HyperLogLog合并为一个HyperLogLog,合并后的HyperLogLog的基数估算值是通过对所有给定HyperLogLog进行并集计算得出的.

命令的复杂度为O\(N\),其中N为被合并的HLL数量,不过这个命令的常数复杂度比较高.

**三个命令使用示例**

```
PFADD unique::ip::counter '192.168.0.1'
(int)1
PFADD unique::ip::counter '127.0.0.1'
(int)1
PFCOUNT unique::ip::counter
(int)3

PFADD str1 "apple" "banana" "cherry"
int(1)
PFCOUNT str1
int(3)
PFADD str2 "apple" "cherry" "durian" "mongo"
int(1)
PFCOUNT str2
int(4)
PFMERGE str1&2 str1 str2
OK
PFCOUNT str1&2
int(5)
```

**唯一计数器的API及其实现**

![](/assets/Snip20160728_4.png)

```
# 创建一个ip地址唯一计数器
ip_counter = UniqueCounter(client, 'unique::ip::counter')
# 添加一些ip
ip_counter.include('192.168.0.1')
ip_counter.include('8.8.8.8')
ip_counter.include('255.255.255.255')
# 查看计数器当前的值
ip_counter.result()
# 返回3
```

代码实现查看:unique\_counter.py

**HyperLogLog实现独立IP统计需要消耗的内存**

![](/assets/Snip20160728_2.png)

