# 一、Redis介绍

## 1.Redis是什么

![image-20200514155504824](D:\notes\Redis\images\redis是什么.png)

## 2.Redis的应用

![image-20200514160416318](D:\notes\Redis\images\redis应用.png)

## 2.下载与使用

[Redis下载路径](https://github.com/microsoftarchive/redis/releases)

启动Redis（端口6379 PID随机）：

### （1）启动服务：

![image-20200514161958026](D:\notes\Redis\images\redis使用.png)

### （2）客户端：

![image-20200514162131694](D:\notes\Redis\images\redis使用2.png)

### （3）本地CMD使用：

定位到指定目录，输入redis-cli

![image-20200514162536242](D:\notes\Redis\images\redis使用3.png)

# 二、Redis基本操作

## 0.Redis命令中心

[Redis命令中心](http://www.redis.cn/commands.html)

## 1.功能性命令

（1）信息添加

~~~
功能：设置一个key、value数据
命令：set key value
例如：set name itheima

功能：设置多个key、value数据
命令：mset key value [key value] [...]
例如：mset name zhangsan age 23 gender 0

~~~

(2)信息查询

~~~
功能：通过key查询对应的value，如果不存在，返回空（nil）
命令：get key
例如：get name

功能：查询多个key、value数据
命令：mget key [key] [...]
例如：mget name age gender

功能：获取字符串长度
命令：strlen key
例如：strlen name
~~~

（3）删除信息

~~~
功能：删除一条信息
命令：del key
例如：del name
成功：返回（integer）1		失败：返回（integer）0
~~~







![image-20200514163629434](D:\notes\Redis\images\redis功能性命令1.png)





## 2.清楚屏幕信息

~~~
clear
~~~



## 3.帮助信息查阅



## 4.退出指令

~~~
quit
exit
<ESC>
~~~

# 三、数据存储类型介绍

## 1.string

![image-20200514191926465](D:\notes\Redis\images\string类型1.png)

------

### （1）string类型数据的基本操作

~~~
添加/修改多个数据：
命令：mset key value [key value] [...]
例如：mset name zhangsan age 23 gender 0

获取多个数据：
命令：mget key [key] [...]
例如：mget name age gender

获取字符串长度：
命令：strlen key
例如：strlen name

追加信息到原始信息后部（原始信息存在就追加，否则新建）：
命令：append key value
例如：append name 123
返回：字符串长度
~~~

### （2）string数据类型的扩展操作

#### 第一种业务场景：

![image-20200514205636160](D:\notes\Redis\images\string类型2.png)

~~~
*********解决方案：
1.设置数值数据增加指定范围的值：
incr key		执行命令后value（key）自增1
incrby key increment		执行命令后value（key）自增increment
incrbyfloat key increment		执行命令后value（key）自增increment（可以小数）
返回值：value（key）自增后的值

2.设置数值数据减少指定范围的值：
decr key		执行命令后value（key）自减1
decrby key increment		执行命令后value（key）自减increment
返回值：value（key）自减后的值
~~~

#### 第二种业务场景：

![image-20200514205708301](D:\notes\Redis\images\string类型3.png)

~~~
********解决方案：
3.设置数据具有指定的生命周期：
命令：setex key seconds value			(单位：s)
例如：setex num 10 857
命令：psetex key milliseconds value	(单位：ms)
例如：psetex num 10000 666
~~~

### （3）应用场景：

![image-20200514210413474](D:\notes\Redis\images\string类型4.png)

------

![image-20200514210504432](D:\notes\Redis\images\string类型5.png)

~~~
*****************解决方案：
第一种方案：
设置用户信息：
例如：
设置粉丝数：set user:id:00789:fans 123
设置微博数：set user:id:00789:blogs 456
增加粉丝数量：incr user:id:00789:fans

第二种方案（json格式存储）：
设置粉丝数和微博数：set user:id:00789 {id:00789,fans:456,blogs:789}
*不方便修改
~~~

第一种方案：

![image-20200514211632137](D:\notes\Redis\images\string类型6.png)

第二种方案：

![image-20200514211700522](D:\notes\Redis\images\string类型7.png)

### （4）key的设置约束：

![image-20200514212050164](D:\notes\Redis\images\string类型8.png)

### （5）注意事项：

![image-20200514215548946](D:\notes\Redis\images\string类型9.png)

------

## 2.hash

### （1）简介：

![image-20200514212534739](D:\notes\Redis\images\hash类型1.png)

------

![image-20200514212553791](D:\notes\Redis\images\hash类型2.png)

### （2）hash数据类型的基本操作：

~~~
添加/修改数据：
命令：hset key field value
例如：
hset user name zhangsan
hset user age 24
hset user gender 0

获取数据：
命令：hget key field
例如：hget user name
功能：获取value（user）中的一个字段
命令：hgetall key
例如：hgetall user
功能：获取value（user）

删除数据：
命令：hdel key field1 [field2]
例如：hdel user name age

添加/修改多个数据：
命令：hmset key field1 value1 [field2 value2]
功能：hmset user name zhangsan age 24
获取多个数据：
命令：hmget key field1 [field2]
例如：hmget user name age
获取哈希表中字段的数量：
命令：hlen key
例如：hlen user
获取哈希表中是否存在指定的字段：
命令：hexists key field
例如：hexists user name
~~~

![image-20200514213327119](D:\notes\Redis\images\hash类型3.png)



### （3）hash类型数据扩展操作：

~~~
获取哈希表中所有的字段名或字段值：
字段名：
命令：hkeys key
例如：hkeys user
字段值：
命令：hvals key
例如：hvals user

设置指定字段的数值数据增加指定范围的值：
命令：hincrby key field increment
例如：hincrby user age 2
命令：hincrbyfloat key field increment
例如：hincrbyfloat user age 1.5
~~~

### （4）注意事项：

![image-20200514215426839](D:\notes\Redis\images\hash类型4.png)

### （5）应用场景：

#### 第一个应用场景：

![image-20200514215952677](D:\notes\Redis\images\hash类型5.png)

![image-20200514220014937](D:\notes\Redis\images\hash类型6.png)

~~~
*user表
id	商品名称	商品数量
001		g01		100
001		g02		200
002		g01		1
002		g04		7
002		g05		100
命令：
hmset user:id:001 g01 100 g02 200
hmset user:id:002 g02 1 g04 7 g05 100

用户001 购买商品g03 数量5
命令：
hset user:id:001 g03 5

用户001 删除商品g01
命令：
hdel user:id:001 g01

用户001 商品g03 数量+1
命令：
hincrby user:id:001 g03 1

用户001 浏览商品
命令：
hgetall user:id:001
~~~

![image-20200514231217588](D:\notes\Redis\images\hash类型7.png)

~~~
将商品保存成两个field，一个数量nums，一个详细信息info
命令：
hmset user:id:003 g01:nums 100 g01:info {...}

不同用户购买同一件商品，商品信息info是一样的，可以将商品信息info拿出来单独弄成一个独立hash

每次用户购买商品时，都会在上面独立hash中中创建一条新的商品信息记录，没有必要，相同的商品只需要存储一次就好了。可以使用hsetnx命令，它会在每次存储时判断hash中是否存在这条商品信息的记录，存在则不存，不存在才存。

例如：
用户003 已经存在商品g01 不会再去存储g01信息
hsetnx user:id:003 g01:nums 50
用户003 不存在商品g02 存储g02信息
hsetnx user:id:003 g02:nums 10
~~~

#### 第二个场景：

![image-20200515004304416](D:\notes\Redis\images\hash类型8.png)

![image-20200515004356399](D:\notes\Redis\images\hash类型9.png)

~~~
*********商家p01
充值卡		c30		c50		c100
数量		1000	1000	1000
hmset p01 c30 1000 c50 1000 c100 1000
顾客过来抢购充值卡c30一张：
hincrby p01 c30 -1
~~~



## 3.list



## 4.set



## 5.sorted_set



## 6.书数据类型实践案例