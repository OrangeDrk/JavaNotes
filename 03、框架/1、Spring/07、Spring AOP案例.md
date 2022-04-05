[TOC]

# 异常信息收集

> 用到了log4j的jar包
>
> log4j的properties文件在`..\log4配置文件.md` 文件中

## Service层代码

```java
@Service
public class UserServiceImpl implements UserService {
    @Override
    public void addUser() {
        int i=1/0;//手动制造一个异常
        System.out.println("添加用户");
    }
}
```



## web层代码

```java
@Controller
public class UserServlet {

    @Autowired
    private UserService userService;

    public void addUser(){
        userService.addUser();//调用service层的addUser方法
    }
}
```



## 切面代码

> 收集异常信息需要通过==异常通知==来收集

```java
@Component
@Aspect
public class ExceptionAspect {

    //声明一个logger对象，用于后面存放日志
    private static Logger logger = Logger.getLogger(ExceptionAspect.class);

    @AfterThrowing(value = "execution(* cn.tedu.service..*(..))",throwing = "e")
    public void afterThrowing(JoinPoint jp,Throwable e){
        //获取目标对象
        Object target = jp.getTarget();
        Class clz = target.getClass();
        //获取目标方法
        MethodSignature signature = (MethodSignature) jp.getSignature();
        logger.info("在["+clz+"]类中的["+signature+"]方法中发生了["+e+"]异常");
    }
}
```



## 单元测试

```java
public class Test {

    @org.junit.Test
    public void test01(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserServlet userServlet = (UserServlet) context.getBean("userServlet");
        userServlet.addUser();
        ((ClassPathXmlApplicationContext) context).close();
    }
}
```



## 结果

![](img\AOP案例：收集异常信息.png)





------



# 统计业务方法执行的时间

## service层代码

```java
@Service
public class UserServiceImpl implements UserService {
    @Override
    public void addUser() {
        System.out.println("添加用户");

        try {
            Thread.sleep(10);//模拟耗时
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```



## web层代码

```java
@Controller
public class UserServlet {

    @Autowired
    private UserService userService;

    public void addUser(){
        userService.addUser();//调用service层的addUser方法
    }
}

```



## 切面类

```java
@Component
@Aspect
public class UseTimeAspect {

    private static Logger logger = Logger.getLogger(UseTimeAspect.class);

    @Around("execution(* cn.tedu.service..*(..))")
    public Object useTime(ProceedingJoinPoint pjp) throws Throwable {

        
        long start = System.currentTimeMillis();//记录开始时间
        
        //手动执行方法
        Object proceed = pjp.proceed();

        long end = System.currentTimeMillis(); //记录结束时间
        long useTime = end - start;//用时

        //获取目标对象的类对象
        Class clz = pjp.getTarget().getClass();
        //获取目标对象执行的方法
        MethodSignature signature = (MethodSignature) pjp.getSignature();
        //记录日志
        logger.debug("在["+clz+"]中["+signature+"]执行耗时["+useTime+"]");

        //返回返回值
        return proceed;
    }
}
```



## 单元测试

![](img\AOP案例：统计业务方法执行时间.png)



------



# 通过AOP进行权限控制

> 通过AOP来实现权限控制
>
> - 通过自定义注解声明业务方法是否需要权限控制
> - 通过权限注解上的属性声明需要什么样的权限
> - 通过切面拦截业务方法，根据是否需要权限、是否具有权限，控制目标方法的执行
>
> AOP控制权限需要用==环绕通知==



## 自定义一个权限枚举类：`PrivEnum`

> 该枚举类枚举所有的权限
>
> - 游客：VISITOR
> - 用户：USER
> - 管理员：ADMIN

```java
public enum  PrivEnum {
    USER,ADMIN,VISITOR;
}
```



## 自定义权限注解：`@PrivAnno`

> 自定义权限注解，在方法上使用，标明该方法需要什么样的权限才可以执行

```java
@Target(ElementType.METHOD)//注解应用在方法上
@Retention(RetentionPolicy.RUNTIME)//注解保留在源代码、字节码、运行期
public @interface PrivAnno {
    PrivEnum[] value();//一个方法可能具有多个权限，所以要用数组
}
```



## service层：`UserServiceImpl`

> 编写各个业务方法，并通过自定义注解`@PrivAnno`标明需要什么样的权限

```java
@Service
public class UserServiceImpl implements UserService {

    // 无权限限制
    @Override
    public void addUser() {
        System.out.println("添加用户");
    }

    // 用户、管理员权限
    @Override
    @PrivAnno({PrivEnum.USER,PrivEnum.ADMIN})
    public void updateUser() {
        System.out.println("修改用户");
    }

    // 用户、管理员权限
    @Override
    @PrivAnno({PrivEnum.USER,PrivEnum.ADMIN})
    public void queryUser() {
        System.out.println("查询用户");
    }

    // 管理员权限
    @Override
    @PrivAnno(PrivEnum.ADMIN)
    public void deleteUser() {
        System.out.println("删除用户");
    }
}

```



## web层：`UserServlet`

> servlet中通过公共的静态的`PrivEnum`类型的参数，声明具有的权限
>
> - `PrivEnum.USER`：用户权限
> - `PrivEnum.ADMIN`：管理员权限
> - `PrivEnum.VISITOR`：游客

```java
@Controller
public class UserServlet {
    public static PrivEnum priv = PrivEnum.USER;//定义权限，表明该servlet具有的权限

    @Autowired
    private UserService userService;

    //添加用户方法
    public void addUser(){
        userService.addUser();
    }

    //修改用户方法
    public void updateUser(){
        userService.updateUser();
    }

    //查询用户方法
    public void queryUser(){
        userService.queryUser();
    }
    
    //删除用户方法
    public void deleteUser(){
        userService.deleteUser();
    }
}

```



## 切面类：`PrivAspect`

1. 最原始的写法

   > 通过获取【该执行的方法】上【所有的权限】
   >
   > - 没有权限注解：放行
   > - 有权限注解：与调用者（UserServlet）的权限是否匹配
   >   - 匹配：执行方法
   >   - 不匹配：抛出异常

   ```java
   @Component
   @Aspect
   public class PrivAspect {
   
       @Around("execution(* cn.tedu.service..*(..))")
       public Object around(ProceedingJoinPoint pjp) throws Throwable {
           //获取目标对象
           Object target = pjp.getTarget();
           //获取目标对象方法------该方法是接口中的方法
           MethodSignature signature = (MethodSignature) pjp.getSignature();
   
           //因为只有在实现类中的方法上才有权限注解，所以要获取实现类的方法
           //在实现类中寻找【参数】和【返回值类型】与【接口中的方法相同】的方法
           Method method = target.getClass().getMethod(signature.getName(), signature.getParameterTypes());
           if (method.isAnnotationPresent(PrivAnno.class)) {//实现类方法上有权限注解
               //获取方法上的所有PrivAnno注解
               PrivAnno annotation = method.getAnnotation(PrivAnno.class);
               PrivEnum[] value = annotation.value();
   
               //拿到当前servlet的权限
               PrivEnum priv = UserServlet.priv;
               //判断servlet权限是否包含在该方法的权限中
               if (Arrays.asList(value).contains(priv)) {//存在
                   //放行
                   Object proceed = pjp.proceed();
                   return proceed;
               }else {//不存在
                   throw new RuntimeException("权限不足");
               }
           }else {//实现类方法上没有权限注解
               //没有注解的方法：只有添加用户，所有权限都可以访问
               Object proceed = pjp.proceed();
               return proceed;
           }
       }
   }
   ```

2. 通过`@annotation()`注解，简化书写

   > `execution(* cn.tedu.service..*(..)) && @annotation(ps)`：此切入点表达式表示：
   >
   > - 切出指定包下以及子孙包中所有类的所有方法并且要求
   >   - 必须有注解，且注解类型必须是`@annotation(ps)`参数所对应的类型
   >   - `@annotation(ps)`:该ps参数对应方法参数的`PrivAnno ps`（`PrivAnno`为自定义的权限注解）

   ```java
   @Component
   @Aspect
   public class PrivAspect {
        @Around("execution(* cn.tedu.service..*(..)) && @annotation(ps)")
       public Object around(ProceedingJoinPoint pjp,PrivAnno ps ) throws Throwable {
           PrivEnum[] privs = ps.value();//获取所有的注解
   
           PrivEnum priv = UserServlet.priv;//获取调用者的权限
   
           if (Arrays.asList(privs).contains(priv)) {//判断所有权限中是否包含调用者权限
               //注解数组中包含当前servlet权限
               Object proceed = pjp.proceed();//放行
               return proceed;
           }else {//不包含
               throw new RuntimeException("权限不足");
           }
       }
   }
   ```

   

## 单元测试

```java
public class Test {

    @org.junit.Test
    public void test01(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserServlet userServlet = (UserServlet) context.getBean("userServlet");
        //userServlet.addUser();
        //userServlet.updateUser();
        userServlet.deleteUser();
        ((ClassPathXmlApplicationContext) context).close();
    }
}

```

1. 当servlet权限位

------



# 事务控制

> 通过AOP实现事务控制
>
> - 开发事务注解，通过业务方法上是否有注解来标识方法是否需要事务
> - 在切面中通过判断目标方法是否具有事务注解决定是否执行事务
> - 通过事务管理器管理事务，防止耦合

## 开发事务注解：`Trans`

> 此注解只是标注是否需要事务，所以注解中不需要传入任何值

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Trans {
}
```



## 业务层(Service)使用注解

```java
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserDao userDao = null;

    @Trans//假设addUser方法需要事务，就添加Trans注解即可
    @Override
    public void addUser(User user) {
        userDao.addUser(user);
    }

    @Override
    public void delUser(int id) {
        userDao.delUser(id);
    }
}
```



## 开发事务管理器：`TransactionManager`

### 单线程情况下

> 将Connection的获取、事务的开启、提交、回滚、Connection的关闭统一在这个类（事务管理器）中执行。
>
> ==dao层的Connection==和==切面中对需要事务的方法的扩展需要的Connection==，全部==从事务管理器类中获取==，保证了使用同一个Connection

```java
public class TransactionManager {
	
	private static Connection conn = JDBCUtils.getConn();
	
	private TransactionManager() {}
	
	//获取连接
	public static Connection getConn() {
		return conn;
	}
    
	//开启事务
	public static void startTran() {
		try {
			conn.setAutoCommit(false);
		} catch (Exception e) {
			e.printStackTrace();
			throw new RuntimeException(e);
		}
	}
    
	//提交事务
	public static void commitTran() {
		try {
			conn.commit();
		} catch (Exception e) {
			e.printStackTrace();
			throw new RuntimeException(e);
		}
	}
    
	// 回滚事务
	public static void rollbackTran()  {
		try {
			conn.rollback();
		} catch (Exception e) {
			e.printStackTrace();
			throw new RuntimeException(e);
		}
	}
    
	// 释放资源
	public static void release() {
		JDBCUtils.close(conn, null, null);
	}
}

```



### 解决多线程下，让每个线程独有一个Connection的问题

> 多线程下，因为`TransactionManager`类中的Connection对象是静态的，每个线程获取到的都是同一个连接，在进行事务操作时会出现线程安全问题。所以要让每个线程单独具有自己的Connection才可以

#### `ThreadLocal`

> 也叫：==本地线程变量==
>
> 当在多线程环境下时，所有线程都用统一个conn必然会出现线程安全问题。此时可以通过`ThreadLocal`让每个`Thread`都具有自己的`Conn`，保证线程安全。
>
> `Thread`类中有一个Map容器，存放当前线程才有的数据。但是这个Map是不能直接操作的。Sun公司提供了一个`ThreadLocal`类，来操作这个Map

1. `ThreadLocal`类的方法及作用

   | `方法`           | `作用`                         |
   | ---------------- | ------------------------------ |
   | `get()`          | 获取本地线程局部变量中的值     |
   | `set()`          | 向本地线程局部变量中添加值     |
   | `remove()`       | 移除本地线程局部变量中的值     |
   | `initialValue()` | 返回本地线程局部变量中的初始值 |

   

2. 图解：

   ![](img\ThreadLocal图解.png)



#### `TransactionManager`类的改造

```java
/*
	ThreadLocal给每个线程内部分配一个属于该线程的conn对象
	保证每个线程都各自使用各自的conn，保证了不会造成线程安全问题
*/
public class TransactionManager {

    /*通过匿名内部类对ThreadLocal的initialValue方法进行重写。
    	使该对象初始化时就有一个Connection
    */
    private static ThreadLocal<Connection> tl = new ThreadLocal<Connection>() {
        protected Connection initialValue() {
            return JDBCUtils.getConn();
        };
    };


    private TransactionManager() {}

    //获取连接
    public static Connection getConn() {
        return tl.get();
    }

    //开启事务
    public static void startTran() {
        try {
           tl.get().setAutoCommit(false);
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }
    }

    //提交事务
    public static void commitTran() {
        try {
            tl.get().commit();
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }
    }

    // 回滚事务
    public static void rollbackTran()  {
        try {
            tl.get().rollback();
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }
    }

    // 释放资源
    public static void release() {
        JDBCUtils.close(tl.get(), null, null);
    }
}

```









## 开发切面：`TransAspect`

> 对以有方法进行扩展，根据目标方法是否具有`@Trans`注解决定要不要增强事务控制

```java
@Component
@Aspect
public class TransAspect {
    @Around("execution(* cn.tedu.service..*.*(..)) && @annotation(tr)")
    public Object aroud(ProceedingJoinPoint pjp,Trans tr) throws Throwable {
        //有注解说明需要事务，管理事务
        try {
            TransactionManager.startTran();
            Object retObj = pjp.proceed();
            TransactionManager.commitTran();
            return retObj;
        } catch (Throwable e) {
            TransactionManager.rollbackTran();
            throw e;
        } finally {
            TransactionManager.release();
        }
    }
}
```



## 测试

### 单线程

```java
@Test
public void test02() {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserServlet userServlet = (UserServlet) context.getBean("userServlet");
    userServlet.delUser(8);
}
@Test
public void test01() {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserServlet userServlet = (UserServlet) context.getBean("userServlet");
    userServlet.addUser(new User(0,"ddd",29));
}

```



### 多线程

```java
@Test
public void test03() {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserServlet userServlet = (UserServlet) context.getBean("userServlet");

    new Thread(new Runnable() {
        @Override
        public void run() {
            userServlet.addUser(new User(0,"ddd",29));
        }
    }).start();

    new Thread(new Runnable() {
        @Override
        public void run() {
            userServlet.addUser(new User(0,"eee",39));
        }
    }).start();

}

```

