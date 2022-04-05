# 1. MyBatis插件实现sql修改

> https://blog.csdn.net/qq_29653517/article/details/86299928?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link

一、应用场景

1.分页，如com.github.pagehelper的分页插件实现；

2.拦截sql做日志监控；

3.统一对某些sql进行统一条件拼接，类似于分页。

二、MyBatis的拦截器简介





> 如果有多个拦截器，需要通过循环sqlSessionFactory添加拦截器

```java
@Configuration
public class MapperConfig {

  @Autowired
  private List<SqlSessionFactory> sqlSessionFactoryList;

  // 注册插件
  @Bean
  public MyPlugin myPlugin() {
    MyPlugin myPlugin = new MyPlugin();
    // ...... 插件的其他属性设置（如果有的话）
    for (SqlSessionFactory sqlSessionFactory : sqlSessionFactoryList) {
      sqlSessionFactory.getConfiguration().addInterceptor(myPlugin);
    }
    return myPlugin;
  }
}
```

