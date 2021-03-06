# 数据库管理

**数据库**

Redis是一个键值对数据库服务器 , 每当我们调用命令 , 创建一个键值对的时候 , 这个键值对都会被放置到Redis的某个数据库里面 .

**数据命令**

和Redis提供了各式各样的命令来操作不同的数据库结构一样 , Redis也提供了一些命令来操作数据库以及数据包含的键值对 . 比如检查一个键的类型 , 删除一个键 , 对键进行改名 , 遍历数据库中的所有键 , 删除数据库中的所有键等等 .

---

#### 处理数据库中的单个键

查看键的类型 , 删除键 , 检查键是否存在 , 修改键的名字 .

**查看键的类型**

```
TYPE key
```

返回键key存储的值的类型 , 复杂度为O\(1\) .

返回值

* none - 键不存在
* string - 字符串或HyperLogLog\(HLL是二进制值\)
* hash - 散列
* list - 列表
* set - 集合
* zset - 有序结合

**删除键**

```
DEL key [key ...]
```

删除给定的任意多个键 , 不存在的键会被忽略 , 命令返回被成功删除的键的数量 .

复杂度为O\(N\) , N为被删除键的数量 .

**检查键是否存在**

```
EXISTS key
```

检查给定的键是否存在于数据库 , 存在返回1 , 不存在返回0 .

复杂度O\(1\) .

**修改键的名字**

Redis提供了RENAME和RENAMENX两个命令 , 用于修改键的名字 :

| 命令 | 作用 | 返回值 | 复杂度 |
| :--- | :--- | :--- | :--- |
| RENAME key newkey | 将键的名字从key改为newkey . 如果newkey已经存在 , 那么覆盖它 . | 键key不存在 , 返回错误 . 修改成功时返回OK . | O\(1\) |
| RENAMENX key newkey | 如果newkey不存在 , 那么将键key改名为newkey . 如果newkey已经存在 , 那么不做动作 . | 修改成功返回1 , 修改失败返回0 . | O\(1\) |

---

#### 对键的值进行排序

SORT命令以及它的各个参数和选项 .

**SORT命令**

在Redis中 , 只有列表和有序集合默认是以有序的方式来存储值的 :

* 列表按照值被推入的顺序来存储值
* 有序集合按照元素的分值来排列元素

通过调用SORT命令 , 可以对列表 , 集合以及有序集合进行排序 , 对于原本已经有序的列表和有序集合来说 , SORT命令可以以其他方式来排列他们的值 .

```
SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC|DESC] [ALPHA] [STORE destination]
```

SORT命令提供的可选选项以及可选参数非常多 , 它们可以修改SORT命令在排序时的行为 .

命令的复杂度为O\(N+M\*log\(M\)\) , N为被排序键包含的值数量 , M为要返回的值数量 .

**最简单的情况**

```
SORT key
```

将输入键包含的值解释为浮点数 , 然后对这些浮点数进行排序 .

例如 :

```
redis> rpush num666 9 3 6 2 1
5
redis> sort num666 # 按照值本身的大小进行排序
0 1
1 2
2 3
3 6
4 9
```

**升序排序和降序排序**

SORT命令允许用户通过给定ASC参数或DESC参数来分别指定排序顺序 :

* 默认给定ASC , 从小到大的顺序升序排序
* 给定DESC , 从大到小的顺序降序排序

不给定 , 则默认ASC排序 .

```
redis> sort num666 asc
redis> sort num666 desc
```

**对文字进行排序**

默认情况下 , SORT命令将键包含的值解释为浮点数 , 然后对浮点数进行排序 . 这里通过 , ALPHA参数 , 让SORT命令基于字典序对文字进行排序 .

```
redis> sadd sanguo "曹操" "刘备" "孙权"
3
redis> smembers sanguo
0 孙权
1 刘备
2 曹操
redis> sort sanguo # 直接使用排序会报错提示这些值不能解释为浮点数
ERR One or more scores can't be converted into double
redis> sort sanguo alpha
0 刘备
1 孙权
2 曹操
```

> 字典序\(lexicographical ordering\) - 是一种对于随机变量形成序列的排序方法 . 其方法是 , 按照字母顺序 , 或者数字小大顺序 , 由小到大的形成序列 .
>
> 比如说有一个随机变量X包含{1 2 3}三个数值 .
>
> 其字典排序就是{} {1} {1 2} {1 2 3} {2} {2 3} {3}

**文字排序特别适用于有序集合**

在一般情况下 , 有序集合里面的元素会根据分值来进行排序 , 但是通过执行带有ALPHA参数的SORT命令 , 可以根绝元素本身来进行排序 :

```
redis> zadd scores 3 "peter" 6 "jack" 4 "tommy"
3
redis> zrange scores 0 -1 # 根据分值,对元素进行排序
0 peter
1 tommy
2 jack
redis> sort scores alpha # 根据元素本身进行排序
0 jack
1 peter
2 tommy
```

**基于外部键的值进行排序**

在默认情况下 , SORT命令在进行排序的时候 , 会使用被排序键本身包含的值来作为权重 . 但是通过执行BY pattern选项 , 就可以让SORT命令使用其他键的值来作为权重 , 对被排序键的值进行排序 .

```
redis> sort scores alpha # 基于三个元素本身进行排序
0 jack
1 peter
2 tommy
redis> mset peter-score 4 jack-score 5 tommy-score 3
OK # 设置三个字符串键作为权重
redis> sort scores by *-score # sort命令首先获取三个元素
0 tommy                       # 然后将它们代入到*-score模式里面
1 peter                       # 得出三个-score键名
2 jack                        # 然后获取这三个键的值作为排序的权重
```

**获取外部键的值作为返回值**

在默认情况下 , SORT命令会返回被排序键的值为返回值 , 但是通过给定GET pattern选项 , 可以让SORT命令返回其他键的值来作为命令的返回值 .

```
redis> sort scores alpha # 基于三个元素本身进行排序
0 jack
1 peter
2 tommy
redis> mset peter-score 4 jack-score 5 tommy-score 3
OK # 设置三个字符串键作为权重
redis> sort scores alpha get *-score # 对集合进行排序
0 5                                  # 然后将这三个元素代入到*-name
1 4                                  # 得出三个键
2 3                                  # 然后返回这些键的值作为排序结果
```

**获取多个外部键的值**

调用一次SORT命令可以给定多个GET选项 .

```
redis> mset peter-score 4 jack-score 5 tommy-score 3
redis> mset peter-name "peter" jack-name "jack" tommy-name "tommy"
redis> sort scores alpha get # get *-score get *-name
0 5
1 jack
2 jack
3 4
4 peter
5 peter
6 3
7 tommy
8 tommy
# 当给定get #时,命令会返回被排序的值本身
```

**指定返回结果的数量**

在默认情况下 , SORT命令总是返回被排序键的所有值作为结果 , 但是通过指定limit offset count选项 , 可以让命令在返回结果之前先跳过offset个值 , 然后只返回count个值 .

```
redis> rpush num-test 8 3 7 4 6 5 1 0 2 9
10
redis> sort num-test limit 0 5
0 0
1 1
2 2
3 3
4 4
redis> sort num-test limit 5 5 # 跳过开头的5个值,返回紧接着的5个值
0 5
1 6 
2 7
3 8
4 9
```

**存储排序结果**

在默认情况下 , sort命令只会向客户端返回排序结果 , 但并不存储排序结果 . 通过指定选项 store destkey , 将排序结果存储到destkey里 .

```
redis> lrange num-test 0 -1
redis> sort num-test store sorted-num-test
3
redis> lrange sorted-num-test 0 -1
```

**使用多个选项和参数**

sort命令允许用户同时使用前面提到的所有选项和参数 , 通过适当的组合使用不同的选项和参数 , 可以将sort命令打造成一个强大的排序工具以及数据获取工具 :

```
redis> sort team-member-ids by *-KPI get # get *-name get *-KPI
# 通过KPI值,对团队中各个成员的ID进行排序
# 然后根据排序结果,依次获取成员的ID,名字和KPI值
redis> sort names alpha desc get # get *-id get *-id get *-name limit 0 5 store profiles
# 对names键的值进行文字形式的降序排序
# 根据排序后的前五个值,获取值本身,以及其对应的id和名字
# 然后将获取到的这些信息存储到的profiles键里面
```



