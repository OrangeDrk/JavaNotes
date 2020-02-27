# 插入一条新数据后，立即获取自增主键



## 对应的Mapper.xml

```xml
<!--添加一条广告后返回自增主键，该主键存放在传入的对象中，通过对象get方法即可获取-->
<insert id="addAdv" useGeneratedKeys="true" keyProperty="advid" parameterType="com.easyadv.domain.Adv">
    insert into adv (advid,advname,advcontext,totalmoney,topay,paymod,paytime,totaltimes)
    values (null , #{advname},#{advcontext},#{totalmoney},#{topay},#{paymod},#{paytime},#{totaltimes});
</insert>
```

## 调用方法

```java
Adv adv = new Adv();
xxxx.insertxxx(adv);
```

> 执行成功后，通过`adv.getAdvid()`即可获取新增的自增主键