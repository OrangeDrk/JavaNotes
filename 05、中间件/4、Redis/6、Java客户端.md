[TOC]

# 客户端

> redis作为流行的缓存软件，支持非常丰富的语言客户端
>
> - Java
> - c
> - c#
> - d
> - rub等。。。

![](https://gitee.com/sxhDrk/images/raw/master/imgs/redis客户端.png)





# Jedis的使用

## 添加依赖

```xml
<!--redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```



## 正常使用

> 使用Jedis对象连接redis服务端，操作redis服务节点

```java
/*
	如何使用jedis链接到对应的redis服务端
*/
@Test
public void test01(){
    //相当于从java代码执行 redis-cli -h 10.9.104.184 -p 6379
    Jedis jedis=new Jedis("10.9.104.184",6379);
    //可以利用jedis对象调用api方法操作redis
    jedis.set("name","王老师");
    System.out.println(jedis.get("name"));
    jedis.exists("name");
    jedis.lpush("list01","1");
    jedis.rpop("list01");
    jedis.hset("user","age","18");
}
```







# Jedis实现登录逻辑

> 用户登录中，用户状态流程图

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Jedis实现登录逻辑.png)



## 登录接口文件

| 后台接收 | /user/manage/login                                           |
| -------- | ------------------------------------------------------------ |
| 请求方式 | Post                                                         |
| 请求参数 | User user 只有用户名和密码,查询需要加密password              |
| 返回数据 | 返回SysResult对象的json,其结构:  Integer status; 200表示成功,其他表示失败  String msg;成功返回 “ok”,失败返回其他信息  Object data;根据需求携带其他数据 |
| 备注     | 更具查询结果 ，判断登陆合法。决定如何使用redis存储数据。在成功时编写cookie相关代码实现将数据关键key返回（ticket） |



## `UserController`

```java
//接受登陆请求，判断合法和存储redis
@RequestMapping("login")
public SysResult doLogin(User user, HttpServletRequest req, HttpServletResponse res){
    //判断 业务层返回数据，是否生成了存储在redis中的key值 ticket
    String ticket= us.doLogin(user);
    if("".equals(ticket)||ticket==null){
        //说明业务层么有存储redis，说明用户名密码不正确
        //登陆失败
        return SysResult.build(201,"登陆失败",null);
    }else{
        //正确存储到redis 登陆成功，控制层使用cookie技术，将返回ticket值
        //带会给浏览器，浏览器客户端才能在登陆之后，有了一张类似票的概念的数据
        //后续访问系统只要判断这张票是否合法 是否超时。
        //调用common-resources cookieUtils
        //其中set方法需要request和response对象 在controller里可以从springmvc拿到
        CookieUtils.setCookie(req,res,"EM_TICKET",ticket);
        //返回数据，告诉ajax登陆成功的
        return SysResult.ok();
    }
}
```



## `UserService`

> 需要用到ObjectMapper类：对象和JSON字符串的互相转换
>
> - `writeValueAsString`：将对象转换成JSON
> - `readValue`：将json字符串转换成对象

```java
@Autowired
private ShardedJedisPool pool;

public String doLogin(User user){
    /*
    	1 判断合法
    	2 存储redis
    */
    // 准备一个返回的字符串，默认值""
    String ticket="";
    //使用user参数 查询数据库，判断是否存在user行数据 验证合法
    //user里明文password加密
    user.setUserPassword(MD5Util.md5(user.getUserPassword()));
    User exist=um.selectUserByNameAndPw(user);//select * from t_user where name= and pw=
    if(exist==null){
        //说明没有查询到user对象登陆时不合法的
        return ticket;
    }else{//说明合法数据
        //创建key-value的数据将用户信息存储在redis供客户端后续访问使用
        //准备一个数据 userJson字符串 可以将密码做空 redis的value
        //调用一个对象的api方法 ObjectMapper 可以将json和对象进行相互转化 writeValueAsString
        //readValue
        exist.setUserPassword(null);
        //将exist 转化成字符串
        ObjectMapper om= MapperUtil.MP;
        
        //准备好链接redis的对象jedis
        //Jedis jedis=new Jedis("10.9.104.184",6380);
        
        //Jedis实现分布式,使用ShardedJedisPool连接池
        ShardedJedis jedis = pool.getResource();
        
        try{
            String uJson = om.writeValueAsString(exist);
            //给ticket赋值，ticket体现特点：用户不同时，ticket要不同，同一个用户，不同
            //时间登陆，生成的ticket也不同
            ticket="EM_TICKET"+System.currentTimeMillis()+user.getUserName();
            //需要存储在redis的key值和value都准备号了，下面可以set方法调用6380
            jedis.setex(ticket,60*60*2,uJson);//不能set永久数据，假设超时时间2小时
        }catch(Exception e){
            e.printStackTrace();
            return "";
        }finally {
            if(jedis!=null){
                //jedis.close();---->普通使用Jedis
                pool.returnResource(jedis);//使用连接池
            }
        }
        //当上述逻辑全部执行完毕，ticket该赋值，就赋值了
        return ticket;
    }
}

```



## `userMapper.xml`

```xml
<select id="selectUserByNameAndPw" resultType="User">
    select *
    from t_user
    where user_name = #{userName}
    and user_password = #{userPassword};
</select>
```



# 获取登录存储的用户信息

> 在登录逻辑执行完毕之后，用户浏览器访问登录，填写用户名密码，浏览器中就会出现cookie，里保存了ticket（redis中存储该用户数据的key值）。
>
> 一旦访问了easymall任何一个页面，都会调用一段js代码。
>
> - 获取浏览器cookie的EM_TICKET
>
> - 判断是否为空ticket
>
> - - 为空：说明用户没有登陆过
>   - 不为空：说明登录过，将会发送携带ticket的请求，到达服务器进行用户数据userJson的获取，获取之后将返回结果拼接到标签中，显示欢迎**回来



## 接口文件

| 后台接收 | /user/manage/query/{ticket}                                  |
| -------- | ------------------------------------------------------------ |
| 请求方式 | Get                                                          |
| 请求参数 | String ticket 就是用户登录时生成的rediskey值                 |
| 返回数据 | 返回SysResult对象的json,其结构: <br />Integer status; 200表示成功,其他表示失败  <br />String msg;成功返回 “ok”,失败返回其他信息 <br />==Object data;封装从redis获取的userJson== |



## `UserController`

```java
//获取用户登陆后存储在redis中的数据
@RequestMapping("query/{ticket}")
public SysResult queryUserData(@PathVariable String ticket){
    //调用业务层，执行redis链接获取数据逻辑
    String userJson=us.queryUserData(ticket);
    //有ticket就一定去得到数据吗？不一定，设置了登陆状态超时。
    if(userJson==null){
        //有ticket，但是redis没有userJson说明确实登陆过，但是已经超时了
        //返回201，不携带任何数据 data
        return SysResult.build(201,"登录可能超时",null);
    }else{
        //ticket也有登陆过，userJson也有表示没超时，登录状态是正常使用的
        return SysResult.build(200,"登录可用",userJson);
    }
}
```



## `UserService`

```java
@Autowired
private ShardedJedisPool pool;

//根据ticket查询redis中数据
public String queryUserData(String ticket){
    //创建jedis对象
    //Jedis jedis=new Jedis("10.9.104.184",6380);
    
    //使用连接池,Sharded底层是一致性hash算法
    ShardedJedis jedis = pool.getResource();
    
    //保证使用完毕，关闭对象
    try{
        //get方法返回get数据
        return jedis.get(ticket);
    }catch(Exception e){
        e.printStackTrace();
        return null;
    }finally {
        if(jedis!=null){
            jedis.close();
        }
    }
}
```



## zuul网关配置修改

> zuul网关会对cookie默认进行过滤，认为cookie等头信息是不安全的 。==默认删掉了==
>
> - request的cookie到达不了服务器
> - 服务器response的cookie也到达不了浏览器

需要在zuul网关中关闭敏感头

> `=`后面什么也不写，代表对任何信息都不敏感
>
> 如果需要对某一信息敏感，在`=`后添加即可

```properties
#关闭敏感头
zuul.sensitive-headers=
```



# 尚待解决的问题

## 业务功能

第一个：登录没有顶替（只允许一个账号最多有1个人登录，第二个人来时，将会把第一个人顶替）
