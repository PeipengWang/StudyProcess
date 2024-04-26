## Fastjson 简介

Ffastjson是阿里巴巴的开源JSON解析库，它可以解析JSON格式的字符串，支持将Java Bean序列化为JSON字符串，也可以从JSON字符串反序列化到JavaBean。

Fastjson 可以操作任何 Java 对象，即使是一些预先存在的没有源码的对象。

Fastjson 源码地址：https://github.com/alibaba/fastjson

Fastjson 中文 Wiki：https://github.com/alibaba/fastjson/wiki/Quick-Start-CN

## 下载和使用

你可以在 maven 中央仓库中直接下载：

```
https://repo1.maven.org/maven2/com/alibaba/fastjson/
```

或者配置 maven 依赖:

```
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>x.x.x</version>
</dependency>
```

## 创建 JSONObject 对象

创建 JSON 对象非常简单，只需使用 JSONObject（fastJson提供的json对象） 和 JSONArray（fastJson提供json数组对象） 对象即可。

我们可以把JSONObject 当成一个 **Map<String,Object>** 来看，只是 JSONObject 提供了更为丰富便捷的方法，方便我们对于对象属性的操作。

1、直接构造

```
JSONObject json = new JSONObject();
json.put("aaa", "title");
json.put("bbb", 2);
json.put("ccc", new Object());
```

查看源码, 可知构造的时候传入 map

```
public JSONObject(Map<String, Object> map){
    if (map == null) {
            throw new IllegalArgumentException("map is null.");
      }
    this.map = map;
 }
```

2、JSON.parseObject 方法

```
// 字符串解析 JSON 对象
JSONObject obj = JSON.parseObject("{\"title\":\"fastJson真的很快\"}");
```

## 创建 JSONArray 对象

同样我们可以把 JSONArray 当做一个 List<Object>，可以把 JSONArray 看成 JSONObject 对象的一个集合。

1、直接构造

```
JSONArray json = new JSONArray();
json.add("xxx");
```

查看源码, 可知构造的时候传入 list

```
public JSONArray(List<Object> list){
   this.list = list;
}
```

2、JSON.parseArray 一系列方法

```
// 从字符串解析JSON数组
JSONArray arr = JSON.parseArray("[\"一只兔子\",\"两只绵羊\"]\n");
```

## 对象转 json 串 (序列化)

com.alibaba.fastjson.JSON的静态方法, 适用于任意 Object 对象.包括JSONObject, JSONArray

```
// 序列化
String text = JSON.toJSONString(obj);
```

由于 JSONObject 和 JSONArray 都继承与JSON抽象类, 如果直接得到了 JSON 抽象类的子类, 可以直接toString 或者 toJSONString方法(两者等价)进行序列化.

**序列化默认情况严格依赖 bean 的 Getter 方法，所以一定要规范 Getter 方法的写法。**

**标准写法**

```
public String getFullName() {
  return this.fullName;
}
```

### 序列化定制之 SerializerFeature

JSON.toJSONString 的一些定制功能

```
// 按理输出为 json 串, 成员变为 key, 为 value, 但是现在只取值组成 array
System.out.println(JSON.toJSONString(person, SerializerFeature.BeanToArray));
System.out.println(JSON.toJSONString(new Person(18, null, null), SerializerFeature.WriteNullStringAsEmpty));
```

### 序列化定制之 SerializeFilter

**NameFilter 对序列化后的参数名进行拦截处理。**

```
Object process(Object object, String name, Object value);
```

这个过滤器是用来拦截最后确定序列化的参数名时被调用的。返回值为最终确认的参数名，如果不做处理，那就直接返回name。

**ValueFilter 对序列化后的value进行拦截处理。**

```
String process(Object object, String name, Object value);
```

对于参数列表中三个参数都很好理解，第一个Object为现在被拦截参数的拥有者，第二个参数为其参数名，第三个参数为其参数值，其实这个地方重点要理解的是返回值。

ValueFilter的作用是在序列化之前对一些参数值做一些后置处理。例如参数值的类型为Double，但是你想做精度控制并且返回值为String，这个ValueFilter就有很大作用了，如果不做处理，直接返回value就可以了。

**ContextValueFilter extends SerializeFilter**

在某些场景下，对Value做过滤，需要获得所属JavaBean的信息，包括类型、字段、方法等。在fastjson-1.2.9中，提供了ContextValueFilter，类似于之前版本提供的**ValueFilter**，只是多了BeanContext参数可用。

```
package com.alibaba.fastjson.serializer;
public interface ContextValueFilter extends SerializeFilter {
    Object process(BeanContext context, 
           Object object, 
           String name, 
           Object value);
}
```



## json 串转对象 (反序列化)

```
VO vo = JSON.parseObject("{...}", VO.class); //反序列化
注意反序列化时为对象时，必须要有默认无参的构造函数，否则会报异常. 新版本不会报异常, 但是最好加上. 记住任何时候加空构造都是个好习惯.
```

## 配置类

### JSONField 类的说明

```
package com.alibaba.fastjson.annotation;
public @interface JSONField {
    // 配置序列化和反序列化的顺序，1.1.42版本之后才支持
    int ordinal() default 0;
     // 指定字段的名称
    String name() default "";
    // 指定字段的格式，对日期格式有用
    String format() default "";
    // 是否序列化
    boolean serialize() default true;
    // 是否反序列化
    boolean deserialize() default true;
}
```

- JSONField 注解可作用与 Field 或者 方法上, 也可以是 Setter (用于反序列化)和 Getter(序列化) 方法.

- 一个简单的使用就是 `@JSONField(name = "abc")`, 序列化和反序列话讲使用abc这个字段, 否则会使用成员变量的名字

举例`JSONField(name = "DATE OF BIRTH", format="yyyy-MM-dd HH:mm:ss", serialize = true, ordinal = 1)`

- JSONField 的 format 参数用于格式化 Date 类型。

```
// 配置date序列化和反序列使用yyyyMMdd日期格式
@JSONField(format="yyyy-MM-dd")
public Date date;
```

- 默认情况下， FastJson 库可以序列化 Java bean 实体， 但我们可以使用 JSONField 的 serialize 指定字段不序列化。

- 使用 JSONField 的 ordinal 参数指定字段的顺序. ordinal = 1表示排在第一列.

> 注意：FastJson 在进行操作时，是根据 getter 和 setter 的方法进行的，并不是依据 Field 进行。建议正常情况下选取注解field上即可. 不要两种都选取.
>
> **若属性是私有的，必须有 set 方法且set方法要书写正确。否则不会按照预期反序列化**。得不到该值, 该值会为 null.
>
> get 用于序列化成字符串. 若属性是私有的, 必须有 set 方法且get方法要书写正确. 否则该字段会被忽略掉!!!

## SerializeConfig

SerializeConfig：内部是个map容器主要功能是配置并记录每种Java类型对应的序列化类。



举例使用:

```java
SerializeConfig.getGlobalInstance().addFilter(A.class, upcaseNameFilter);
```

```java
public class ClassNameFilterTest extends TestCase {
    public void test_filter() throws Exception {
         NameFilter upcaseNameFilter = new NameFilter() {
            public String process(Object object, String name, Object value) {
                return name.toUpperCase();
        }
    };
        SerializeConfig.getGlobalInstance() //
               .addFilter(A.class, upcaseNameFilter);
        Assert.assertEquals("{\"ID\":0}", JSON.toJSONString(new A()));
        Assert.assertEquals("{\"id\":0}", JSON.toJSONString(new B()));
    }
    public static class A {
        public int id;
    }
    public static class B {
        public int id;
    }
}
```

## fastjson 的一些用法



用 fastjson 实现克隆



```
JSON.parseObject(JSON.toJSONString(this), this.getClass());
```



**将对象中的null赋值为""**



```
Object object = xxx;
JSON.parseObject(JSON.toJSONString(object, SerializerFeature.WriteNullStringAsEmpty), object.getClass());
```



### Fastjson的SerializerFeature序列化属性

QuoteFieldNames———-输出key时是否使用双引号,默认为true

WriteMapNullValue——–是否输出值为null的字段,默认为false

WriteNullNumberAsZero—-数值字段如果为null,输出为0,而非null

WriteNullListAsEmpty—–List字段如果为null,输出为[],而非null

WriteNullStringAsEmpty—字符类型字段如果为null,输出为”“,而非null

WriteNullBooleanAsFalse–Boolean字段如果为null,输出为false,而非null

## 示例

```
package qy.likai.demo;
import java.util.Collections;
import java.util.Date;
import java.util.List;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.annotation.JSONField;
import com.alibaba.fastjson.serializer.SerializerFeature;
public class App {
    
    public static void main( String[] args) {
        // 创建 JSONObject 对象
        JSONObject json = new JSONObject();
        json.put("aaa", "title");
        json.put("bbb", 2);
        json.put("ccc", 3.1415);
        json.put("ddd", true);
        // 对 null 默认是不输出
        json.put("eee", null);
        // 输出空对象
        json.put("fff", new Object());
        json.put("ggg", new int[]{1, 2, 3});
        json.put("hhh", Collections.singletonList(666));
        json.put("iii", Collections.singletonMap("my", "myvalue"));
        json.put("jjj", Boolean.TRUE);
        json.put("kkk", 'k');
        // 直接输出
        System.out.println(json.toString());
        // 两者等价
        System.out.println(json.toJSONString());
        // 两者等价
        System.out.println(JSON.toJSONString(json));
        // 序列化也可以定制一特定的功能, 比如
        final Person person = new Person(18, "zhangsan", new Date());
        System.out.println(JSON.toJSONString(person));
        // 按理输出为json 串, 成员变为key, 为value, 但是现在只取值组成array
        System.out.println(JSON.toJSONString(person, SerializerFeature.BeanToArray));
        System.out.println(JSON.toJSONString(new Person(18, null, null)));
        // WriteNullStringAsEmpty 字符类型字段如果为null,输出为 "", 而不是不输出 null 值
        System.out.println(JSON.toJSONString(new Person(18, null, null), SerializerFeature.WriteNullStringAsEmpty));
        // 反序列化 1
        // 从字符串解析JSON对象
        // parse 方法
        Object object = JSON.parse("{\"runoob\":\"菜鸟教程\"}");
        JSONObject jsonObject = (JSONObject)object;        
        // 也可以使用 JSON.parseObject 一步到位
        jsonObject = JSON.parseObject(json.toJSONString());
        
        // 反序列化 2
        //从字符串解析JSON数组
        object = JSON.parse("[\"菜鸟教程\",\"RUNOOB\"]\n");
        JSONArray arr = (JSONArray)object;
        System.out.println("arr1: " + arr);
        // 也可以使用 JSON.parseArray 一步到位
        arr = JSON.parseArray("[\"菜鸟教程\",\"RUNOOB\"]\n");
        System.out.println("arr2: " + arr);
        
        // json 串转 Java 对象
        Person p = JSON.parseObject("{\"AGE\":18,\"FULL NAME\":\"zhangsan\",\"birthday\":\"2020-04-25 12:12:53\"}", Person.class);
        // json串(数组类型) 转 list(需要传入已知类型)
        List<Integer> list = JSON.parseArray("[1,2,3]", Integer.class);
        System.out.println(list);
    }
    
    public static class Person {
        
        @JSONField(name = "AGE")
        private int age;
        
        @JSONField(name = "FULL NAME")
        private String fullName;
     
        @JSONField(name = "DATE OF BIRTH")
        private Date dateOfBirth;
        
        public Person(int age, String fullName, Date dateOfBirth) {
            super();
            this.age = age;
            this.fullName = fullName;
            this.dateOfBirth = dateOfBirth;
        }
        // 标准 getters & setters
        public int getAge() {
            return age;
        }
        public void setAge(int age) {
            this.age = age;
        }
        public String getFullName() {
            return this.fullName;
        }
        public void setFullName(String fullName) {
            this.fullName = fullName;
        }
        public Date getDateOfBirth() {
            return dateOfBirth;
        }
        public void setDateOfBirth(Date dateOfBirth) {
            this.dateOfBirth = dateOfBirth;
        }
        
    }
}
```



### fastjson 处理枚举



很多人也喜欢给枚举定义一个私有的属性，序列化为JSON时，希望以这个属性值作为value，这个时候就需要自己定义JSON的序列化和反序列化实现了。Fastjson提供了2个接口。用户控制序列化和反序列化行为，这个实在是太简单，这里不多说。看代码

- ObjectSerializer

- ObjectDeserializer

自定义 ObjectSerializer /ObjectDeserializer 的方式最为灵活，可以考虑抽象一个接口出来，让所有的枚举都实现接口。这样针对接口编写ObjectSerializer /ObjectDeserializer实现，就可以很好的复用了。

```
public static class GenderEnumSerializer implements ObjectSerializer {
        
        @Override
        public void write(JSONSerializer serializer, Object object, Object fieldName, Type fieldType, int features) throws IOException {
            // 强制把值转换为Gender
            Gender gender = (Gender) object;
            // 序列化为自定义的name属性，输出就行
            serializer.out.writeString(gender.getName());
        }
    }    
    
    public static class GenderEnumDeserializer implements ObjectDeserializer {
        
        @SuppressWarnings("unchecked")
        @Override
        public <T> T deserialze(DefaultJSONParser parser, Type type, Object fieldName) {
            // 解析值为字符串
            String value = parser.parseObject(String.class);
            // 遍历所有的枚举实例
            for (Gender gender : Gender.values()) {
                if (gender.getName().equals(value)) {
                    // 成功匹配，返回实例
                    return (T) gender;
                }
            }
            // 没有匹配到，可以抛出异常或者返回null
            return null;
        }
        @Override
        public int getFastMatchToken() {
            // 仅仅匹配字符串类型的值
            return JSONToken.LITERAL_STRING;
        }
    }
    enum Gender {
        
        BOY("男"), GIRL("女");
        
        public final String name;
        
        Gender(String name) {
            this.name = name;
        }
        public String getName() {
            return name;
        }
    }
    
    static class User {
        
        private Integer id;
        
        // 标识注解，指定枚举的序列化。反序列化实现类
        @JSONField(serializeUsing = GenderEnumSerializer.class, deserializeUsing = GenderEnumDeserializer.class)
        private Gender gender;
        public User() {
        }
        public User(Integer id, Gender gender) {
            super();
            this.id = id;
            this.gender = gender;
        }
        public Integer getId() {
            return id;
        }
        public void setId(Integer id) {
            this.id = id;
        }
        public Gender getGender() {
            return gender;
        }
        public void setGender(Gender gender) {
            this.gender = gender;
        }
    }
```

调用代码

```
// 序列化为JSON输出，枚举值为 getName()
        User user = new User(10002, Gender.BOY);
        String jsonString = JSON.toJSONString(user);
        System.out.println(jsonString); // {"gender":"男","id":10002}
        // 反序列化为对象
        user = JSON.parseObject(jsonString, User.class);
        System.out.println(user.getGender()); // BOY
```

### fastjson 处理布尔值

建议 POJO 中布尔值一律定义为 Boolean 类型，且都不要加 is前缀，防止一些框架解析引起的序列化错误。

```
public static class Person {
    
    private Boolean male;        
    public Boolean getMale() {
        return male;
    }
    public void setMale(Boolean male) {
        this.male = male;
    }
    @Override
    public String toString() {
        return "Person [male=" + male + "]";
    }    
}
```

## 参考

https://developer.aliyun.com/article/930348

Fastjson 项目地址

[https://github.com/alibaba/fastjson](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Falibaba%2Ffastjson)

Fastjson 简明教程

[https://www.runoob.com/w3cnote/fastjson-intro.html](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.runoob.com%2Fw3cnote%2Ffastjson-intro.html)

Fastjson处理枚举 - 技术交流 - SpringBoot中文社区

[https://springboot.io/t/topic/3648](https://links.jianshu.com/go?to=https%3A%2F%2Fspringboot.io%2Ft%2Ftopic%2F3648)
