##Spring boot 笔记

#####1 Spring Boot的配置有哪些配置方式？有什么区别？
答: 有两种配置方式，1.application.properties,2.application.yml。这两种方式的主要的区别是：语法不同，首先properties是需要将每个配置信息的路径写全，而yml文件是可以在相同路径下使用空格区分上下级关系的，总而言之yml文件是将空格玩到极致的配置文件。而且yml有天然的树状结构，看的更清晰。

#####2 Spring Boot的核心注解是什么？
答：@SpringBootApplication注解

#####3 开启Spring Boot特性的方式是什么？
答：1.继承spring-boot-starter-parent项目spring-boot-dependencies项目依赖

#####4 运行Spring Boot的方式？
答：1.运行带有main方法  2.将项目打包成jar包通过命令行 java -jar 的方式 3.通过spring-boot-plugin的方式

#####5 Spring Boot读取配置的方式？
答：1.读取application配置文件：@value注解读取方式 ，@ConfigurationProperties注解读取方式 2.读取指定文件：@PropertySource+@Value的读取方式 ，@ConfigurationProperties+@PropertySource的读取方式。以上所有加载出来的配置都可以通过Environment注入获取到。从以上示例来看，Spring Boot可以通过@PropertySource,@Value,@Environment,@ConfigurationProperties来绑定变量。

#####6 Spring Boot中的监视器？
答：Spring boot actuator，用来访问生产环境中正在运行的应用程序的当前状态。

#####7 Spring Boot自动配置的原理 ？
答：SpringBoot 自动配置主要通过 @EnableAutoConfiguration, @Conditional, @EnableConfigurationProperties 或者 @ConfigurationProperties 等几个注解来进行自动配置完成的。
@EnableAutoConfiguration 开启自动配置，主要作用就是调用 Spring-Core 包里的 loadFactoryNames()，将 autoconfig 包里的已经写好的自动配置加载进来。
@Conditional 条件注解，通过判断类路径下有没有相应配置的 jar 包来确定是否加载和自动配置这个类。
@EnableConfigurationProperties 的作用就是，给自动配置提供具体的配置参数，只需要写在 application.properties 中，就可以通过映射写入配置类的 POJO 属性中。
#####8 spring-boot-maven-plugin？

答：在添加了该插件之后，当运行“mvn package”进行打包时，会打包成一个可以直接运行的JAR文件，使用"java -jar"命令就可以直接运行。如果未进行上述配置，应用本地可以正常启动，但是发布到测试机器就无法启动。

#####9 使用配置文件通过Spring Boot配置特定环境的配置？
答：如果是properties文件分三步：1.在application.properties中配置通用内容，使用spring.profiles.active=dev，指定默认的配置。2.在application-{profile}.properties中配置各个环境不同的内容。3.通过命令行方式去激活不同的环境配置。Yml文件支持多文档块形式。

#####10 如何禁用一个特定自动配置类？
答：使用 @EnableAutoConfiguration 或 @SpringBootApplication 注解的 exclude 属性来指明。
#####11 Spring Boot工厂模式的加载？
答：工厂加载机制是Spring内部提供的一个约定俗成的加载方式。只需要在模块的META-INF目录下定义Properties格式的spring.factories文件，这个Properties格式的文件中的key是接口或抽象类的全名，value是以逗号 " , " 分隔的实现类。我们只需要遵守这个机制并在对应的文件中写出需要加载的接口和实例即可，或者自己使用SpringFactoriesLoader实现加载。