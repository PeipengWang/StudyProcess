PageHelper是Mybatis的一个很好的分页插件，但要使用它的分页功能需要注意一下几点

**1.导入相关包，例如maven导入依赖**

```c
 <dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.1.4</version>
</dependency>
```

**2.在mybatis-config.xml中添加插件**

引入 pageHelper插件

注意这里要写成PageInterceptor, 5.0之前的版本都是写PageHelper, 5.0之后要换成PageInterceptor

reasonable：分页合理化参数，默认值为false。
　　当该参数设置为 true 时，pageNum<=0 时会查询第一页，
　　pageNum>pages（超过总数时），会查询最后一页。
　　默认false 时，直接根据参数进行查询。

复制代码

```c
 <plugins>
     <plugin interceptor="com.github.pagehelper.PageInterceptor">
         <!--分页参数合理化  -->
        <property name="reasonable" value="true"/>
    </plugin>
 </plugins>

```

**3.在Controller的方法中**

以上代码后面需紧跟查询语句

```c
 List<Orders> ordersList = ordersService.findAll(page, size);
        //PageInfo就是一个分页Bean,
 PageInfo pageInfo=new PageInfo(ordersList);
 PageHelper.startPage(page,size);//从第一页开始，每页5条记录
 List<User> findAll();
```

当一个方法中有多个查询语句时，只有紧跟在PageHelper.starPage()方法后的查询结果才会分页。

缺少以上三步都会导致分页失效