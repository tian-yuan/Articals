# Spring Boot AutoConfiguration 解密

![96](https://upload.jianshu.io/users/upload_avatars/901303/92f1b589fd1b?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96)

 

[Allen在学习](https://www.jianshu.com/u/9e72bcc82f4b)

 

**关注

2016.10.25 01:33* 字数 707 阅读 1439评论 0喜欢 2

Boot 中提供了 @EnableAutoConfiguration 这个注解来实现spring的context以及配置一些它“揣测”我们可能需要的配置。那么我们下面来分析下它揣测的过程。

先回顾下在上一篇中介绍过 [Profile](https://www.jianshu.com/p/01efe59d6a64) 的一些用法，大致是可以根据我们设置的 active profile 来使用不同的配置文件或者加载不同的 bean。 但是还有些情况使用 profile 并不能解决问题 。比如只有在 Class A 存在的情况下才加载 Class B， 或者当配置文件里面存在某个特定的配置项时才加载 Class A，这时我们可以使用Spring 中的 @Conditional。 如下的例子就是通过判断是否有mongo/pgsql的驱动类存在于ClassPath中来决定是否加载mongo/pgsql的配置。

```
@Configuration
public class DBConfig {
    @Bean
    @Conditional(MongoDriverCondition.class)
    public DbConfig getMongoClient() {
        return new MongoClient();
    }
    
    @Bean
    @Conditional(PostgresDriverCondition.class)
    public DbConfig getPGClient() {
        return new PGClient();
    }

}

public class MongoDriverCondition implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext,
        AnnotatedTypeMetadata metadata) 
    try {·
            Class.forName("com.mongodb.Server");
            return true;
        } catch (ClassNotFoundException e) {
            return false;
        }
}

```

Boot 的 @EnableAutoConfiguration 的关键点就是使用了 @Conditional. 首先当我们使用注解 @EnableAutoConfiguration 在我们的应用的主入口类上，在项目启动时 Boot 便会加载 spring-boot-autoconfiguration 包里面的 spring.factories ，截取部分内容如下：

```
    # Auto Configure
    org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
    org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
    org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
    org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration,\
    org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration,\
    org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
    org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\

```

意思就是要加载这些配置文件，置于怎么判断要不要加载，我们拿 AMQP 举个栗子。 比方说我们想要使用 RabbitMQ 来作为我们项目的MQ，（假定我们使用gradle）那么我们只需要引入 `compile("org.springframework.boot:spring-boot-starter-amqp")` 这个依赖就OK了，然后就可以顺水推舟写我们的各种receiver和publisher了，那么为什么这个加了这个依赖就好了呢？ 我们可以看下它的源码，发现只有一个pom文件， 里面有如下依赖：

- `spring-messaging`
- `spring-rabbit`

此时我们再去看 class RabbitAutoConfiguration， 注意 它有个 @Configuration注解，表明它是一个普通的 spring 的 Config 类，接着有如下两个注解：

- `@ConditionalOnClass({ RabbitTemplate.class, Channel.class })`
- `@EnableConfigurationProperties(RabbitProperties.class)`

第一个注解 @ConditionalOnClass 的意思就是说当后面 （）里面指定的类存在于 ClassPath 里面时，condition的条件成立，就加载目标类。 对于此处，指定的两个 Class已经被包含在 `spring-messaging`和`spring-messaging` 这两个依赖里面，所以 RabbitAutoConfiguration 会在项目启动时被加载，故而我们可以直接使用rabbit了。顺便说下第二个注解，它的意思是说把配置文件里面的 形如 `spring.rabbitmq.XXX`（这个前缀定义在 RabbitProperties里面）的值绑定到 `Class RabbitProperties` 上面。 AMQP的分析就到这里，其他的自动配置类也是相似的原理，比如想要使用 Ehcache,只要在依赖中加上 Ehcache , 那么项目中的 CacheManager 的实现类就是 Ehcache 对应的了。

The end.