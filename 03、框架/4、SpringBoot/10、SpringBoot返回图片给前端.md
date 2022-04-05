[toc]



# 上传图片

# 显示图片

## 1. 直接返回文件byte数组

```java
    @RequestMapping(value = "/get",produces = MediaType.IMAGE_JPEG_VALUE)
    @ResponseBody
    public byte[] getImage() throws IOException {
        File file = new File("D://863247.jpg");
        FileInputStream inputStream = new FileInputStream(file);
        byte[] bytes = new byte[inputStream.available()];
        inputStream.read(bytes, 0, inputStream.available());
        inputStream.close();
        return bytes;
    }
```



## 2. 添加MVC连接器返回图片

1. 添加拦截器

   > SpringBoot 1.5.9.RELEASE 版本

   ```java
   @Configuration
   public class WebMvcConfig extends WebMvcConfigurerAdapter {
   
       /**
        * 增加图片转换器
        * @param converters
        */
       @Override
       public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
           converters.add(new BufferedImageHttpMessageConverter());
       }
   
   }
   ```

   > SpringBoot 2.3.3.RELEASE版本

   ```java
   @Configuration
   public class WebMvcConfig implements WebMvcConfigurer {
   
       /**
        * 增加图片转换器
        * @param converters
        */
       @Override
       public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
           converters.add(new BufferedImageHttpMessageConverter());
       }
   
   }
   ```

   

2. 接口返回图片

   ```java
       //通过produces 告知浏览器我要返回的媒体类型
       @GetMapping(value = "/getImage2", produces = {MediaType.IMAGE_JPEG_VALUE, MediaType.IMAGE_GIF_VALUE, MediaType.IMAGE_PNG_VALUE})
       @ResponseBody
       public BufferedImage getImage2() throws IOException {
           return ImageIO.read(new FileInputStream(new File("D://863247.jpg")));
       }
   ```

   

