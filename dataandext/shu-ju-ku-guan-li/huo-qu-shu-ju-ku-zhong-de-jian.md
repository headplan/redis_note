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



