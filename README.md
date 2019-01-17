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