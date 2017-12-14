# 处理数据库本身

#### 获取数据库的大小

**DBSIZE**

返回数据库目前包含的键值对数量 . 复杂度O\(1\) .

```
redis> dbsize
47
```

#### 清空当前数据库

**FLUSHDB**

删除当前数据库包含的所有键值对 , 命令总是返回OK , 表示删除成功 . 

```
redis> dbsize
47
redis> flushdb
OK
redis> dbsize
0
```

复杂度为O\(N\) , N为被删除键值对的数量 . 

