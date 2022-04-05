# xml

> 1. 类似于一个配置文件，里面也有标签，可以自定义

## xml提取所有的服务器响应类型

- html------> text/html(判断文件是以html结尾的，返回类型就是text/html)

- xml中的形式：

  > <mime-mapping>
  >         <extension>html</extension>
  >         <mime-type>text/html</mime-type>
  >  </mime-mapping>

- ```java
  if("html".equals(name)) {
      os.write("Content-Type: text/html".getBytes("ISO8859-1"));
  }
  ```

> 1. 将xml文件放到项目文件的conf目录下
>
> 2. 在项目目录下新建文件夹lib，放入2个jar包
>
>    - dom4j-1.6.1jar
>    - xml-apis-1.0.b2.jar
>
>    选中这两个jar包，右键选择 `Build Path`,选择`Add Build Path`
>
> 3. 通过添加的jar包中的类创建对象
>
>    - `SAXReader reader = new SAXReader();`
>
> 4. 将xml文件读出来
>
>    - `Document doc = reader.read(newFile("conf/web.xml"));`
>
> 5. 获取xml文档中的根标签，也就是：<web-app>
>
>    - `Element root = doc.getRootElement();`
>
> 6. 获取根标签的子标签元素,子标签为：<mime-mapping>
>
>    - `List<Element> es = root.elements(mime-mapping);`
>
> 7. 遍历所有的子元素并添加到Map中
>
>    - 标签的形式
>
>      > <web-app>
>      >
>      > ​		<mime-mapping>
>      >
>      > ​			<extension>html</extension>
>      >
>      > ​			<mime-type>text/html</mime-type>
>      >
>      > ​		</mime-mapping>
>
>      > </web-app>
>
>    - 通过Map的键值对存放所有的  后缀-类型
>
>      `public static Map<String,String> mime_types = new HashMap<String,String>();`
>
>    ```java
>    for(Element e : es) {
>        String key = e.element("extension").getText();
>        String value = e.element("mime-type").getText();
>        mime_types.put(key, value);
>    }
>    ```
>
>    - 结果：
>
>      > mp3=audio/mpeg
>      >
>      > mp4=video/mp4
>      >
>      > html=text/html
>      >
>      > ………………

## 完整代码

```java
public class HttpContext {
	/*存放MIMEtype值的map，也就是Context-type的值*/
	public static Map<String,String> mime_types;
	static {
		try {
			mime_types = new HashMap<String,String>();
			//解析xml文件,SAXReader不在Java官方库里，在dom4j里
			SAXReader reader = new SAXReader();
			Document doc = reader.read(new File("conf/web.xml"));
			
			Element root = doc.getRootElement();//获取文档的根标签<web-app>
			List<Element> es = root.elements("mime-mapping");//根标签的子元素
			for(Element e : es) {
				String key = e.element("extension").getText();
				String value = e.element("mime-type").getText();
				mime_types.put(key, value);
			}
			
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		System.out.println(mime_types);	
	}
}
```

