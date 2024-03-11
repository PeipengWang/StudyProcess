# 搭建mybatis环境
1、创建数据库
```
#创建mybatis数据库
create database mybatis;
#使用数据库
use mybatis;
#创建表，有id， name， age
create table user (
  ID INT(11) PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR(18) DEFAULT NULL,
  AGE INT(11) DEFAULT NULL
)
```
2、创建Mybatis连接数据库的xml文件
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!--环境配置，连接的数据库，这里使用的是MySQL-->
    <environments default="mysql">
        <environment id="mysql">
            <!--指定事务管理的类型，这里简单使用Java的JDBC的提交和回滚设置-->
            <transactionManager type="JDBC"/>
            <!--dataSource 指连接源配置，POOLED是JDBC连接对象的数据源连接池的实现-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url"
                          value="jdbc:mysql://127.0.0.1/mybatis?useUnicode=true&amp;characterEncoding=utf8&amp;zeroDateTimeBehavior=convertToNull&amp;useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>        <!--这是告诉Mybatis区哪找持久化类的映射文件，对于在src下的文件直接写文件名，            如果在某包下，则要写明路径,如：com/mybatistest/config/User.xml-->
        <mapper resource="UserMapper.xml"/>
    </mappers>
</configuration>

```
3、创建UserMapper.xml
注意这个文件名称和路径要与mybatis的配置文件相对应
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.wpp.dao.UserDao">
    <select id="getUserById1" parameterType="int"
            resultType="com.wpp.pojo.User">
        select * from user where id = #{id}
    </select>
    <select id="getUserById2" parameterType="int"
            resultType="com.wpp.pojo.User">
        select * from user where id = #{id}
    </select>
<!--    <select id="selectList" resultType="com.wpp.pojo.User">-->
<!--        select * from t_user-->
<!--    </select>-->
</mapper>
```
其中id="getUserById1"时dao层的方法名称，也是一个唯一id
namespace是对应的类，这个类中定义了方法getUserById1
 parameterType="int"表示传入的参数类型为int
 resultType="com.wpp.pojo.User"代表返回的数据映射dao实体类User
4、创建dao层
```
package com.wpp.dao;

import com.wpp.pojo.User;

public interface UserDao {
    public User getUserById1(int id);
    public User getUserById2(int id);
}
```
需要注意getUserById1这个方法名称要与UserMapper.xml中的id保持一致，不然报错
5、创建实体类
```
package com.wpp.pojo;

public class User {
    private int ID;
    private String name;
    private int age;

    public User(int id, String name, int age){
        this.ID = id;
        this.name = name;
        this.age = age;
    }

    public void setID(int ID) {
        this.ID = ID;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public int getID() {
        return ID;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    @Override
    public String toString() {
        return "User{" +
                "ID=" + ID +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```
6、创建测试用例
```
package mybstis;

import com.wpp.pojo.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.apache.log4j.Logger;

import java.io.IOException;
import java.io.InputStream;

public class Test{

    Logger logger  =  Logger.getLogger(this.getClass());
    @org.junit.Test
    public void test() throws IOException {

        //读取配置文件
        InputStream is = Resources.getResourceAsStream("SqlMapperConfig.xml");
        //初始化mybatis，创建SqlSessionFactory类实例
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        //创建Session实例
        SqlSession session = sqlSessionFactory.openSession();
        //插入数据
        User user1 = session.selectOne("com.wpp.dao.UserDao.getUserById1", 1);
        System.out.println(user1);
        User user2 = session.selectOne("com.wpp.dao.UserDao.getUserById1", 1);
        System.out.println(user2);

        //关闭Session
        session.close();
    }
}
```
7、日志分析
23:17:14.374 [main] DEBUG org.apache.ibatis.datasource.pooled.PooledDataSource - Created connection 1381713434.
23:17:14.374 [main] DEBUG org.apache.ibatis.transaction.jdbc.JdbcTransaction - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@525b461a]
23:17:14.385 [main] DEBUG com.wpp.dao.UserDao.getUserById1 - ==>  Preparing: select * from user where id = ? 
23:17:14.415 [main] DEBUG com.wpp.dao.UserDao.getUserById1 - ==> Parameters: 1(Integer)
23:17:14.431 [main] DEBUG com.wpp.dao.UserDao.getUserById1 - <==      Total: 1
**User{ID=1, name='zhangsan', age=18}
User{ID=1, name='zhangsan', age=18}**
23:17:14.431 [main] DEBUG org.apache.ibatis.transaction.jdbc.JdbcTransaction - Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@525b461a]
23:17:14.432 [main] DEBUG org.apache.ibatis.transaction.jdbc.JdbcTransaction - Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@525b461a]
23:17:14.432 [main] DEBUG org.apache.ibatis.datasource.pooled.PooledDataSource - Returned connection 1381713434 to pool.
可以看出这个数据在查询时，第一次还需要生成sql来查询，但是第二次直接查到了，这是根据getUserById1这个id来确定缓存中的数据时存在的，因此直接查询。
## 错误一：
Error building SqlSession.
The error may exist in SQL Mapper Configuration
 Cause: org.apache.ibatis.builder.BuilderException: Error parsing SQL Mapper Configuration. Cause: org.apache.ibatis.datasource.DataSourceException: Unknown DataSource property: driverClass
解决：dirver写错了，写成了driverClass
 <property name="driver" value="com.mysql.cj.jdbc.Driver"/>