---
title: 几种流行的JSON库介绍
date: 2017-06-02 11:12:07
categories: Framework
tags:
- fastjson
- Gson
- jackjson
---

近年来JSON作为一个流行的数据交换格式逐渐取代了XML在网络通信方面的作用。基于REST原则的面向资源的方式再加上使用JSON串携带数据已经成为了成为了客户端和服务器端传递数据的主流。而在JAVA中便需要一些将POJO转换为标准的JSON的库来减轻开发人员的工作量。下面介绍一下当下流行的几种JSON库。

### fastjson

fastjson应该算是当下国内最流行的一个Json库，它是阿里的一位大神采用Java语言编写的。据说有无与伦比的性能，在解析速度上完虐国外其他的JSON库，但是网上的评论并不友好，看来是否真的完虐还值得商榷。本人也没有做过性能测试，这里便不置喙。但是在国内这个库确实相当的流行，在从JSON转化为POJO是确实性能优越，但是在复杂类型的POJO到JSON的转化上会出现以下问题，在功能上较之下面要介绍的Gson还有一些不足。因此国内的有些团队通常将它和Gson结合使用。下面介绍一下它的基础用法。

#### 常用API

下面是fastjson的一些常用的API，通过使用这写API便可以实现JSON与POJO的基本转化。fastjson中的方法一般很多都是静态的，在这一点上我认为是优于Gson和jackJson的地方，在使用上确实方便一些。

###### 序列化

```Java
package com.alibaba.fastjson;

public abstract class JSON {
    // 将Java对象序列化为JSON字符串，支持各种各种Java基本类型和JavaBean
    public static String toJSONString(Object object, SerializerFeature... features);

    // 将Java对象序列化为JSON字符串，返回JSON字符串的utf-8 bytes
    public static byte[] toJSONBytes(Object object, SerializerFeature... features);

    // 将Java对象序列化为JSON字符串，写入到Writer中
    public static void writeJSONString(Writer writer,
                                       Object object,
                                       SerializerFeature... features);

    // 将Java对象序列化为JSON字符串，按UTF-8编码写入到OutputStream中
    public static final int writeJSONString(OutputStream os, //
                                            Object object, //
                                            SerializerFeature... features);
}
```

###### 反序列化

```java
package com.alibaba.fastjson;

public abstract class JSON {
    // 将JSON字符串反序列化为JavaBean
    public static <T> T parseObject(String jsonStr,
                                    Class<T> clazz,
                                    Feature... features);

    // 将JSON字符串反序列化为JavaBean
    public static <T> T parseObject(byte[] jsonBytes,  // UTF-8格式的JSON字符串
                                    Class<T> clazz,
                                    Feature... features);

    // 将JSON字符串反序列化为泛型类型的JavaBean
    public static <T> T parseObject(String text,
                                    TypeReference<T> type,
                                    Feature... features);

    // 将JSON字符串反序列为JSONObject
    public static JSONObject parseObject(String text);
}
```

#### 代码示例

###### JSON转POJO

```Java
String jsonString = ...;
Group group = JSON.parseObject(jsonString, Group.class);
```

###### POJO转JSON

```Java
Group group = new Group();
String jsonString = JSON.toJSONString(group);
```

###### 集合泛型的反序列化

```Java
Type type = new TypeReference<List<Model>>() {}.getType();
List<Model> list = JSON.parseObject(jsonStr, type);
```

###### POJO转byte

```Java
Model model = ...;
byte[] jsonBytes = JSON.toJSONBytes(model);
```

###### 将POJO作为字符串写入OutputStream

```Java
Model model = ...;
OutputStream os;
JSON.writeJSONString(os, model);
```

###### 将POJO作为字符串写入Write

```Java
Model model = ...;
Writer writer = ...;
JSON.writeJSONString(writer, model);
```

### 定制序列化

有时我们在序列化是可能希望排除某些属性，这时就需要定制，fastjson在这方面也有支持。fastjson有多种定制序列化的方式：

- 通过@JSONField定制序列化
- 通过@JSONType定制序列化
- 通过SerializeFilter定制序列化
- 通过ParseProcess定制反序列化

###### 使用@JSONField定制序列化

源码种这个泛型的定义如下：

```JAVA
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER })
public @interface JSONField {
    /**
     * config encode/decode ordinal
     * @since 1.1.42
     * @return
     */
    int ordinal() default 0;
    String name() default "";
    String format() default "";
    boolean serialize() default true;
    boolean deserialize() default true;
    SerializerFeature[] serialzeFeatures() default {};
    Feature[] parseFeatures() default {};
    String label() default "";
    /**
     * @since 1.2.12
     */
    boolean jsonDirect() default false;
    /**
     * Serializer class to use for serializing associated value.
     *
     * @since 1.2.16
     */
    Class<?> serializeUsing() default Void.class;
    /**
     * Deserializer class to use for deserializing associated value.
     *
     * @since 1.2.16
     */
    Class<?> deserializeUsing() default Void.class;
    /**
     * @since 1.2.21
     * @return the alternative names of the field when it is deserialized
     */
    String[] alternateNames() default {};
    /**
     * @since 1.2.31
     */
    boolean unwrapped() default false;
}
```

我们可以在序列化的类的属性或者方法上使用上述注解来实现定制。如：

```JAVA
public class A {
     private int id;

     @JSONField(name="ID")
     public int getId() {return id;}
     @JSONField(name="ID")
     public void setId(int value) {this.id = id;}
}
```

或者：

```JAVA
public class A {
     @JSONField(name="ID")
     private int id;

     public int getId() {return id;}
     public void setId(int value) {this.id = id;}
}
```

###### 通过@JSONType定制序列化

源码如下：

```JAVA
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.TYPE })
public @interface JSONType {

    boolean asm() default true;
    String[] orders() default {};
    /**
     * @since 1.2.6
     */
    String[] includes() default {};
    String[] ignores() default {};
    SerializerFeature[] serialzeFeatures() default {};
    Feature[] parseFeatures() default {}
    boolean alphabetic() default true;
    Class<?> mappingTo() default Void.class;
    Class<?> builder() default Void.class;
    /**
     * @since 1.2.11
     */
    String typeName() default "";
    /**
     * @since 1.2.11
     */
    Class<?>[] seeAlso() default{};
    /**
     * @since 1.2.14
     */
    Class<?> serializer() default Void.class;
    /**
     * @since 1.2.14
     */
    Class<?> deserializer() default Void.class;
    boolean serializeEnumAsJavaBean() default false;
}
```

这个注解和@JSONField类似，但是是作用在类级别上的。

###### 通过SerializeFilter定制序列化

通过SerializeFilter可以使用扩展编程的方式实现定制序列化。fastjson提供了多种SerializeFilter：

- PropertyPreFilter 根据PropertyName判断是否序列化
- PropertyFilter 根据PropertyName和PropertyValue来判断是否序列化
- NameFilter 修改Key，如果需要修改Key,process返回值则可
- ValueFilter 修改Value
- BeforeFilter 序列化时在最前添加内容
- AfterFilter 序列化时在最后添加内容

代码示例：

```java
public interface PropertyFilter extends SerializeFilter {
    boolean apply(Object object, String propertyName, Object propertyValue);
}
```

```java
PropertyFilter filter = new PropertyFilter() {

    public boolean apply(Object source, String name, Object value) {
        if ("id".equals(name)) {
            int id = ((Integer) value).intValue();
            return id >= 100;
        }
        return false;
    }
};

JSON.toJSONString(obj, filter); // 序列化的时候传入filter
```

###### 通过ParseProcess定制反序列化

ParseProcess是编程扩展定制反序列化的接口。fastjson支持如下ParseProcess：

- ExtraProcessor 用于处理多余的字段
- ExtraTypeProvider 用于处理多余字段时提供类型信息

代码示例：

```java
public static class VO {
    private int id;
    private Map<String, Object> attributes = new HashMap<String, Object>();
    public int getId() { return id; }
    public void setId(int id) { this.id = id;}
    public Map<String, Object> getAttributes() { return attributes;}
}

ExtraProcessor processor = new ExtraProcessor() {
    public void processExtra(Object object, String key, Object value) {
        VO vo = (VO) object;
        vo.getAttributes().put(key, value);
    }
};

VO vo = JSON.parseObject("{\"id\":123,\"name\":\"abc\"}", VO.class, processor);
Assert.assertEquals(123, vo.getId());
Assert.assertEquals("abc", vo.getAttributes().get("name"));
```

#### JSONPath

fastjson中还有一个功能强大的工具类，主要源码如下：

```java
package com.alibaba.fastjson;

public class JSONPath {          
     //  求值，静态方法
     public static Object eval(Object rootObject, String path);
     // 计算Size，Map非空元素个数，对象非空元素个数，Collection的Size，数组的长度。其他无法求值返回-1
     public static int size(Object rootObject, String path);
     // 是否包含，path中是否存在对象
     public static boolean contains(Object rootObject, String path) { }
     // 是否包含，path中是否存在指定值，如果是集合或者数组，在集合中查找value是否存在
     public static boolean containsValue(Object rootObject, String path, Object value) { }
     // 修改制定路径的值，如果修改成功，返回true，否则返回false
     public static boolean set(Object rootObject, String path, Object value) {}
     // 在数组或者集合中添加元素
     public static boolean array_add(Object rootObject, String path, Object... values);
}
```

### Gson

Gson是Google的一个JSON开源库，功能强大，转换稳定，能够实现复杂类型的json到bean或bean到json的转换。在功能性和稳定性上它应该是比fastjson要优秀的，但是解析速度可能稍逊。在两者之间的选择可以结合项目需求进行抉择。

### jackson

jackson是国外比较流行的一个jackson库（国内的应该是fastjson用的比较多），被后端web开发的很多开源库所依赖，比如spring的webmvc项目。既然被那么多牛逼库所使用，说明它也是一个很优秀的项目，但是在国内使用不多，有兴趣的读者可以翻看一下它的文档。

### jsonlib

比较老的一个json库，现在已经很少被使用了，主要是它需要依赖很多第三方的包，且性能也不算好。

### 总结

这些json库使用起来都不是很难，毕竟源库的流行程度和它的学习曲线也是有关系的，因此很多开源库都讲究容易上手，当然要想达到精通，还是要发点时间学习的。上面这几个开源库读者只要熟悉fastjson和Gson，基本上已经可以满足开发需求了。
