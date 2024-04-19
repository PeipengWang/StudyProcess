邮件发送，在我们的日常开发中，也非常的多，使用原理如下所示：
我们配置zhangsan的邮箱，然后发送给lisi。
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201130141900442.png)
Springboot也帮我们做了支持，使用流程如下：
邮件发送需要引入spring-boot-start-mail
SpringBoot 自动配置MailSenderAutoConfiguration
定义MailProperties内容，配置在application.yml中
自动装配JavaMailSender
测试邮件发送

###  邮件发送需要引入依赖spring-boot-starter-mail

```xml
<!--邮件-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
```
### 定义MailProperties内容，配置在application.properties中

```
spring.mail.username=243588992@qq.com
spring.mail.password=你的授权码
#qq邮箱的服务器
spring.mail.host=smtp.qq.com
#开启验证信息
spring.mail.properties.mail.smtp.ssl.enable=true;
```



### 测试邮件发送
简单发送
```java
 @Autowired
    JavaMailSenderImpl mailSender;
    @Test
    void contextLoads() {
        SimpleMailMessage message = new SimpleMailMessage();
        //邮件设置
        message.setSubject("今晚开会");
        message.setText("今晚7点30，426开会");
        message.setTo("wangpeipeng123@163.com");
        message.setFrom("243588992@qq.com");
        mailSender.send(message);

    }
```

![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/2020113014435296.png)
复杂发送

```java
    @Test
    void contextLoads2() throws MessagingException {

        //一个复杂的邮件
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        //组装
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);

        //正文
        helper.setSubject("通知~plus");
        helper.setText("<p style='collor:red'>今晚开会，材料在附件</p>",true);

        //附件
        helper.addAttachment("材料.pptx",new File("E:\\桌面\\三反射镜.pptx"));

        helper.setTo("wangpeipeng123@163.com");
        helper.setFrom("243588992@qq.com");

        mailSender.send(mimeMessage);
    }
```
