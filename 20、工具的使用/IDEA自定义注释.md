[TOC]

# 类注释

在Settings中添加对应的模板即可

![image-20210402170014041](https://gitee.com/sxhDrk/images/raw/master/imgs-2021-04-27/image-20210402170014041.png)

创建完类后，展示下图效果

```java
/**
 * @Author shixiaohao
 * @Date 2021/4/2 17:01
 * @Description: Test
 */
public class Test {
}
```





# 方法注释

1. Settings中选择Live Templates

2. 点击+号，选择 Template Groups..

3. 输入自己的模板组名称

   ![image-20210402170344701](https://gitee.com/sxhDrk/images/raw/master/imgs-2021-04-27/image-20210402170627687.png)

4. 选中自己的模板组，点+号选择Live Template

5. 在`Template text`输入以下内容

   > **@author 后面跟自己的名字**

   ```java
   *
    * @author 你自己的名字
    * @createTime $date$ $TIME$
    * @description $description$
    * $param$
    * $return$
    * @throws $throws$
    */
   ```

   ![image-20210402170627687](https://gitee.com/sxhDrk/images/raw/master/imgs-2021-04-27/image-20210402170344701.png)

6. 在`Abbreviation`输入：`*`

7. 选择右侧的`Edit variables`，编辑模板变量显示的内容

   ![image-20210402170739801](https://gitee.com/sxhDrk/images/raw/master/imgs-2021-04-27/image-20210402170739801.png)

   其中return为：

   ```
   groovyScript("def result=''; def data=\"${_1}\"; def stop=false; if(data==null || data=='null' || data=='' || data=='void' ) { stop=true; }; if(!stop) { result += '@return  ' + data; }; return result;", methodReturnType())
   ```

   param为：

   ```
   groovyScript("def result=''; def stop=false; def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); if (params.size()==1 && (params[0]==null || params[0]=='null' || params[0]=='')) { stop=true; }; if(!stop) { for(i=0; i < params.size(); i++) {result +=((i==0) ? '\\r\\n' : '') + ((i < params.size() - 1) ? ' * @param  ' + params[i] + '\\r\\n' : ' * @param  ' + params[i] + '')}; }; return result;", methodParameters())
   ```

8. 点击右侧Options，选择触发的快捷键（建议Tab，避免覆盖IDEA默认的注释）

   ![image-20210402171524096](https://gitee.com/sxhDrk/images/raw/master/imgs-2021-04-27/image-20210402171234730.png)

9. 点击OK ---- Apply

10. 创建一个方法，查看效果

    > 在方法上  输入：/**，然后按Tab键，即可生成模板

    ```java
    package org.example;
    
    /**
     * @Author shixiaohao
     * @Date 2021/4/2 17:01
     * @Description: Test
     */
    public class Test {
    
        public static void main(String[] args) {
            new Test().Test("1", "2");
        }
    
        /**
         * @author shixiaohao
         * @createTime 2021/4/2 17:10
         * @description
         *
         * @param  id 用户ID
         * @param  name  用户名
         * @return  int 数量
         * @throws
         */
        public int Test(String id,String name) throws NullPointerException {
            return 0;
        }
    }
    
    ```

    > 这样创建出来的方法注释在调用时，也可以看到注释内容

    ![image-20210402171234730](https://gitee.com/sxhDrk/images/raw/master/imgs-2021-04-27/image-20210402171524096.png)