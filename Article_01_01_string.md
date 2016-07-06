# 字符串
Redis中最简单的数据结构,存储文字,数字或者二进制数据.这几种类型分别设置了相应的操作命令,可以针对不同的值做不同的处理.
![string](Snip20160706_3.png)
三个字符串分别存储了文字(msg),数字(number)以及二进制数据(bits).
## 基本操作
### 为字符串键设置值
```
SET key value
```
* 将字符串键key的值设置为value,命令返回OK表示设置成功.
* 如果字符串键key已经存在,那么用新值覆盖原来的旧值.
* 复杂度:O(1)
  ```
  SET msg "hello world"
  OK
  SET msg "goodbye" # 覆盖上面的"hello world"
  OK
  ```
```
SET key value [NX|XX]
```
