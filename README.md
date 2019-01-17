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