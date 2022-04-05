[TOC]

# 首先我门要知道什么是跨域：

跨域是指 不同域名之间相互访问。跨域，指的是浏览器不能执行其他网站的脚本。它是由浏览器的同源策略造成的，是浏览器对JavaScript施加的安全限制。

也就是如果在A网站中，我们希望使用Ajax来获得B网站中的特定内容
如果A网站与B网站不在同一个域中，那么就出现了跨域访问问题。

# 什么是同一个域？
同一协议，同一ip，同一端口，三同中有一不同就产生了跨域。

# 前端解决跨域：
前边也说了，跨域是浏览器不能执行其他网站的脚本。它是由浏览器的同源策略造成的，是浏览器对JavaScript施加的安全限制。
解决：
所以搞一个node 服务器做代理，发出请求到node 服务器，node服务器转发到后端就可以绕过跨域问题。

# 后端解决跨域问题：
后端解决就比较简单了。例如我用的springboot，只用在Controller类上添加一个“@CrossOrigin“注解就可以实现对当前controller 的跨域 访问了，当然这个标签也可以加到方法上。

```java
@RequestMapping(value = "/users")
@RestController
@CrossOrigin
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping(method = RequestMethod.POST)
    @CrossOrigin
    public User create(@RequestBody @Validated User user) {
        return userService.create(user);
    }
    ｝
```

## 相关知识：

CSRF是什么？

CSRF（Cross-site request forgery），中文名称：跨站请求伪造，也被称为：one click attack/session riding，缩写为：CSRF/XSRF。

## 解决方法2

新建跨域配置文件如下即可

```java
@Configuration
public class CorsConfiguration {
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**")
                    .allowCredentials(false)
                    .allowedMethods("POST", "GET", "PUT", "OPTIONS", "DELETE")
                    .allowedOrigins("*");
            }
        };
    }

}

```

