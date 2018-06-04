#### ServiceLoader

首先定义一个接口，具体如下：

```
public interface IService {  
	public String sayHello();  
       
	public String getScheme();  
}
```

该接口有两个子类，分别为HDFSService和LocalService：

```
public class HDFSService implements IService {
    @Override  
    public String sayHello() { 
        return "Hello HDFS!!";  
    }

    @Override  
    public String getScheme() {
        return "hdfs";  
	}
}
```

  ```
public class LocalService implements IService {  
    @Override  
    public String sayHello() {  
        return "Hello Local!!";  
    }  

    @Override  
    public String getScheme() {  
        return "local";  
    }
}
  ```



需要在META-INF/services下以IService这个类的全名来新建立一个文件（文件名为com.rabbitai.test.IService），文件中的内容为两个实现类的全名，如下：

```
org.hadoop.java.HDFSService  
org.hadoop.java.LocalService  
```



所有的实现和配置都已经完成，下面写一个测试类来看一下结果:

```
public class ServiceLoaderTest {  
    /** 
     * @param args
     */  
    public static void main(String[] args) {  
        //need to define related class full name in /META-INF/services/....  
        ServiceLoader<IService> serviceLoader = ServiceLoader  
                .load(IService.class);  
        for (IService service : serviceLoader) {  
            System.out.println(service.getScheme()+"="+service.sayHello());  
        }  
    }  
}  

```



具体的输出来如下：

```
hdfs=Hello HDFS!!  
local=Hello Local!!  
```

可以看到ServiceLoader可以根据IService把定义的两个实现类找出来，返回一个ServiceLoader的实现，而ServiceLoader实现了Iterable接口，所以可以通过ServiceLoader来遍历所有在配置文件中定义的类的实例。Hadoop FileSystem就是通过这个机制来根据不同文件的scheme来返回不同的FileSystem。

因此，以后可以通过ServiceLoader来实现一些类似的功能。而不用依赖像Spring这样的第三方框架。



#### 在 apollo-client 中存在相同的使用方式

在 apollo-client 工程下 /src/main/resources/META-INF/services 下有 com.ctrip.framework.apollo.internals.Injector 文件，文件内容如下：

```
com.ctrip.framework.apollo.internals.DefaultInjector
```

DefaultInjector 中有 ApolloModule 的实现:

```
private static class ApolloModule extends AbstractModule {
    @Override
    protected void configure() {
      	bind(ConfigManager.class).to(DefaultConfigManager.class).in(Singleton.class);
      	bind(ConfigFactoryManager.class).to(DefaultConfigFactoryManager.class).in(Singleton.class);
          bind(ConfigRegistry.class).to(DefaultConfigRegistry.class).in(Singleton.class);
      bind(ConfigFactory.class).to(DefaultConfigFactory.class).in(Singleton.class);
      bind(ConfigUtil.class).in(Singleton.class);
      bind(HttpUtil.class).in(Singleton.class);
      bind(ConfigServiceLocator.class).in(Singleton.class);
      bind(RemoteConfigLongPollService.class).in(Singleton.class);
    }
  }
```



由于这里用到了 google 的 Guice 作为依赖注入，但是实际上还是要用 spring 框架，因为在 DefaultInjector 加载时，依赖于 spring 框架的 org/springframework/core/env/EnumerablePropertySource。