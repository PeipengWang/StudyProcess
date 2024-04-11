
[Mybatis-plus官方网站](https://baomidou.com/)

@[TOC](SpringBoot整合MybatisPlus)


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 一，MybatisPlus比Mybatis的优势
摘自[Mybatis-Plus和Mybatis的区别](https://blog.csdn.net/qq_34508530/article/details/88943858?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160679054719724818030062%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=160679054719724818030062&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-1-88943858.first_rank_v2_pc_rank_v29&utm_term=Mybatisplus%E6%AF%94Mybatis&spm=1018.2118.3001.4449)
<font color=#999AAA >Mybatis-Plus是一个Mybatis的增强工具，它在Mybatis的基础上做了增强，却不做改变。我们在使用Mybatis-Plus之后既可以使用Mybatis-Plus的特有功能，又能够正常使用Mybatis的原生功能。Mybatis-Plus(以下简称MP)是为简化开发、提高开发效率而生，但它也提供了一些很有意思的插件，比如SQL性能监控、乐观锁、执行分析等。
如果Mybatis Plus是扳手，那Mybatis Generator就是生产扳手的工厂。
通俗来讲——
MyBatis：一种操作数据库的框架，提供一种Mapper类，支持让你用java代码进行增删改查的数据库操作，省去了每次都要手写sql语句的麻烦。但是！有一个前提，你得先在xml中写好sql语句，是不是很麻烦？于是有下面的↓
Mybatis Generator：自动为Mybatis生成简单的增删改查sql语句的工具，省去一大票时间，两者配合使用，开发速度快到飞起。至于标题说的↓
Mybatis Plus：国人团队苞米豆在Mybatis的基础上开发的框架，在Mybatis基础上扩展了许多功能，荣获了2018最受欢迎国产开源软件第5名，当然也有配套的↓
Mybatis Plus Generator：同样为苞米豆开发，比Mybatis Generator更加强大，支持功能更多，自动生成Entity、Mapper、Service、Controller等
总结：
数据库框架：Mybatis Plus > Mybatis
代码生成器：Mybatis Plus Generator > Mybatis Generator</font>


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">



# 二、快速开始


<font color=#999AAA >根据官方文档进行的利用SpringBoot整合MybatisPlus进行快速测试
## 1，数据库设计

```sql
DROP TABLE IF EXISTS user;

CREATE TABLE user
(
	id BIGINT(20) NOT NULL COMMENT '主键ID',
	name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
	age INT(11) NULL DEFAULT NULL COMMENT '年龄',
	email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
	PRIMARY KEY (id)
);
DELETE FROM user;

INSERT INTO user (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
```
## 2，添加依赖

```xml
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.1</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.19</version>
        </dependency>
```
## 3，配置文件

```yml
server:
  port: 8080
spring:
  datasource:
    username: root
    password: 123456
    #?serverTimezone=UTC解决时区的报错
    url: jdbc:mysql://localhost:3306/Mp?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource


```

##  4，根据数据库设计实体类

```java
package com.uestc.wpp.bean;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```
## 5，添加dao层

```java
package com.uestc.wpp.dao;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.uestc.wpp.bean.User;

public interface UserDao extends BaseMapper<User> {

}
```
## 6，测试

```java
package com.uestc.wpp;

import com.uestc.wpp.bean.User;
import com.uestc.wpp.dao.UserDao;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
class WppApplicationTests {

    @Autowired
    private UserDao userMapper;

    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ------"));
        List<User> userList = userMapper.selectList(null);
        userList.forEach(user -> System.out.println("user"+ user));
    }

}

```
得出结果：

```
----- selectAll method test ------
2020-12-01 11:27:18.510  INFO 3720 --- [           main] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} inited
2020-12-01 11:27:24.918 DEBUG 3720 --- [           main] com.uestc.wpp.dao.UserMapper.selectList  : ==>  Preparing: SELECT id,name,age,email FROM user
2020-12-01 11:27:25.003 DEBUG 3720 --- [           main] com.uestc.wpp.dao.UserMapper.selectList  : ==> Parameters: 
2020-12-01 11:27:25.491 DEBUG 3720 --- [           main] com.uestc.wpp.dao.UserMapper.selectList  : <==      Total: 5
userUser(id=1, name=Jone, age=18, email=test1@baomidou.com)
userUser(id=2, name=Jack, age=20, email=test2@baomidou.com)
userUser(id=3, name=Tom, age=28, email=test3@baomidou.com)
userUser(id=4, name=Sandy, age=21, email=test4@baomidou.com)
userUser(id=5, name=Billie, age=24, email=test5@baomidou.com)
```
可以发现不用在Dao层对应的xml文件中编写对应的sql语句，这样很方便的进行CRUD

## 7，其他方法

```java
    //主键查询
    @Test
    public void testById(){
        User user= userMapper.selectById("1");
        System.out.println(user);
    }
    //条件查询
    @Test
    public void testFind(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
//        queryWrapper.eq("age","28");//等值查询：年龄等于28
//        queryWrapper.lt("age","28");//年龄小于28
//          queryWrapper.le("age","28");//年龄小于等于28
//        如此类推gt为大于   ge：大于等于
        List<User> list = userMapper.selectList(queryWrapper);
        list.forEach(user -> System.out.println(user));
    }
    //模糊查询
    @Test
    public void testFindLike(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        //like %？% likeLeft %？ 以xxx结尾  likeRight ？% 以xxx开头
        queryWrapper.likeRight("username","王");
        List<User> list = userMapper.selectList(queryWrapper);
        list.forEach(user -> System.out.println(user));
    }
   
        //保存
    @Test
    public void testSave(){
        User user = new User();
        user.setId(8l);
        userMapper.insert(user);

    }
    //基于id修改方法
    public void testUpdateById(){
        User user = userMapper.selectById("1");
        user.setName("王者容易");
        userMapper.updateById(user);
    }
    //基于属性修改方法
    public void testUpdate(){
        //修改成的属性
        User user = new User();
        user.setName("王者容易");
        //设置不修改的属性
        user.setId(null);
        
        //查询符合条件的数据
       QueryWrapper<User> queryWrapper = new QueryWrapper<>();
       queryWrapper.eq("age",28);
       
       //对符合条件的数据修改
       userMapper.update(user,queryWrapper);
       
    }
```

# 二、MybatisPlus常用注解
[官方文档常用注解](https://baomidou.com/guide/annotation.html#tablename)，这里介绍最常用的几个注解
## 1.@ TableName
默认是将类名当做表名的，但是如果类名与表名不同，需要用这个注解来指定类所对应的表名。
常用属性：
|属性 |	类型	|必须指定|	默认值	|描述|
|--|--|--|--|--|--|--|--|
|value|	String|	否|	""|	表名|
resultMap|	String|	否|	""|	xml 中 resultMap 的 id

## 2.@TableId
value属性：主键注解，可以指定表中对应的名字。
type属性：枚举类型，指定主键生成类型
|值	|描述|
|--|--|
|AUTO	|数据库ID自增
|NONE|	无状态,该类型为未设置主键类型(注解里等于跟随全局,全局里约等于 INPUT)
|INPUT	|insert前自行set主键值
|ASSIGN_ID|	分配ID(主键类型为Number(Long和Integer)或String)(since 3.3.0),使用接口IdentifierGenerator的方法nextId(默认实现类为DefaultIdentifierGenerator雪花算法)
|ASSIGN_UUID	|分配UUID,主键类型为String(since 3.3.0),使用接口IdentifierGenerator的方法nextUUID(默认default方法)

## 3.@TableField
value属性：映射表中非主键属性
exits属性：不映射数据库表中的列，或者说表中没有这个属性对应的列。




#  三、分页查询
导入分页拦截器的配置：
Spring boot方式
```java
@Configuration
@MapperScan("com.uestc.wpp.dao")
public class MybatisPlusConfig {

    @Bean
    public PaginationInterceptor paginationInterceptor() {
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
        // paginationInterceptor.setOverflow(false);
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
        // paginationInterceptor.setLimit(500);
        // 开启 count 的 join 优化,只针对部分 left join
        paginationInterceptor.setCountSqlParser(new JsqlParserCountOptimize(true));
        return paginationInterceptor;
    }
}
```

这样就可以直接查询使用了

```java

    @Test
    public void testFindByPage(){
        //配置分页设置，设置第几页和每页数目
        IPage<User> page = new Page<>(1,2);
        IPage<User> userIPage = userMapper.selectPage(page,null);
        Long total = userIPage.getTotal();
        System.out.println("总记录数目："+total);
        userIPage.getRecords().forEach(user -> System.out.println("user = " +user));
    }
```
    注意：分页查询仅支持单表查询
    输出：
```
2020-12-01 15:36:58.155  INFO 12128 --- [           main] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} inited
2020-12-01 15:37:03.814 DEBUG 12128 --- [           main] com.uestc.wpp.dao.UserDao.selectPage     : ==>  Preparing: SELECT COUNT(1) FROM user
2020-12-01 15:37:03.898 DEBUG 12128 --- [           main] com.uestc.wpp.dao.UserDao.selectPage     : ==> Parameters: 
2020-12-01 15:37:04.067 DEBUG 12128 --- [           main] com.uestc.wpp.dao.UserDao.selectPage     : ==>  Preparing: SELECT id,name,age,email FROM user LIMIT ?
2020-12-01 15:37:04.070 DEBUG 12128 --- [           main] com.uestc.wpp.dao.UserDao.selectPage     : ==> Parameters: 2(Long)
2020-12-01 15:37:04.528 DEBUG 12128 --- [           main] com.uestc.wpp.dao.UserDao.selectPage     : <==      Total: 2
总记录数目：5
user = User(id=1, name=Jone, age=18, email=test1@baomidou.com)
user = User(id=2, name=Jack, age=20, email=test2@baomidou.com)
```
# 四、 多数据源实现读写分离
[多数据源文档](https://mybatis.plus/guide/dynamic-datasource.html)
## 1.引入依赖

```xml
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
            <version>3.0.0</version>
        </dependency>
```
## 2.配置数据源
```yml
spring:
  datasource:
    dynamic:
      primary: master #设置默认的数据源或者数据源组,默认值即为master
      strict: false #设置严格模式,默认false不启动. 启动后在未匹配到指定数据源时候会抛出异常,不启动则使用默认数据源.
      datasource:
        master:
          url: jdbc:mysql://xx.xx.xx.xx:3306/dynamic
          username: root
          password: 1234
          driver-class-name: com.mysql.jdbc.Driver # 3.2.0开始支持SPI可省略此配置
        slave_1:
          url: jdbc:mysql://xx.xx.xx.xx:3307/dynamic
          username: root
          password: 1234
          driver-class-name: com.mysql.jdbc.Driver
        slave_2:
          url: ENC(xxxxx) # 内置加密,使用请查看详细文档
          username: ENC(xxxxx)
          password: ENC(xxxxx)
          driver-class-name: com.mysql.jdbc.Driver
          schema: db/schema.sql # 配置则生效,自动初始化表结构
          data: db/data.sql # 配置则生效,自动初始化数据
          continue-on-error: true # 默认true,初始化失败是否继续
          separator: ";" # sql默认分号分隔符
          
     
```
## 3. DS注解
作用：用来切换数据源的注解
修饰范围：类或者方法，方法上优先于类上
value属性：切换数据源名称

## 4.业务层
业务接口

```java
public interface UserService {
    List<User> finDAll();
    void save(User user);
}

```
业务实现

```java
@Service
@Transactional
@DS(value = "master")
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;
    @Override
    @DS(value = "slave_1")
    public List<User> finDAll() {
        return userDao.selectList(null);
    }

    @Override
    public void save(User user) {
        userDao.insert(user);
    }
}

```
## 5.测试代码

```java
    @Autowired
    private UserService userService;
    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ------"));
        List<User> userList = userService.findAll();
        userList.forEach(user -> System.out.println("user"+ user));
    }
```
输出结果：

```
userUser(id=1, name=Jone, age=18, email=test1@baomidou.com)
userUser(id=2, name=Jack-1, age=20, email=test2@baomidou.com)
userUser(id=3, name=Tom, age=28, email=test3@baomidou.com)
userUser(id=4, name=Sandy, age=21, email=test4@baomidou.com)
userUser(id=5, name=Billie, age=24, email=test5@baomidou.com)
```
这个时候会发现，会在slave_1的从数据库中读取一个修改后的数据，在修改前name为jack.

# 五.代码生成器
## 1.导入依赖

```xml
       <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.4.1</version>
        </dependency>
        <dependency>
            <groupId>org.freemarker</groupId>
            <artifactId>freemarker</artifactId>
            <version>2.3.30</version>
        </dependency>
```
## 2.代码生成文件

```java
package com.uestc.wpp;

import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

// 演示例子，执行 main 方法控制台输入模块表名回车自动生成对应项目目录中
public class CodeGenerator {

    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotBlank(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("jobob");
        gc.setOpen(false);
        // gc.setSwagger2(true); 实体属性 Swagger2 注解
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/Mp?useUnicode=true&useSSL=false&characterEncoding=utf8");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName(scanner("模块名"));
        pc.setParent("com.uestc.wpp");
        mpg.setPackageInfo(pc);

        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };

        // 如果模板引擎是 freemarker
        String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
        // String templatePath = "/templates/mapper.xml.vm";

        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
                return projectPath + "/src/main/resources/mapper/" + pc.getModuleName()
                        + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
        /*
        cfg.setFileCreate(new IFileCreate() {
            @Override
            public boolean isCreate(ConfigBuilder configBuilder, FileType fileType, String filePath) {
                // 判断自定义文件夹是否需要创建
                checkDir("调用默认方法创建的目录，自定义目录用");
                if (fileType == FileType.MAPPER) {
                    // 已经生成 mapper 文件判断存在，不想重新生成返回 false
                    return !new File(filePath).exists();
                }
                // 允许生成模板文件
                return true;
            }
        });
        */
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);

        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();

        // 配置自定义输出模板
        //指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别
        // templateConfig.setEntity("templates/entity2.java");
        // templateConfig.setService();
        // templateConfig.setController();

        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setSuperEntityClass("你自己的父类实体,没有就不用设置!");
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        // 公共父类
        strategy.setSuperControllerClass("你自己的父类控制器,没有就不用设置!");
        // 写于父类中的公共字段
        strategy.setSuperEntityColumns("id");
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix(pc.getModuleName() + "_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }

}

```
运行上面的代码，输入想要加入的目录和表名即可。
