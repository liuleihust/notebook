## 配置 Mysql 主从分离

**Master**  主机

>```
>log-bin=mysql-bin //启用二进制日志文件
>server-id=130 服务器唯一ID 
>主机授权 从机复制
>```

**slave 从机**

>```
>server-id=132  服务器id，唯一
>relay-log=slave-relay-bin
>relay-log-index=slave-relay-bin.index
>read_only=1
>```

```
从机
CHANGE MASTER TO 
MASTER_HOST='39.107.245.253', 
MASTER_USER='repl', 
MASTER_PASSWORD='repl', 
MASTER_LOG_FILE='mysql-bin 隆.000001', 
MASTER_LOG_POS= 154;
```



## 使用Mycat 实现读写分离

#### **1.Mycat**

Mycat是一款开源的数据库中间件，其官网为http://www.mycat.io/，其中官方对它介绍为：

```
Mycat 是一个强大的数据库中间件，不仅仅可以用作读写分离、以及分表分库、容灾备份，而且可以用于多租户应用开发、云平台基础设施、让你的架构具备很强的适应性和灵活性，借助于即将发布的Mycat 智能优化模块，系统的数据访问瓶颈和热点一目了然，根据这些统计分析数据，你可以自动或手工调整后端存储，将不同的表映射到不同存储引擎上，而整个应用的代码一行也不用改变。
```

Mycat的实现原理为：

```
Mycat 的原理中最重要的一个动词是“拦截”，它拦截了用户发送过来的SQL 语句，首先对SQL 语句做了一些特定的分析：如分片分析、路由分析、读写分离分析、缓存分析等，然后将此SQL 发往后端的真实数据库，并将返回的结果做适当的处理，最终再返回给用户。
```

MyCat作为数据库的中间件，对于上层应用来说，他就是一个数据库。因此需**要配置数据库的用户名，密码，数据库名，以及读写权限**。

#### 2. 部署Mycat

应用是直接连接Mycat，然后Mycat管理了1个主数据库和1个从数据库，架构如下：

![](https://img2018.cnblogs.com/blog/840503/201904/840503-20190429132621211-438885332.png)

其中每个组件对应服务器地址为：

- Mycat：192.168.197.131
- 主库：192.168.197.135
- 从库：192.168.197.136

下面是MyCat的默认配置(部分)：

![](https://user-gold-cdn.xitu.io/2019/6/18/16b6afde1beb0271?w=640&h=425&f=jpeg&s=35594)

**在 schema.xml 文件中配置读写分离**

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">
 <!-- 数据库配置，与server.xml中的数据库对应 -->
 <schema name="db_test" checkSQLschema="false" sqlMaxLimit="100" dataNode="db_node"></schema>
 <!-- 分片配置 -->
 <dataNode name="db_node" dataHost="db_host" database="db_test" />
 <!-- 物理数据库配置 -->
 <dataHost name="db_host" maxCon="1000" minCon="10" balance="3"
 writeType="0" dbType="mysql" dbDriver="native" switchType="1" slaveThreshold="100">
 <heartbeat>select user()</heartbeat>
 <!-- can have multi write hosts -->
 <writeHost host="hostM1" url="mysql_master:3306" user="root"
 password="apple">
 <!-- can have multi read hosts -->
 <readHost host="hostS2" url="mysql_slaver:3306" user="root" password="apple" />
 </writeHost>
 </dataHost>
</mycat:schema>
```

```xml
sqlMaxLimit配置默认查询数量
database为真实数据库名
balance="0", 不开启读写分离机制，所有读操作都发送到当前可用的writeHost 上。
balance="1"，全部的 readHost 与 stand by writeHost 参与 select 语句的负载均衡，简单的说，当双主双从模式(M1 ->S1 ， M2->S2，并且 M1 与 M2 互为主备)，正常情况下， M2,S1,S2 都参与 select 语句的负载均衡。
balance="2"，所有读操作都随机的在 writeHost、 readhost 上分发。
balance="3"， 所有读请求随机的分发到 wiriterHost 对应的 readhost 执行,writerHost 不负担读压力，注意 balance=3 只在 1.4 及其以后版本有， 1.3 没有。
writeType="0", 所有写操作发送到配置的第一个 writeHost，第一个挂了切到还生存的第二个writeHost，重新启动后已切换后的为准，切换记录在配置文件中:dnindex.properties .
writeType="1"，所有写操作都随机的发送到配置的 writeHost。
writeType="2"，没实现。
-1 表示不自动切换
1 默认值，自动切换
2 基于MySQL 主从同步的状态决定是否切换
```

### 启动Mycat

# 启动mycat

在mycat所在的服务器启动

```
./mycat start
mysql -uroot -p123456 -P8066 -h127.0.0.1
#stop
./mycat stop
```