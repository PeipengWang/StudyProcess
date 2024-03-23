1、查 web.xml 文件：
确保你的项目中只有一个 web.xml 文件，并且没有其他重复的配置。有时 IDE 会自动生成一个，你需要确认是否有多余的 web.xml 文件。

2、检查 Tomcat 配置：
在 Tomcat 的 conf/Catalina/localhost 目录下有可能存在与你的项目相关的 XML 文件（通常是项目的 WAR 文件名加 .xml 后缀）。确保这个文件中没有重复的配置。


3、清理 Tomcat 缓存：
在 Tomcat 的 work 目录下删除缓存，然后重新启动 Tomcat。

4、IDE 清理和重新构建：
如果你使用的是 IDE（如 IntelliJ IDEA 或 Eclipse），尝试执行清理和重新构建项目的操作，以确保 IDE 中的配置是正确的。
尤其是out目录，尽量清理一下