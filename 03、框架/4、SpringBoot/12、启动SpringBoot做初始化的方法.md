# CommandLineRunner

在Spring boot项目的实际开发中，我们有时需要项目服务启动时加载一些数据或预先完成某些动作。为了解决这样的问题，Spring boot 为我们提供了一个方法：通过实现接口 CommandLineRunner 来实现这样的需求。

实现方式：只需要一个类即可，无需其他配置。 



```java
@Component
@Order(value=1)
public class CommandLineRunnerImpl implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
            // DOTO
    }
}
```

@Order注解标识了执行顺序，Order值越小，优先级越大，越先执行





