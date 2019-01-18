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