# Java5ExtTree
Java高级基础技术研究


<pre>
注解：

      @Resource的装配顺序：
          1、@Resource后面没有任何内容，默认通过name属性去匹配bean，找不到再按type去匹配
          2、指定了name或者type则根据指定的类型去匹配bean
          3、指定了name和type则根据指定的name和type去匹配bean，任何一个不匹配都将报错

      然后，区分一下@Autowired和@Resource两个注解的区别：
          1、@Autowired默认按照byType方式进行bean匹配，@Resource默认按照byName方式进行bean匹配

          2、@Autowired是Spring的注解，@Resource是J2EE的注解，这个看一下导入注解的时候这两个注解的包名就一清二楚了


      Spring属于第三方的，J2EE是Java自己的东西，因此，建议使用@Resource注解，以减少代码和Spring之间的耦合。
</pre>

<pre>
@SpringBootApplication注解

      1）@SpringBootConfiguration继承自@Configuration，二者功能也一致，标注当前类是
        配置类，并会将当前类内声明的一个或多个以@Bean注解标记的方法的实例纳入到srping容
        器中，并且实例名就是方法名。

      2）@EnableAutoConfiguration的作用启动自动的配置，@EnableAutoConfiguration注
        解的意思就是Springboot根据你添加的jar包来配置你项目的默认配置，比如根据
        spring-boot-starter-web ，来判断你的项目是否需要添加了webmvc和tomcat，就会自
        动的帮你配置web项目中所需要的默认配置。在下面博客会具体分析这个注解，快速入门的
        demo实际没有用到该注解。
      3）@ComponentScan，扫描当前包及其子包下被@Component，@Controller，@Service，
        @Repository注解标记的类并纳入到spring容器中进行管理。是以前的
        <context:component-scan>（以前使用在xml中使用的标签，用来扫描包配置的平行支
        持）。所以本demo中的User为何会被spring容器管理。
</pre>

<pre>
Redis,Memcache,mongoDB的区别

      1、性能
           1) 都比较高，性能对我们来说应该都不是瓶颈
           2) 总体来讲，TPS方面redis和memcache差不多，要大于mongodb

      2、操作的便利性
           1) memcache数据结构单一
           2) redis丰富一些，数据操作方面，redis更好一些，较少的网络IO次数
           3) mongodb支持丰富的数据表达，索引，最类似关系型数据库，支持的查询语言非常丰富

      3、内存空间的大小和数据量的大小
           1) redis在2.0版本后增加了自己的VM特性，突破物理内存的限制；可以对key value设
              置过期时间（类似memcache）
           2) memcache可以修改最大可用内存,采用LRU算法
           3) mongoDB适合大数据量的存储，依赖操作系统VM做内存管理，吃内存也比较厉害，服
              务不要和别的服务在一起

      4、可用性（单点问题）
           对于单点问题，
               1) redis，依赖客户端来实现分布式读写；主从复制时，每次从节点重新连接主节
                  点都要依赖整个快照,无增量复制，因性能和效率问题，
                  所以单点问题比较复杂；不支持自动sharding,需要依赖程序设定一致hash 机制。

                  一种替代方案是，不用redis本身的复制机制，采用自己做主动复制（多份存
                  储），或者改成增量复制的方式（需要自己实现），一致性问题和性能的权衡

               2) Memcache本身没有数据冗余机制，也没必要；对于故障预防，采用依赖成熟的
                  hash或者环状的算法，解决单点故障引起的抖动问题。

               3) mongoDB支持master-slave,replicaset（内部采用paxos选举算法，自动故
                  障恢复）,auto sharding机制，对客户端屏蔽了故障转移和切分机制。

      5、可靠性（持久化）
            对于数据持久化和数据恢复，
               1) redis支持（快照、AOF）：依赖快照进行持久化，aof增强了可靠性的同时，对性
                  能有所影响
               2) memcache不支持，通常用在做缓存,提升性能；
               3) MongoDB从1.8版本开始采用binlog方式支持持久化的可靠性

      6、数据一致性（事务支持）
               1) Memcache 在并发场景下，用cas保证一致性
               2) redis事务支持比较弱，只能保证事务中的每个操作连续执行
               3) mongoDB不支持事务

      7、数据分析
               mongoDB内置了数据分析的功能(mapreduce),其他不支持

      8、应用场景
               1) redis：数据量较小的更性能操作和运算上
               2) memcache：用于在动态系统中减少数据库负载，提升性能;做缓存，提高性能
                 （适合读多写少，对于数据量比较大，可以采用sharding）
               3) MongoDB:主要解决海量数据的访问效率问题
</pre>

<pre>
使用druid连接池的超时回收机制排查连接泄露问题:

    在工程中使用了druid连接池，运行一段时间后系统出现异常：

           Caused by: org.springframework.jdbc.CannotGetJdbcConnectionException: Could not get JDBC Connection; nested exception is com.alibaba.druid.pool.GetConnectionTimeoutException: wait millis 60009, active 50  
                at org.springframework.jdbc.datasource.DataSourceUtils.getConnection(DataSourceUtils.java:80)  
                at org.springframework.jdbc.support.JdbcUtils.extractDatabaseMetaData(JdbcUtils.java:280)  
                ... 64 more  
Caused by: com.alibaba.druid.pool.GetConnectionTimeoutException: wait millis 60000, active 50  
                at com.alibaba.druid.pool.DruidDataSource.getConnectionInternal(DruidDataSource.java:1071)  
                at com.alibaba.druid.pool.DruidDataSource.getConnectionDirect(DruidDataSource.java:898)  
                at com.alibaba.druid.filter.FilterChainImpl.dataSource_connect(FilterChainImpl.java:4544)  

      mysql数据库最大连接数设置为500，使用客户端能正常连接。连接数被未被占满。

      分析原因应该是程序中有地方连接未关闭造成的。那如何来定呢？使用druid连接池的超时回收机
      制，在配置中增加以下内容：

          <!-- 超过时间限制是否回收 -->  
          <property name="removeAbandoned" value="true" />  
          <!-- 超时时间；单位为秒。180秒=3分钟 -->  
          <property name="removeAbandonedTimeout" value="180" />  
          <!-- 关闭abanded连接时输出错误日志 -->  
          <property name="logAbandoned" value="true" />

      运行程序，当连接超过3分钟后会强制进行回收，并输出异常日志。 

      2014-10-13 16:02:28,919 ERROR [com.alibaba.druid.pool.DruidDataSource] - <abandon connection, open stackTrace  
        at java.lang.Thread.getStackTrace(Thread.java:1567)  
        at com.alibaba.druid.pool.DruidDataSource.getConnectionDirect(DruidDataSource.java:995)  
        at com.alibaba.druid.filter.FilterChainImpl.dataSource_connect(FilterChainImpl.java:4544)  
        at com.alibaba.druid.filter.stat.StatFilter.dataSource_getConnection(StatFilter.java:661)  
        at com.alibaba.druid.filter.FilterChainImpl.dataSource_connect(FilterChainImpl.java:4540)  
        at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:919)  
        at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:911)  
        at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:98)  
          
        at cn.org.xxx.xxx.xxx.PaginationInterceptor.intercept(PaginationInterceptor.java:96)  
          
        at org.apache.ibatis.plugin.Plugin.invoke(Plugin.java:60)  
        at com.sun.proxy.$Proxy59.query(Unknown Source)  
        at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:108) 
</pre>

<pre>
tomcat调优改用APR库

       1） tomcat默认采用的BIO模型，在几百并发下性能会有很严重的下降。tomcat自带还有NIO的模型，另外也可以调用APR的库来实现操作
          系统级别控制。

       2） NIO模型是内置的，调用很方便，只需要将上面配置文件中protocol修改成org.apache.coyote.http11.Http11NioProtocol，
          重启即可生效。上面配置我已经改过了，默认的是HTTP/1.1。
    
       3）APR则需要安装第三方库，在高并发下会让性能有明显提升，安装完成后重启即可生效。如使用默认protocal就是apr，但最好把将
          protocol修改成org.apache.coyote.http11.Http11AprProtocol，会更加明确。

       补充Bio、Nio、Apr模式的测试结果：

       对于这几种模式，我用ab命令模拟1000并发测试10000词，测试结果比较意外，为了确认结果，我每种方式反复测试了10多次，并且在两
       个服务器上都测试了一遍。结果发现Bio和Nio性能差别非常微弱，难怪默认居然还是Bio。但是采用apr，连接建立的速度会有50%～100%
       的提升。直接调用操作系统层果然神速啊，这里强烈推荐apr方式！
</pre>

<pre>
1、为什么要重写clone()方法？

答案：Java中的浅度复制是不会把要复制的那个对象的引用对象重新开辟一个新的引用空间，当我们需要深度复制的时候，这个时候我们就要重写
clone()方法。

利用串行化来做深复制

      把对象写到流里的过程是串行化（Serilization）过程，但是在Java程序师圈子里又非常形象地称为“冷冻”或者“腌咸菜（picking）”
      过程；而把对象从流中读出来的并行化（Deserialization）过程则叫做 “解冻”或者“回鲜(depicking)”过程。

      应当指出的是，写在流里的是对象的一个拷贝，而原对象仍然存在于JVM里面，因此“腌成咸菜”的只是对象的一个拷贝，Java咸菜还可以回
      鲜。

      在Java语言里深复制一个对象，常常可以先使对象实现Serializable接口，然后把对象（实际上只是对象的一个拷贝）写到一个流里（腌成
      咸菜），再从流里读出来（把咸菜回鲜），便可以重建对象。
</pre>

<pre>
Java中的10颗语法糖

      1：泛型与类型擦除
      2：自动装箱与拆箱
      3：foreach循环
      5：变长参数
      6：条件编译
      7：内部类
      8：枚举类
      9：断言语句
      11：对枚举和字符串的switch支持
      12：在try语句中定义和关闭资源 jdk7提供了try-with-resources,可以自动关闭相关的资源(只要该资源实现了AutoCloseable接口，
          jdk7为绝大部分资源对象都实现了这个接口)
</pre>

Nginx/LVS/HAProxy负载均衡软件的优缺点详解
https://www.cnblogs.com/duanxz/p/5011671.html

<pre>
负载均衡技术种类

      1：DNS负载均衡
      2：代理服务器负载均衡
      3：地址转换网关负载均衡
      5：反向代理负载均衡
      6: 混合型负载均衡
</pre>

<pre>
INSERT INTO SELECT语句 要求T2表存在
SELECT INTO FROM语句  要求目标表Table2不存在，因为在插入时会自动创建表Table2，并将Table1中指定字段数据复制到Table2中.

用主键倒序插入：
      情况和1一样。即默认的"select * from tb" 和 "select * from tb order id(PK) DESC" 是一样的情况，这里说的一样是锁方式
      一样（都是逐步，只是顺序不一样）。

      从上面可知：通过主键排序或则不加排序字段的导入操作"insert into tb select * from tbx"，是会锁tbx表，但他的锁是逐步地锁
      定已经扫描过的记录。
</pre>

<pre>
Mysql的行转列 VS 列转行
</pre>

![](https://i.imgur.com/I0O15cZ.png)

![](https://i.imgur.com/wO5axNN.png)

![](https://i.imgur.com/kq2NL6n.png)

<pre>
Mysql保留关键字

      遇到关键字使用时冲突，查询可以使用``括住关键字。使用的符号是半角的``,即esc下面的那个
      按键。

      今天在重MySql 语句时出现错误：
            select * from kw_photo where albumId=102 order by order
      原来order字段跟关键字冲突，需要用''引起来。

            select * from kw_photo where albumId=102 order by 'order'
</pre>

<pre>
Guava Math：
      主要包含：
              1）IntMath（处理int），LongMath(处理 long)， BigIntegerMath(处理BigInteger)

Guava HashFunction：
      另外还包含Hasher, Funnel

      Java内建的散列码[hash code]概念被限制为32位，并且没有分离散列算法和它们所作用的数
      据，因此很难用备选算法进行替换。此外，使用Java内建方法实现的散列码通常是劣质的，部分
      是因为它们最终都依赖于JDK类中已有的劣质散列码。

      Object.hashCode往往很快，但是在预防碰撞上却很弱，也没有对分散性的预期。这使得它们很
      适合在散列表中运用，因为额外碰撞只会带来轻微的性能损失，同时差劲的分散性也可以容易地
      通过再散列来纠正（Java中所有合理的散列表都用了再散列方法）。然而，在简单散列表以外的
      散列运用中，Object.hashCode几乎总是达不到要求——因此，有了com.google.common.hash包

Guava Range区间

Guava Cache

Guava 函数式编程

Guava字符串处理
      Joiner， Splitter
</pre>

![](https://i.imgur.com/l3BM18m.png)

<pre>
序列化框架对比
</pre>

<pre>
10G 个整数找出中位数，内存限制为 2G

      解法：首先假设是32位无符号整数。
           1. 读一遍10G个整数，把整数映射到256M个区段中，用一个64位无符号整数给每个相应区
              段记数。
              说 明：整数范围是0 - 2^32 - 1，一共有4G种取值，映射到256M个区段，则每个区
              段有16（4G/256M = 16）种值，每16个值算一段， 0～15是第1段，16～31是第2
              段，……2^32-16 ～2^32-1是第256M段。一个64位无符号整数最大值是0～8G-1，这里
              先不考虑溢出的情况。总共占用内存256M×8B=2GB。

           2. 从前到后对每一段的计数累加，当累加的和超过5G时停止，找出这个区段（即累加停止
              时达到的区段，也是中位数所在的区段）的数值范围，设为[a，a+15]，同时记录累加
              到前一个区段的总数，设为m。然后，释放除这个区段占用的内存。

           3. 再读一遍10G个整数，把在[a，a+15]内的每个值计数，即有16个计数。

           4. 对新的计数依次累加，每次的和设为n，当m+n的值超过5G时停止，此时的这个计数所
              对应的数就是中位数。
</pre>

<pre>
HTTP重复提交解决方法：

      1）缓存lock，缓存此用户的操作行为，注意仅仅缓存操作的标志，下次进入判断此标志是否存在，存在即不进入数据库事务。
      2）应用程序application lock，和1）相比，会阻塞其他用户的行为
      3）模仿银行扣款机制，数据表建一个随机唯一标志，每次请求带上这个标志，操作的同时进行修改这个标志。
      5）应用程序生成唯一标志，数据库做字段的唯一索引
      6）使用事务的隔离级别
      7）使用redis的incr控制用户的并发数，memcache的add也可以实现这种效果，memcached借助cas
</pre>