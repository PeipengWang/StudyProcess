# Mybatis
快速入门：
加入依赖
添加mybatis配置文件，一般spring中不用配
添加map.xml添加对sql的映射
编写实体类、map接口
1、读取mybatis的xml配置文件和注释中的配置信息，创建配置对象，并完成各个模块的初始化工作。
2、封装iBatis的编程模型，使用mapper接口开发的初始工作。
3、通过sqlSession完成SQL的解析，参数的映射、SQL的执行、结果的反射映射过程。
sqlSession定义了大部分sql语句内容，包括增删查改，commit、回滚等。

最后基于Executor实现。
sqlSession相当于接待于门户，只做规范工作
体现了单一职责原则。
![](_v_images/20220704220051156_10997.png =1229x)
实现思路：
1、创建SqlSessionFactory实例；
2、实例化过程中，加载配置文件创建configuration对象；
3、通过factory创建SqlSeesion
4、通过SqlSession获取mapper接口动态代理
5、动态代理回调sqlsession中查询方法；
6、Executor基于JDBC访问数据库获取数据；
7、Executor通过反射将数据转化为POJO并返回；给session
8、将数据返回调用者。
![](_v_images/20220704225239613_273.png =1013x)


