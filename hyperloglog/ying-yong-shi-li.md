# 应用实例

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



