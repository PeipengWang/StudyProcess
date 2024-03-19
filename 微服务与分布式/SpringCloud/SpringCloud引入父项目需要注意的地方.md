要引入两个组件，一个是SpringCloud一个是SpringCloudAlibaba
组件依赖关系详细可查看：https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E
例如：
```
        <spring.cloud.alibaba.version>2021.0.1.0</spring.cloud.alibaba.version>
        <spring.cloud.version>2021.0.1</spring.cloud.version>
     <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
      </dependencyManagement>
```
注意：父项目要加    <packaging>pom</packaging>，父项目中</dependencyManagement>意味着可以被子项目共享
子项目要加    <packaging>jar</packaging>，子项目需要加个<parent>来引用父项目的资源
也可以是war，看打包方式
