# 散列(hash)
* 一个散列由多个域值对(field-value pair)组成,散列的域和值都可以是文字,整数,浮点数或二进制数据.
* 同一个散列里面的每个域必须是独一无二,各不相同的,而域的值则没有要求,即不同域的值可以是重复的.
* 通过命令,用户可以对散列执行
  - 设置域值对
  - 获取域的值
  - 检查域是否存在
  - 返回散列包含的所有域,所有值,所有域值对等等
![散列](Snip20160709_1.png)

## 基本操作
### 关联域值对
```
HSET key field value
```
* 在散列键key中关联给定的域值对field和value
* 如果域field之前没有关联值,那么命令返回1
* 如果域field已经有关联值,那么命令用新值覆盖旧值,并返回0
* 复杂度为O(1)
```
HSET msg "id" 10086
HSET msg "sender" "peter"
HSET msg "receiver" "jack"
```
### 获取域关联的值
```
HGET key field
```
* 返回散列键key中,域field所关联的值
* 如果域field没有关联值,那么返回nil
* 复杂度O(1)
```
HGET msg "id"
HGET msg "sender"
HGET msg "content"
```
### 仅当域不存在时,关联域值对
```
HSETNX key field value
```


