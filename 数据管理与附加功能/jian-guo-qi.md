# 键过期功能

让Redis在指定的时间自动删除特定的键 . 

#### 键盘过期功能的相关命令

* 设置生存时间 - EXPIRE命令和PEXPIRE命令
* 设置过期时间 - EXPIREAT命令和PEXPIREAT命令
* 查看剩余生存时间 - TTL命令和PTTL命令
* 删除生存时间或过期时间 - PERSIST命令

#### 设置生存时间

```
EXPIRE key seconds # 将键 key 的生存时间设置为指定的秒数,复杂度O(1)
PEXPIRE key milliseconds # 将键 key 的生存时间设置为指定的毫秒数,复杂度O(1)
```

如果给定的键不存在,那么EXPIRE和PEXPIRE将返回0,表示设置失败;如果命令返回1,那么表 示设置成功 . 

当一个键的生存时间被减少至低于0时,Redis就会自动将这个键删除掉 . 

#### 设置过期时间





