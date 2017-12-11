# 服务器配置

通过调整服务器配置 , 来适应需求 .

#### 配置选项

Redis服务器提供了一些配置选项\(configuration option\) , 通过修改这些选项的值,可以改变选项对应功能的行为 .

例如 , 前面介绍SELECT命令时曾经说过 , Redis服务器默认会创建0号至15号共十六个数据库以供用户使用 . 但Redis服务器的数据库数量并不是一成不变的 , Redis提供了databases选项 , 它的默认值为16 , 通过修改这个选项的值,我们可以让服务器创建指定数量的数据库,比如5个、10个、32个、100个,诸如此类 .

再举一个例子 , 在介绍Lua脚本的时候 , 我们曾说过 , 如果一个脚本的运行时间过长 , 用户可以使用SCRIPT KILL命令来强制停止脚本,其中lua-time-limit选项的值就决定了脚本可以不被打扰地运行的**最大毫秒数 **, 如果这个选项的值是5000的话 , 那么只有在脚本运行时间超过5000毫秒之后 , 服务器才会开始接受SCRIPT KILL命令 , 允许用户终结正在运行的脚本 . 如果有需要的话 , 用户也可以把这个选项的值调小或者调大 .

#### 修改和获取配置选项

Redis提供了三种方法来修改配置选项的值 :

**第一种方法**

在启动服务器时 , 通过给定参数的方式来为配置选项设置值 , 格式为 :

```
redis-server --<option1> <value1> --<option1> <value1> --<option1> <value1> ...
```

例如 , 前面提到的database选项 , 启动时创建32个数据库 :

```
redis-server --databases 32
```

启动时创建100个数据库 , 同时指定端口为10086 :

```
redis-server --databases 100 --port 10086
```

**第二种方法**

将要修改的配置选项以及选项的值记录到一个配置文件里面 , 并在启动服务器时 , 让服务器载入该配置文件 :

```
redis-server <path-to-config-file>
```

例如 , 创建一个redis.conf文件 :

```
databases 128
port 10086
```

启动服务器时让服务器载入该文件 :

```
redis-server redis.conf
```

**第三种方法**

以上两种方法都只能在服务器启动时修改配置选项的值 . 第三种方法 , 使用CONFIG命令 , 用户可以在服务器运行时动态修改选项的值 , 也可以通过命令获取选项当前的值 .

获取当前选项的值 :

```
CONFIG GET <option>
```

例如 :

```
redis> config get databases
    0 databases
    1 16
```

修改配置选项的值 :

```
CONFIG SET <option> <value>
```

例如 :

```
redis> CONFIG SET lua-time-limit 3000
OK
# 检查设置是否成功
redis> CONFIG SET lua-time-limit
    0 lua-time-limit
    1 3000
```

需要注意的是 , 并不是所有配置选项都可以在服务器运行时动态地设置的 , 有一些配置选项必须在服务器启动时才能设置 . 因为创建数据库的工作是在服务器启动时进行的 , 所以数据库的数量必须在启动服务器时指定 , 在服务器运行的过程中,尝试使用CONFIG SET去修改数据库的数量是不可行的 , 例如 :

```
redis> CONFIG SET databases 100

(error) ERR Unsupported CONFIG parameter: databases
```

还有配置端口等 , 也是会报错的 . 还有 , 因为是动态设置 , 所以CONFIG SET设置的选项值只会在服务器运行的过程中生效 , 一旦服务器重启 , CONFIG SET设置的选项值就会失效了 , 恢复为默认值 .

**写入配置文件**

如果服务器启动时 , 是以第二种方式 , 载入了配置文件启动的 , 并且在服务器运行的过程中使用CONFIG SET修改了配置选项的值 , 那 么执行CONFIG REWRITE命令可以将被修改的配置选项以及它的值写入到配置文件里面 . 这里不再举例了 . 

---

#### 一些基本的配置项

| 选项 | 作用 | 默认值 | 可在线修改? |
| :--- | :--- | :--- | :--- |
| port &lt;num&gt; | 服务器的监听端口号。 | 6379 | 否 |
| timeout &lt;seconds&gt; | 在客户端处于空闲状态多久之后,服务器才会自动断开它。 | 0\(不主动断开\) | 是 |
| loglevel &lt;level&gt; | 服务器记录日志的级别。 | notice | 是 |
| databases &lt;num&gt; | 数据库的数量。 | 16 | 否 |
| requirepass &lt;pswd&gt; | 客户端连接服务器的密码。 | \(空密码\) | 是 |
| maxmemory &lt;bytes&gt; | 可用的最大内存数量。 | \(不限制\) | 是 |
| lua-time-limit &lt;ms&gt; | Lua脚本正常运行的最大时间 | 5000 | 是 |



